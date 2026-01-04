# Export Pipeline Complete Specification

**Document Type:** Technical Specification  
**Version:** 1.0.0  
**Date:** 2026-01-02T00:00:00Z  
**Status:** Draft  
**Dependencies:** ARTIFACT_03 (The Bin), ARTIFACT_05 (Data Model), TRI_SURFACE_CORRECTED_ANALYSIS

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Pipeline Architecture](#2-pipeline-architecture)
3. [Template Formatter System](#3-template-formatter-system)
4. [Destination Handler System](#4-destination-handler-system)
5. [The Bin Integration](#5-the-bin-integration)
6. [Error Handling](#6-error-handling)
7. [Audit Trail](#7-audit-trail)
8. [Performance Considerations](#8-performance-considerations)
9. [Security Considerations](#9-security-considerations)
10. [API Specification](#10-api-specification)
11. [Implementation Notes](#11-implementation-notes)

---

## 1. Executive Summary

### 1.1 Purpose

The Export Pipeline is a system that transforms chat selections from the DevGuide Cockpit Reasoning surface into formatted files and publishes them to various destinations. It enables users to capture AI-assisted reasoning sessions, apply professional formatting templates, and persist outputs to local storage, version control systems, databases, or cloud storage.

### 1.2 Architecture Overview

The pipeline implements a **5-stage processing model**:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Assembly   â”‚â”€â”€â”€â–¶â”‚ Transformation  â”‚â”€â”€â”€â–¶â”‚  Validation  â”‚â”€â”€â”€â–¶â”‚  Preview  â”‚â”€â”€â”€â–¶â”‚  Publish  â”‚
â”‚  (Stage 1)  â”‚    â”‚   (Stage 2)     â”‚    â”‚  (Stage 3)   â”‚    â”‚ (Stage 4) â”‚    â”‚ (Stage 5) â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
      â”‚                    â”‚                     â”‚                  â”‚               â”‚
      â”‚                    â”‚                     â”‚                  â”‚               â”‚
      â–¼                    â–¼                     â–¼                  â–¼               â–¼
 AssembledContent    FormattedContent     ValidationResult   PreviewRendered  PublishResult
```

### 1.3 Core Principles

**All Mutations Flow Through The Bin:** Every export operation that creates, updates, or publishes content is validated by The Bin validation system. This ensures governance compliance, impact analysis, and audit trail integrity.

**Template-Based Formatting:** Content is transformed through configurable templates (Markdown, YAML, Skill, XML, custom) that enforce consistent output structure and metadata injection.

**Destination Abstraction:** Export destinations (Local, GitHub, TiDB, Drive) implement a common interface, enabling extensibility and consistent error handling.

### 1.4 Key Metrics

| Metric | Target | Maximum |
|--------|--------|---------|
| Assembly time | < 50ms | 200ms |
| Transformation time | < 100ms | 500ms |
| Validation time | < 200ms | 500ms |
| Preview render | < 300ms | 1s |
| Publish (local) | < 100ms | 500ms |
| Publish (remote) | < 2s | 10s |

---

## 2. Pipeline Architecture

### 2.1 Stage 1: Assembly

**Purpose:** Collect and organize selected chat content into a structured object.

#### Input

```typescript
interface SelectionSet {
  sessionId: string;           // Chat session identifier
  messageIds: string[];        // Selected message UUIDs
  blockIds?: string[];         // Optional: specific blocks within messages
  includeMetadata: boolean;    // Include timestamps, roles, etc.
}
```

#### Process

1. **Fetch Messages:** Query `chat_messages` table by IDs
2. **Order by Timestamp:** Sort ascending by `created_at`
3. **Preserve Provenance:** Retain role (user/assistant), agent_id, timestamp
4. **Calculate Statistics:** Token count, character count, block count
5. **Extract Blocks:** If `blockIds` specified, extract only selected blocks from message content

```python
class AssemblyStage:
    """Stage 1: Assemble selected content from database"""
    
    def __init__(self, db_client: TiDBClient):
        self.db = db_client
    
    async def execute(self, selection: SelectionSet) -> AssembledContent:
        # 1. Fetch messages
        messages = await self.db.query("""
            SELECT id, session_id, surface, agent_id, role, content, 
                   metadata, created_at
            FROM chat_messages
            WHERE id IN %(message_ids)s
            ORDER BY created_at ASC
        """, {'message_ids': tuple(selection.message_ids)})
        
        # 2. Extract blocks if specified
        blocks = []
        for msg in messages:
            if selection.block_ids:
                msg_blocks = self._extract_blocks(msg, selection.block_ids)
                blocks.extend(msg_blocks)
            else:
                blocks.append(ContentBlock(
                    id=msg['id'],
                    role=msg['role'],
                    content=msg['content'],
                    timestamp=msg['created_at'],
                    agent_id=msg['agent_id'],
                    surface=msg['surface']
                ))
        
        # 3. Calculate statistics
        full_content = '\n\n'.join(b.content for b in blocks)
        token_count = self._estimate_tokens(full_content)
        
        return AssembledContent(
            session_id=selection.session_id,
            blocks=blocks,
            raw_content=full_content,
            statistics=ContentStatistics(
                token_count=token_count,
                char_count=len(full_content),
                block_count=len(blocks),
                message_count=len(messages)
            ),
            provenance=self._build_provenance(messages)
        )
    
    def _extract_blocks(self, message: dict, block_ids: list) -> list:
        """Extract specific blocks from message metadata"""
        msg_metadata = message.get('metadata', {})
        msg_blocks = msg_metadata.get('blocks', [])
        return [
            ContentBlock(
                id=b['id'],
                role=message['role'],
                content=b['content'],
                timestamp=message['created_at'],
                agent_id=message['agent_id'],
                surface=message['surface']
            )
            for b in msg_blocks if b['id'] in block_ids
        ]
    
    def _estimate_tokens(self, content: str) -> int:
        """Approximate token count (4 chars â‰ˆ 1 token)"""
        return len(content) // 4
    
    def _build_provenance(self, messages: list) -> Provenance:
        """Build provenance chain from messages"""
        return Provenance(
            sources=[
                ProvenanceEntry(
                    message_id=m['id'],
                    role=m['role'],
                    timestamp=m['created_at'],
                    agent_id=m['agent_id']
                )
                for m in messages
            ]
        )
```

#### Output

```python
@dataclass
class AssembledContent:
    session_id: str
    blocks: List[ContentBlock]
    raw_content: str
    statistics: ContentStatistics
    provenance: Provenance

@dataclass
class ContentBlock:
    id: str
    role: str              # 'user' | 'assistant'
    content: str
    timestamp: datetime
    agent_id: Optional[str]
    surface: str           # 'inputs' | 'reasoning' | 'outputs'

@dataclass
class ContentStatistics:
    token_count: int
    char_count: int
    block_count: int
    message_count: int

@dataclass
class Provenance:
    sources: List[ProvenanceEntry]
```

### 2.2 Stage 2: Transformation

**Purpose:** Apply a template formatter to convert assembled content into the target format.

#### Input

```python
# AssembledContent from Stage 1
# Template specification
# User-provided metadata
```

#### Process

1. **Load Template:** Retrieve template by ID or name from registry
2. **Merge Metadata:** Combine system metadata with user-provided metadata
3. **Apply Formatter:** Execute template's `format()` method
4. **Inject Metadata:** Add metadata header/block per template rules
5. **Handle Edge Cases:** Empty content, special characters, encoding

```python
class TransformationStage:
    """Stage 2: Transform content using template"""
    
    def __init__(self, template_registry: TemplateRegistry):
        self.templates = template_registry
    
    async def execute(
        self,
        assembled: AssembledContent,
        template_id: str,
        user_metadata: Dict[str, Any]
    ) -> FormattedContent:
        # 1. Load template
        template = await self.templates.get(template_id)
        if not template:
            raise TemplateNotFoundError(f"Template '{template_id}' not found")
        
        # 2. Build complete metadata
        metadata = self._merge_metadata(assembled, user_metadata)
        
        # 3. Handle edge cases
        content = self._sanitize_content(assembled.raw_content)
        if not content.strip():
            raise EmptyContentError("Cannot export empty content")
        
        # 4. Apply template
        formatted = template.format(content, metadata)
        
        # 5. Add metadata per template
        formatted = template.add_metadata(formatted, metadata)
        
        return FormattedContent(
            content=formatted,
            template_id=template_id,
            template_name=template.name,
            metadata=metadata,
            original_statistics=assembled.statistics,
            provenance=assembled.provenance
        )
    
    def _merge_metadata(
        self,
        assembled: AssembledContent,
        user_metadata: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Merge system and user metadata"""
        return {
            # System metadata
            'session_id': assembled.session_id,
            'block_count': assembled.statistics.block_count,
            'token_count': assembled.statistics.token_count,
            'exported_at': datetime.now(timezone.utc).isoformat() + 'Z',
            'source_surface': assembled.blocks[0].surface if assembled.blocks else 'unknown',
            # User metadata (overrides system)
            **user_metadata
        }
    
    def _sanitize_content(self, content: str) -> str:
        """Handle special characters and encoding issues"""
        # Normalize unicode
        content = unicodedata.normalize('NFC', content)
        # Replace null bytes
        content = content.replace('\x00', '')
        return content
```

#### Output

```python
@dataclass
class FormattedContent:
    content: str                          # The formatted output
    template_id: str
    template_name: str
    metadata: Dict[str, Any]
    original_statistics: ContentStatistics
    provenance: Provenance
```

### 2.3 Stage 3: Validation

**Purpose:** Verify formatted content meets schema and content rules before proceeding.

#### Input

```python
# FormattedContent from Stage 2
# Template validation rules
# The Bin pre-flight check
```

#### Process

1. **Schema Validation:** Check format-specific syntax (YAML, XML, etc.)
2. **Content Rules:** Verify max length, required fields, forbidden patterns
3. **The Bin Pre-flight:** Submit validation request to The Bin
4. **Aggregate Results:** Combine all validation outcomes

```python
class ValidationStage:
    """Stage 3: Validate formatted content"""
    
    def __init__(
        self,
        template_registry: TemplateRegistry,
        bin_client: BinClient
    ):
        self.templates = template_registry
        self.bin = bin_client
    
    async def execute(
        self,
        formatted: FormattedContent,
        destination_id: str
    ) -> ValidationResult:
        errors = []
        warnings = []
        
        # 1. Schema validation
        template = await self.templates.get(formatted.template_id)
        if not template.validate(formatted.content):
            errors.append(ValidationError(
                code='SCHEMA_INVALID',
                message=f"Content does not match {template.name} schema",
                field='content',
                severity='error'
            ))
        
        # 2. Content rules
        content_errors = self._validate_content_rules(formatted)
        errors.extend(content_errors)
        
        # 3. The Bin pre-flight check
        bin_result = await self.bin.preflight_check(
            change_type='export',
            payload={
                'content_hash': hashlib.sha256(formatted.content.encode()).hexdigest(),
                'template_id': formatted.template_id,
                'destination_id': destination_id,
                'token_count': formatted.original_statistics.token_count
            }
        )
        
        if bin_result.blocked:
            errors.append(ValidationError(
                code='BIN_BLOCKED',
                message=bin_result.reason,
                field='bin_validation',
                severity='error'
            ))
        
        if bin_result.warnings:
            warnings.extend(bin_result.warnings)
        
        return ValidationResult(
            passed=len(errors) == 0,
            errors=errors,
            warnings=warnings,
            bin_transaction_id=bin_result.transaction_id
        )
    
    def _validate_content_rules(self, formatted: FormattedContent) -> List[ValidationError]:
        """Apply content validation rules"""
        errors = []
        
        # Max length check
        MAX_CONTENT_LENGTH = 1_000_000  # 1MB
        if len(formatted.content) > MAX_CONTENT_LENGTH:
            errors.append(ValidationError(
                code='CONTENT_TOO_LARGE',
                message=f"Content exceeds maximum length ({MAX_CONTENT_LENGTH} chars)",
                field='content',
                severity='error'
            ))
        
        # Required metadata fields
        required_fields = ['title', 'exported_at']
        for field in required_fields:
            if field not in formatted.metadata:
                errors.append(ValidationError(
                    code='MISSING_REQUIRED_FIELD',
                    message=f"Required metadata field '{field}' is missing",
                    field=f'metadata.{field}',
                    severity='error'
                ))
        
        return errors
```

#### Output

```python
@dataclass
class ValidationResult:
    passed: bool
    errors: List[ValidationError]
    warnings: List[ValidationWarning]
    bin_transaction_id: Optional[str]

@dataclass
class ValidationError:
    code: str
    message: str
    field: str
    severity: str  # 'error' | 'warning'
```

### 2.4 Stage 4: Preview

**Purpose:** Render content for user review before publishing.

#### Input

```python
# FormattedContent from Stage 2
# ValidationResult from Stage 3
# Optional: existing file for diff generation
```

#### Process

1. **Render Preview:** Generate display-ready preview
2. **Generate Diff:** If updating existing file, compute diff
3. **Compile Statistics:** Token count, file size, metadata summary
4. **Build Preview Object:** Package all preview data

```python
class PreviewStage:
    """Stage 4: Generate preview for user review"""
    
    def __init__(self, destination_registry: DestinationRegistry):
        self.destinations = destination_registry
    
    async def execute(
        self,
        formatted: FormattedContent,
        validation: ValidationResult,
        destination_id: str,
        filename: str
    ) -> PreviewRendered:
        destination = await self.destinations.get(destination_id)
        
        # 1. Check for existing file
        existing_content = None
        try:
            existing_content = await destination.fetch_existing(filename)
        except FileNotFoundError:
            pass
        
        # 2. Generate diff if updating
        diff = None
        if existing_content:
            diff = self._generate_diff(existing_content, formatted.content)
        
        # 3. Build preview
        return PreviewRendered(
            content=formatted.content,
            content_preview=self._truncate_preview(formatted.content, max_lines=50),
            filename=filename,
            destination_name=destination.name,
            destination_type=destination.type,
            is_update=existing_content is not None,
            diff=diff,
            metadata_summary=self._summarize_metadata(formatted.metadata),
            statistics=PreviewStatistics(
                file_size_bytes=len(formatted.content.encode('utf-8')),
                token_count=formatted.original_statistics.token_count,
                line_count=formatted.content.count('\n') + 1
            ),
            validation_passed=validation.passed,
            validation_warnings=validation.warnings
        )
    
    def _generate_diff(self, old: str, new: str) -> Diff:
        """Generate unified diff"""
        import difflib
        diff_lines = list(difflib.unified_diff(
            old.splitlines(keepends=True),
            new.splitlines(keepends=True),
            fromfile='existing',
            tofile='new'
        ))
        return Diff(
            unified='\n'.join(diff_lines),
            additions=sum(1 for l in diff_lines if l.startswith('+')),
            deletions=sum(1 for l in diff_lines if l.startswith('-'))
        )
    
    def _truncate_preview(self, content: str, max_lines: int) -> str:
        """Truncate content for preview display"""
        lines = content.split('\n')
        if len(lines) <= max_lines:
            return content
        return '\n'.join(lines[:max_lines]) + f'\n\n... ({len(lines) - max_lines} more lines)'
    
    def _summarize_metadata(self, metadata: Dict) -> Dict:
        """Create human-readable metadata summary"""
        return {
            'title': metadata.get('title', 'Untitled'),
            'author': metadata.get('author', 'Unknown'),
            'date': metadata.get('exported_at', 'Unknown'),
            'tags': ', '.join(metadata.get('tags', []))
        }
```

#### Output

```python
@dataclass
class PreviewRendered:
    content: str
    content_preview: str
    filename: str
    destination_name: str
    destination_type: str
    is_update: bool
    diff: Optional[Diff]
    metadata_summary: Dict[str, str]
    statistics: PreviewStatistics
    validation_passed: bool
    validation_warnings: List[ValidationWarning]

@dataclass
class Diff:
    unified: str
    additions: int
    deletions: int

@dataclass
class PreviewStatistics:
    file_size_bytes: int
    token_count: int
    line_count: int
```

### 2.5 Stage 5: Publish

**Purpose:** Execute the export to the target destination.

#### Input

```python
# FormattedContent from Stage 2
# ValidationResult from Stage 3 (must be passed)
# Destination configuration
# Filename
```

#### Process

1. **Final Validation:** Verify ValidationResult.passed == True
2. **Execute Handler:** Call destination's `save()` method
3. **Track in Database:** Create `export_jobs` record
4. **The Bin Commit:** Finalize Bin transaction
5. **Handle Failure:** Rollback mechanism if any step fails

```python
class PublishStage:
    """Stage 5: Publish to destination"""
    
    def __init__(
        self,
        destination_registry: DestinationRegistry,
        bin_client: BinClient,
        db_client: TiDBClient
    ):
        self.destinations = destination_registry
        self.bin = bin_client
        self.db = db_client
    
    async def execute(
        self,
        formatted: FormattedContent,
        validation: ValidationResult,
        destination_id: str,
        filename: str,
        user_id: str
    ) -> PublishResult:
        # 1. Final validation check
        if not validation.passed:
            return PublishResult(
                success=False,
                error=PublishError(
                    code='VALIDATION_FAILED',
                    message='Cannot publish: validation failed',
                    recoverable=False
                )
            )
        
        # 2. Create export job record
        job_id = str(uuid.uuid4())
        await self._create_job_record(job_id, formatted, destination_id, filename, user_id)
        
        try:
            # 3. Execute destination handler
            destination = await self.destinations.get(destination_id)
            location = await destination.save(filename, formatted.content)
            
            # 4. The Bin final commit
            await self.bin.commit(validation.bin_transaction_id)
            
            # 5. Update job record - success
            await self._update_job_status(
                job_id,
                status='completed',
                location=location
            )
            
            return PublishResult(
                success=True,
                job_id=job_id,
                location=location,
                destination_name=destination.name
            )
            
        except Exception as e:
            # 6. Rollback on failure
            await self._handle_failure(job_id, validation.bin_transaction_id, e)
            
            return PublishResult(
                success=False,
                job_id=job_id,
                error=PublishError(
                    code='PUBLISH_FAILED',
                    message=str(e),
                    recoverable=isinstance(e, (ConnectionError, TimeoutError))
                )
            )
    
    async def _create_job_record(
        self,
        job_id: str,
        formatted: FormattedContent,
        destination_id: str,
        filename: str,
        user_id: str
    ):
        """Create initial export job record"""
        await self.db.execute("""
            INSERT INTO export_jobs (
                id, user_id, session_id, template_id, destination_id,
                filename, status, content_hash, token_count, created_at
            ) VALUES (
                %(id)s, %(user_id)s, %(session_id)s, %(template_id)s,
                %(destination_id)s, %(filename)s, 'running',
                %(content_hash)s, %(token_count)s, NOW()
            )
        """, {
            'id': job_id,
            'user_id': user_id,
            'session_id': formatted.provenance.sources[0].message_id if formatted.provenance.sources else None,
            'template_id': formatted.template_id,
            'destination_id': destination_id,
            'filename': filename,
            'content_hash': hashlib.sha256(formatted.content.encode()).hexdigest(),
            'token_count': formatted.original_statistics.token_count
        })
    
    async def _update_job_status(
        self,
        job_id: str,
        status: str,
        location: str = None,
        error_message: str = None
    ):
        """Update export job status"""
        await self.db.execute("""
            UPDATE export_jobs 
            SET status = %(status)s,
                output_location = %(location)s,
                error_message = %(error)s,
                completed_at = CASE WHEN %(status)s IN ('completed', 'failed') 
                               THEN NOW() ELSE completed_at END
            WHERE id = %(id)s
        """, {
            'id': job_id,
            'status': status,
            'location': location,
            'error': error_message
        })
    
    async def _handle_failure(
        self,
        job_id: str,
        bin_transaction_id: str,
        error: Exception
    ):
        """Handle publish failure with rollback"""
        # Rollback Bin transaction
        try:
            await self.bin.rollback(bin_transaction_id)
        except Exception as rollback_error:
            # Log but don't raise - original error is more important
            logger.error(f"Bin rollback failed: {rollback_error}")
        
        # Update job status
        await self._update_job_status(
            job_id,
            status='failed',
            error_message=str(error)
        )
```

#### Output

```python
@dataclass
class PublishResult:
    success: bool
    job_id: Optional[str] = None
    location: Optional[str] = None        # URL, path, or record ID
    destination_name: Optional[str] = None
    error: Optional[PublishError] = None

@dataclass
class PublishError:
    code: str
    message: str
    recoverable: bool                      # Can retry?
```

---

## 3. Template Formatter System

### 3.1 Base Class Architecture

```python
from abc import ABC, abstractmethod
from typing import Dict, Any
from datetime import datetime

class ExportTemplate(ABC):
    """Base class for all export templates"""
    
    def __init__(self, name: str, config: Dict[str, Any] = None):
        self.name = name
        self.config = config or {}
        self.version = "1.0.0"
    
    @abstractmethod
    def format(self, content: str, metadata: Dict[str, Any]) -> str:
        """
        Format content according to template rules.
        
        Args:
            content: Raw content string
            metadata: Dictionary of metadata fields
            
        Returns:
            Formatted content string
        """
        pass
    
    def add_metadata(self, content: str, metadata: Dict[str, Any]) -> str:
        """
        Add metadata section to formatted content.
        Default: no-op. Override in subclasses.
        """
        return content
    
    def validate(self, formatted_content: str) -> bool:
        """
        Validate that formatted content meets template requirements.
        Default: always valid. Override for schema validation.
        """
        return True
    
    def get_file_extension(self) -> str:
        """Return the appropriate file extension for this template"""
        return ".md"
    
    def get_mime_type(self) -> str:
        """Return MIME type for this template's output"""
        return "text/markdown"
```

### 3.2 Built-in Templates

#### MarkdownTemplate

**Format:** Plain Markdown with title and metadata header  
**Use Case:** General documentation, notes, articles

```python
class MarkdownTemplate(ExportTemplate):
    """Simple Markdown formatter with header"""
    
    def __init__(self, name: str = 'markdown', config: Dict = None):
        super().__init__(name, config)
    
    def format(self, content: str, metadata: Dict[str, Any]) -> str:
        title = metadata.get('title', 'Untitled Document')
        author = metadata.get('author', 'Unknown')
        date = metadata.get('date', datetime.now().strftime('%Y-%m-%d'))
        
        formatted = f"""# {title}

**Author:** {author}  
**Date:** {date}  

---

{content}
"""
        return formatted
    
    def validate(self, formatted_content: str) -> bool:
        # Basic validation: must have title
        return formatted_content.startswith('# ')
    
    def get_file_extension(self) -> str:
        return ".md"
```

#### YAMLFrontMatterTemplate

**Format:** YAML front matter block + Markdown body  
**Validation:** YAML syntax check  
**Use Case:** Static site generators (Jekyll, Hugo), documentation systems

```python
import yaml

class YAMLFrontMatterTemplate(ExportTemplate):
    """Markdown with YAML front matter"""
    
    def __init__(self, name: str = 'yaml-markdown', config: Dict = None):
        super().__init__(name, config)
        self._required_fields = config.get('required_fields', ['title'])
    
    def format(self, content: str, metadata: Dict[str, Any]) -> str:
        # Build front matter
        front_matter_data = {
            'title': metadata.get('title', 'Untitled'),
            'author': metadata.get('author', 'Unknown'),
            'date': metadata.get('exported_at', datetime.now().isoformat() + 'Z'),
            'tags': metadata.get('tags', []),
            'format': 'markdown',
            'source': {
                'session_id': metadata.get('session_id'),
                'block_count': metadata.get('block_count'),
                'token_count': metadata.get('token_count')
            }
        }
        
        # Add custom metadata
        for key, value in metadata.items():
            if key not in front_matter_data and not key.startswith('_'):
                front_matter_data[key] = value
        
        front_matter = yaml.dump(
            front_matter_data,
            default_flow_style=False,
            allow_unicode=True,
            sort_keys=False
        )
        
        return f"""---
{front_matter}---

{content}
"""
    
    def validate(self, formatted_content: str) -> bool:
        """Validate YAML front matter syntax"""
        try:
            if not formatted_content.startswith('---'):
                return False
            
            parts = formatted_content.split('---', 2)
            if len(parts) < 3:
                return False
            
            # Parse YAML
            parsed = yaml.safe_load(parts[1])
            
            # Check required fields
            for field in self._required_fields:
                if field not in parsed:
                    return False
            
            return True
        except yaml.YAMLError:
            return False
    
    def get_file_extension(self) -> str:
        return ".md"
```

#### SkillFileTemplate

**Format:** DevGuide SKILL.md structure  
**Validation:** Required sections present  
**Use Case:** Claude/DevGuide skill files

```python
class SkillFileTemplate(ExportTemplate):
    """Format for DevGuide SKILL.md files"""
    
    REQUIRED_SECTIONS = ['Description', 'Instructions', 'Metadata']
    
    def __init__(self, name: str = 'skill', config: Dict = None):
        super().__init__(name, config)
    
    def format(self, content: str, metadata: Dict[str, Any]) -> str:
        name = metadata.get('name', 'custom-skill')
        description = metadata.get('description', 'A custom skill exported from DevGuide Cockpit.')
        version = metadata.get('version', '1.0.0')
        author = metadata.get('author', 'Unknown')
        
        return f"""# Skill: {name}

## Description

{description}

## Instructions

{content}

## Examples

_Add usage examples here._

## Metadata

| Field | Value |
|-------|-------|
| Name | {name} |
| Version | {version} |
| Author | {author} |
| Created | {datetime.now().isoformat()}Z |
| Type | custom-skill |
| Source | DevGuide Cockpit Export |

---

_Exported from DevGuide Cockpit on {datetime.now().strftime('%Y-%m-%d')}._
"""
    
    def validate(self, formatted_content: str) -> bool:
        """Validate required sections present"""
        for section in self.REQUIRED_SECTIONS:
            if f"## {section}" not in formatted_content:
                return False
        return True
    
    def get_file_extension(self) -> str:
        return ".md"
```

#### XMLWrapperTemplate

**Format:** XML document with structured sections  
**Validation:** Tag balance, well-formed XML  
**Use Case:** System prompts, structured data exchange

```python
import xml.etree.ElementTree as ET
from xml.sax.saxutils import escape

class XMLWrapperTemplate(ExportTemplate):
    """XML document wrapper"""
    
    def __init__(self, name: str = 'xml', config: Dict = None):
        super().__init__(name, config)
        self._root_tag = config.get('root_tag', 'document')
        self._content_tag = config.get('content_tag', 'content')
    
    def format(self, content: str, metadata: Dict[str, Any]) -> str:
        # Build metadata XML
        meta_entries = []
        for key, value in metadata.items():
            if isinstance(value, (str, int, float, bool)):
                meta_entries.append(f'    <{key}>{escape(str(value))}</{key}>')
            elif isinstance(value, list):
                items = ''.join(f'<item>{escape(str(v))}</item>' for v in value)
                meta_entries.append(f'    <{key}>{items}</{key}>')
        
        meta_xml = '\n'.join(meta_entries)
        
        # Escape content
        escaped_content = escape(content)
        
        return f"""<?xml version="1.0" encoding="UTF-8"?>
<{self._root_tag}>
  <metadata>
{meta_xml}
  </metadata>
  <{self._content_tag}>
{escaped_content}
  </{self._content_tag}>
</{self._root_tag}>
"""
    
    def validate(self, formatted_content: str) -> bool:
        """Validate well-formed XML"""
        try:
            ET.fromstring(formatted_content)
            return True
        except ET.ParseError:
            return False
    
    def get_file_extension(self) -> str:
        return ".xml"
    
    def get_mime_type(self) -> str:
        return "application/xml"
```

### 3.3 Custom Template Creation

Users can create custom templates via Python scripts or Jinja2 templates.

#### Template Registry

```python
class TemplateRegistry:
    """Registry for managing export templates"""
    
    def __init__(self, db_client: TiDBClient):
        self.db = db_client
        self._cache: Dict[str, ExportTemplate] = {}
        self._builtin = {
            'markdown': MarkdownTemplate(),
            'yaml-markdown': YAMLFrontMatterTemplate(),
            'skill': SkillFileTemplate(),
            'xml': XMLWrapperTemplate()
        }
    
    async def get(self, template_id: str) -> Optional[ExportTemplate]:
        """Get template by ID"""
        # Check builtin
        if template_id in self._builtin:
            return self._builtin[template_id]
        
        # Check cache
        if template_id in self._cache:
            return self._cache[template_id]
        
        # Load from database
        template = await self._load_from_db(template_id)
        if template:
            self._cache[template_id] = template
        
        return template
    
    async def register(
        self,
        name: str,
        template_type: str,  # 'python' | 'jinja2'
        template_content: str,
        config: Dict = None,
        user_id: str = None
    ) -> str:
        """Register a new custom template"""
        template_id = str(uuid.uuid4())
        
        # Validate template before saving
        template = self._create_from_content(
            template_type, template_content, name, config
        )
        
        # Test with sample content
        try:
            sample = template.format("Test content", {"title": "Test"})
            if not template.validate(sample):
                raise TemplateValidationError("Template failed self-validation")
        except Exception as e:
            raise TemplateValidationError(f"Template test failed: {e}")
        
        # Save to database
        await self.db.execute("""
            INSERT INTO export_templates (
                id, name, template_type, template_content, config,
                created_by, created_at, is_active
            ) VALUES (
                %(id)s, %(name)s, %(type)s, %(content)s, %(config)s,
                %(user)s, NOW(), TRUE
            )
        """, {
            'id': template_id,
            'name': name,
            'type': template_type,
            'content': template_content,
            'config': json.dumps(config or {}),
            'user': user_id
        })
        
        return template_id
    
    def _create_from_content(
        self,
        template_type: str,
        content: str,
        name: str,
        config: Dict
    ) -> ExportTemplate:
        """Create template instance from stored content"""
        if template_type == 'jinja2':
            return Jinja2Template(name, content, config)
        elif template_type == 'python':
            return self._create_python_template(name, content, config)
        else:
            raise ValueError(f"Unknown template type: {template_type}")
    
    def _create_python_template(
        self,
        name: str,
        code: str,
        config: Dict
    ) -> ExportTemplate:
        """Create template from Python code (sandboxed)"""
        # Execute in restricted namespace
        sandbox = RestrictedPython.compile_restricted(code)
        namespace = {'ExportTemplate': ExportTemplate}
        exec(sandbox, namespace)
        
        # Find the template class
        for item in namespace.values():
            if (isinstance(item, type) and 
                issubclass(item, ExportTemplate) and 
                item != ExportTemplate):
                return item(name, config)
        
        raise TemplateValidationError("No ExportTemplate subclass found in code")
```

#### Jinja2 Template Wrapper

```python
from jinja2 import Environment, BaseLoader, sandbox

class Jinja2Template(ExportTemplate):
    """Template using Jinja2 syntax"""
    
    def __init__(self, name: str, template_str: str, config: Dict = None):
        super().__init__(name, config)
        self._env = sandbox.SandboxedEnvironment(loader=BaseLoader())
        self._template = self._env.from_string(template_str)
    
    def format(self, content: str, metadata: Dict[str, Any]) -> str:
        return self._template.render(
            content=content,
            metadata=metadata,
            now=datetime.now()
        )
```

#### Database Schema for Templates

```sql
CREATE TABLE export_templates (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    template_type ENUM('python', 'jinja2') NOT NULL,
    template_content LONGTEXT NOT NULL,
    config JSON,
    created_by VARCHAR(36),
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    is_active BOOLEAN DEFAULT TRUE,
    version VARCHAR(20) DEFAULT '1.0.0',
    UNIQUE KEY idx_name (name),
    INDEX idx_created_by (created_by),
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 4. Destination Handler System

### 4.1 Base Class Architecture

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional

class ExportDestination(ABC):
    """Base class for export destinations"""
    
    def __init__(self, name: str, config: Dict[str, Any]):
        self.name = name
        self.config = config
        self.type = self.__class__.__name__
    
    @abstractmethod
    async def save(self, filename: str, content: str) -> str:
        """
        Save content to destination.
        
        Args:
            filename: Target filename
            content: Content to save
            
        Returns:
            Location identifier (URL, path, or record ID)
        """
        pass
    
    @abstractmethod
    def validate_config(self) -> bool:
        """Validate destination configuration is complete and correct"""
        pass
    
    @abstractmethod
    async def test_connection(self) -> bool:
        """Test that destination is reachable and accessible"""
        pass
    
    async def fetch_existing(self, filename: str) -> Optional[str]:
        """
        Fetch existing file content for diff generation.
        Returns None if file doesn't exist.
        """
        return None
    
    def get_location_url(self, location: str) -> Optional[str]:
        """Convert location identifier to user-facing URL"""
        return None
```

### 4.2 Built-in Destinations

#### LocalFileDestination

**Saves to:** Browser Downloads folder (web) or specified path (desktop)  
**Use Case:** Quick local saves, offline work

```python
import os
from pathlib import Path
import re

class LocalFileDestination(ExportDestination):
    """Save to local file system"""
    
    def __init__(self, name: str = 'local', config: Dict = None):
        super().__init__(name, config or {})
        self._base_path = config.get('base_path', './exports')
    
    def validate_config(self) -> bool:
        return True  # Local always valid
    
    async def test_connection(self) -> bool:
        """Check write access to base path"""
        try:
            Path(self._base_path).mkdir(parents=True, exist_ok=True)
            test_file = Path(self._base_path) / '.write_test'
            test_file.touch()
            test_file.unlink()
            return True
        except (PermissionError, OSError):
            return False
    
    async def save(self, filename: str, content: str) -> str:
        # Sanitize filename
        safe_filename = self._sanitize_filename(filename)
        
        # Ensure directory exists
        Path(self._base_path).mkdir(parents=True, exist_ok=True)
        
        # Build full path
        filepath = os.path.join(self._base_path, safe_filename)
        
        # Handle existing file
        if os.path.exists(filepath):
            filepath = self._get_unique_path(filepath)
        
        # Write file
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(content)
        
        return filepath
    
    async def fetch_existing(self, filename: str) -> Optional[str]:
        safe_filename = self._sanitize_filename(filename)
        filepath = os.path.join(self._base_path, safe_filename)
        
        try:
            with open(filepath, 'r', encoding='utf-8') as f:
                return f.read()
        except FileNotFoundError:
            return None
    
    def _sanitize_filename(self, filename: str) -> str:
        """Remove unsafe characters from filename"""
        # Remove path traversal attempts
        filename = os.path.basename(filename)
        # Remove unsafe characters
        filename = re.sub(r'[<>:"/\\|?*]', '_', filename)
        # Limit length
        return filename[:255]
    
    def _get_unique_path(self, filepath: str) -> str:
        """Add suffix if file exists"""
        base, ext = os.path.splitext(filepath)
        counter = 1
        while os.path.exists(filepath):
            filepath = f"{base}_{counter}{ext}"
            counter += 1
        return filepath
    
    def get_location_url(self, location: str) -> Optional[str]:
        return f"file://{os.path.abspath(location)}"
```

#### GitHubDestination

**Saves to:** GitHub repository via API  
**Configuration:** Repository, branch, path, token  
**Use Case:** Documentation updates, code snippets, version-controlled content

```python
import aiohttp
import base64

class GitHubDestination(ExportDestination):
    """Commit to GitHub repository"""
    
    def __init__(self, name: str = 'github', config: Dict = None):
        super().__init__(name, config or {})
        self._repo = config.get('repo')           # 'user/repo'
        self._branch = config.get('branch', 'main')
        self._path = config.get('path', '')       # Path within repo
        self._token = config.get('token')         # Encrypted
        self._create_pr = config.get('create_pr', False)
    
    def validate_config(self) -> bool:
        if not self._repo or '/' not in self._repo:
            return False
        if not self._token:
            return False
        return True
    
    async def test_connection(self) -> bool:
        """Test GitHub API access and repo permissions"""
        try:
            async with aiohttp.ClientSession() as session:
                headers = {
                    'Authorization': f'Bearer {self._token}',
                    'Accept': 'application/vnd.github.v3+json'
                }
                async with session.get(
                    f'https://api.github.com/repos/{self._repo}',
                    headers=headers
                ) as resp:
                    if resp.status != 200:
                        return False
                    data = await resp.json()
                    # Check write permission
                    return data.get('permissions', {}).get('push', False)
        except Exception:
            return False
    
    async def save(self, filename: str, content: str) -> str:
        """Create or update file in repository"""
        full_path = f"{self._path}/{filename}".lstrip('/')
        
        async with aiohttp.ClientSession() as session:
            headers = {
                'Authorization': f'Bearer {self._token}',
                'Accept': 'application/vnd.github.v3+json'
            }
            
            # Check if file exists
            sha = await self._get_file_sha(session, headers, full_path)
            
            # Encode content
            encoded = base64.b64encode(content.encode()).decode()
            
            # Commit message
            message = f"{'Update' if sha else 'Create'} {filename} via DevGuide Cockpit"
            
            if self._create_pr:
                return await self._create_pull_request(
                    session, headers, full_path, encoded, message, sha
                )
            else:
                return await self._direct_commit(
                    session, headers, full_path, encoded, message, sha
                )
    
    async def _get_file_sha(
        self,
        session: aiohttp.ClientSession,
        headers: Dict,
        path: str
    ) -> Optional[str]:
        """Get SHA of existing file"""
        url = f'https://api.github.com/repos/{self._repo}/contents/{path}'
        async with session.get(url, headers=headers, params={'ref': self._branch}) as resp:
            if resp.status == 200:
                data = await resp.json()
                return data.get('sha')
        return None
    
    async def _direct_commit(
        self,
        session: aiohttp.ClientSession,
        headers: Dict,
        path: str,
        content_b64: str,
        message: str,
        sha: Optional[str]
    ) -> str:
        """Commit directly to branch"""
        url = f'https://api.github.com/repos/{self._repo}/contents/{path}'
        
        payload = {
            'message': message,
            'content': content_b64,
            'branch': self._branch
        }
        if sha:
            payload['sha'] = sha
        
        async with session.put(url, headers=headers, json=payload) as resp:
            if resp.status not in (200, 201):
                error = await resp.text()
                raise GitHubAPIError(f"GitHub API error: {error}")
            
            data = await resp.json()
            return data['content']['html_url']
    
    async def _create_pull_request(
        self,
        session: aiohttp.ClientSession,
        headers: Dict,
        path: str,
        content_b64: str,
        message: str,
        sha: Optional[str]
    ) -> str:
        """Create PR with changes"""
        # Create branch
        branch_name = f"devguide-export-{uuid.uuid4().hex[:8]}"
        
        # Get base branch ref
        ref_url = f'https://api.github.com/repos/{self._repo}/git/ref/heads/{self._branch}'
        async with session.get(ref_url, headers=headers) as resp:
            ref_data = await resp.json()
            base_sha = ref_data['object']['sha']
        
        # Create new branch
        create_ref_url = f'https://api.github.com/repos/{self._repo}/git/refs'
        async with session.post(create_ref_url, headers=headers, json={
            'ref': f'refs/heads/{branch_name}',
            'sha': base_sha
        }) as resp:
            if resp.status != 201:
                raise GitHubAPIError("Failed to create branch")
        
        # Commit to new branch
        contents_url = f'https://api.github.com/repos/{self._repo}/contents/{path}'
        payload = {
            'message': message,
            'content': content_b64,
            'branch': branch_name
        }
        if sha:
            payload['sha'] = sha
        
        async with session.put(contents_url, headers=headers, json=payload) as resp:
            if resp.status not in (200, 201):
                raise GitHubAPIError("Failed to commit to branch")
        
        # Create PR
        pr_url = f'https://api.github.com/repos/{self._repo}/pulls'
        async with session.post(pr_url, headers=headers, json={
            'title': f'DevGuide Export: {os.path.basename(path)}',
            'body': f'Automated export from DevGuide Cockpit.\n\n{message}',
            'head': branch_name,
            'base': self._branch
        }) as resp:
            if resp.status != 201:
                raise GitHubAPIError("Failed to create PR")
            
            pr_data = await resp.json()
            return pr_data['html_url']
    
    async def fetch_existing(self, filename: str) -> Optional[str]:
        """Fetch current file content from repo"""
        full_path = f"{self._path}/{filename}".lstrip('/')
        
        async with aiohttp.ClientSession() as session:
            headers = {
                'Authorization': f'Bearer {self._token}',
                'Accept': 'application/vnd.github.v3+json'
            }
            
            url = f'https://api.github.com/repos/{self._repo}/contents/{full_path}'
            async with session.get(url, headers=headers, params={'ref': self._branch}) as resp:
                if resp.status == 200:
                    data = await resp.json()
                    return base64.b64decode(data['content']).decode()
        return None
    
    def get_location_url(self, location: str) -> Optional[str]:
        return location  # Already a URL
```

#### TiDBDestination

**Saves to:** TiDB database table  
**Configuration:** Table name, column mappings, credentials  
**Use Case:** Knowledge base, artifact storage, searchable archives

```python
import aiomysql

class TiDBDestination(ExportDestination):
    """Save to TiDB database table"""
    
    def __init__(self, name: str = 'tidb', config: Dict = None):
        super().__init__(name, config or {})
        self._table = config.get('table', 'exported_artifacts')
        self._host = config.get('host')
        self._port = config.get('port', 4000)
        self._user = config.get('user')
        self._password = config.get('password')  # Encrypted
        self._database = config.get('database')
        self._column_mappings = config.get('column_mappings', {
            'filename': 'filename',
            'content': 'content',
            'metadata': 'metadata'
        })
    
    def validate_config(self) -> bool:
        required = ['host', 'user', 'password', 'database', 'table']
        return all(self.config.get(k) for k in required)
    
    async def test_connection(self) -> bool:
        """Test database connection and table access"""
        try:
            conn = await aiomysql.connect(
                host=self._host,
                port=self._port,
                user=self._user,
                password=self._password,
                db=self._database
            )
            async with conn.cursor() as cur:
                await cur.execute(f"SELECT 1 FROM {self._table} LIMIT 1")
            conn.close()
            return True
        except Exception:
            return False
    
    async def save(self, filename: str, content: str) -> str:
        """Insert or update record in database"""
        artifact_id = str(uuid.uuid4())
        
        conn = await aiomysql.connect(
            host=self._host,
            port=self._port,
            user=self._user,
            password=self._password,
            db=self._database
        )
        
        try:
            async with conn.cursor() as cur:
                # Check if exists (by filename)
                await cur.execute(
                    f"SELECT id FROM {self._table} WHERE filename = %s",
                    (filename,)
                )
                existing = await cur.fetchone()
                
                if existing:
                    # Update
                    await cur.execute(f"""
                        UPDATE {self._table}
                        SET content = %s, updated_at = NOW()
                        WHERE filename = %s
                    """, (content, filename))
                    artifact_id = existing[0]
                else:
                    # Insert
                    await cur.execute(f"""
                        INSERT INTO {self._table} (id, filename, content, created_at)
                        VALUES (%s, %s, %s, NOW())
                    """, (artifact_id, filename, content))
                
                await conn.commit()
        finally:
            conn.close()
        
        return f"tidb://{self._database}/{self._table}/{artifact_id}"
    
    async def fetch_existing(self, filename: str) -> Optional[str]:
        """Fetch existing content by filename"""
        conn = await aiomysql.connect(
            host=self._host,
            port=self._port,
            user=self._user,
            password=self._password,
            db=self._database
        )
        
        try:
            async with conn.cursor() as cur:
                await cur.execute(
                    f"SELECT content FROM {self._table} WHERE filename = %s",
                    (filename,)
                )
                result = await cur.fetchone()
                return result[0] if result else None
        finally:
            conn.close()
    
    def get_location_url(self, location: str) -> Optional[str]:
        return None  # No public URL for database records
```

#### DriveDestination

**Saves to:** Google Drive folder  
**Configuration:** Folder ID, OAuth credentials  
**Use Case:** Team documentation, shared resources

```python
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build
from googleapiclient.http import MediaInMemoryUpload

class DriveDestination(ExportDestination):
    """Save to Google Drive"""
    
    def __init__(self, name: str = 'drive', config: Dict = None):
        super().__init__(name, config or {})
        self._folder_id = config.get('folder_id')
        self._credentials = config.get('oauth_credentials')  # Encrypted JSON
    
    def validate_config(self) -> bool:
        return self._folder_id and self._credentials
    
    async def test_connection(self) -> bool:
        """Test Drive API access and folder permissions"""
        try:
            service = self._get_drive_service()
            # Try to get folder metadata
            folder = service.files().get(
                fileId=self._folder_id,
                fields='id,name,capabilities'
            ).execute()
            return folder.get('capabilities', {}).get('canAddChildren', False)
        except Exception:
            return False
    
    async def save(self, filename: str, content: str) -> str:
        """Create or update file in Drive folder"""
        service = self._get_drive_service()
        
        # Check for existing file
        existing = self._find_file(service, filename)
        
        # Determine MIME type
        mime_type = self._get_mime_type(filename)
        
        media = MediaInMemoryUpload(
            content.encode('utf-8'),
            mimetype=mime_type
        )
        
        if existing:
            # Update
            file = service.files().update(
                fileId=existing['id'],
                media_body=media
            ).execute()
        else:
            # Create
            metadata = {
                'name': filename,
                'parents': [self._folder_id]
            }
            file = service.files().create(
                body=metadata,
                media_body=media,
                fields='id,webViewLink'
            ).execute()
        
        return file.get('webViewLink', f"drive://files/{file['id']}")
    
    async def fetch_existing(self, filename: str) -> Optional[str]:
        """Fetch existing file content"""
        service = self._get_drive_service()
        existing = self._find_file(service, filename)
        
        if existing:
            content = service.files().get_media(fileId=existing['id']).execute()
            return content.decode('utf-8')
        return None
    
    def _get_drive_service(self):
        """Build Drive API service"""
        creds = Credentials.from_authorized_user_info(
            json.loads(self._credentials)
        )
        return build('drive', 'v3', credentials=creds)
    
    def _find_file(self, service, filename: str) -> Optional[Dict]:
        """Find file by name in folder"""
        query = f"name='{filename}' and '{self._folder_id}' in parents and trashed=false"
        results = service.files().list(
            q=query,
            spaces='drive',
            fields='files(id,name)'
        ).execute()
        files = results.get('files', [])
        return files[0] if files else None
    
    def _get_mime_type(self, filename: str) -> str:
        """Determine MIME type from filename"""
        ext = os.path.splitext(filename)[1].lower()
        mime_map = {
            '.md': 'text/markdown',
            '.xml': 'application/xml',
            '.json': 'application/json',
            '.yaml': 'text/yaml',
            '.yml': 'text/yaml',
            '.txt': 'text/plain'
        }
        return mime_map.get(ext, 'text/plain')
    
    def get_location_url(self, location: str) -> Optional[str]:
        if location.startswith('http'):
            return location
        return None
```

### 4.3 Destination Registry

```python
class DestinationRegistry:
    """Registry for managing export destinations"""
    
    def __init__(self, db_client: TiDBClient, crypto: CryptoService):
        self.db = db_client
        self.crypto = crypto
        self._cache: Dict[str, ExportDestination] = {}
    
    async def get(self, destination_id: str) -> Optional[ExportDestination]:
        """Get destination by ID"""
        if destination_id in self._cache:
            return self._cache[destination_id]
        
        # Load from database
        result = await self.db.query_one("""
            SELECT id, name, dest_type, config, is_active
            FROM export_destinations
            WHERE id = %s AND is_active = TRUE
        """, (destination_id,))
        
        if not result:
            return None
        
        # Decrypt sensitive config
        config = json.loads(result['config'])
        config = self._decrypt_config(config)
        
        # Create instance
        dest = self._create_destination(
            result['dest_type'],
            result['name'],
            config
        )
        
        self._cache[destination_id] = dest
        return dest
    
    async def register(
        self,
        name: str,
        dest_type: str,
        config: Dict,
        user_id: str
    ) -> str:
        """Register new destination"""
        destination_id = str(uuid.uuid4())
        
        # Create and validate
        dest = self._create_destination(dest_type, name, config)
        if not dest.validate_config():
            raise DestinationValidationError("Invalid configuration")
        
        # Test connection
        if not await dest.test_connection():
            raise DestinationValidationError("Connection test failed")
        
        # Encrypt sensitive fields
        encrypted_config = self._encrypt_config(config)
        
        # Save to database
        await self.db.execute("""
            INSERT INTO export_destinations (
                id, name, dest_type, config, created_by, created_at, is_active
            ) VALUES (
                %s, %s, %s, %s, %s, NOW(), TRUE
            )
        """, (
            destination_id,
            name,
            dest_type,
            json.dumps(encrypted_config),
            user_id
        ))
        
        return destination_id
    
    def _create_destination(
        self,
        dest_type: str,
        name: str,
        config: Dict
    ) -> ExportDestination:
        """Factory method for destination types"""
        types = {
            'local': LocalFileDestination,
            'github': GitHubDestination,
            'tidb': TiDBDestination,
            'drive': DriveDestination
        }
        
        cls = types.get(dest_type)
        if not cls:
            raise ValueError(f"Unknown destination type: {dest_type}")
        
        return cls(name, config)
    
    def _encrypt_config(self, config: Dict) -> Dict:
        """Encrypt sensitive config fields"""
        sensitive_fields = ['token', 'password', 'oauth_credentials', 'api_key']
        encrypted = config.copy()
        
        for field in sensitive_fields:
            if field in encrypted:
                encrypted[field] = self.crypto.encrypt(encrypted[field])
        
        return encrypted
    
    def _decrypt_config(self, config: Dict) -> Dict:
        """Decrypt sensitive config fields"""
        sensitive_fields = ['token', 'password', 'oauth_credentials', 'api_key']
        decrypted = config.copy()
        
        for field in sensitive_fields:
            if field in decrypted:
                decrypted[field] = self.crypto.decrypt(decrypted[field])
        
        return decrypted
```

#### Database Schema for Destinations

```sql
CREATE TABLE export_destinations (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    dest_type ENUM('local', 'github', 'tidb', 'drive', 'custom') NOT NULL,
    config JSON NOT NULL,
    created_by VARCHAR(36),
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    is_active BOOLEAN DEFAULT TRUE,
    last_used_at TIMESTAMP(6),
    UNIQUE KEY idx_name_user (name, created_by),
    INDEX idx_created_by (created_by),
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 5. The Bin Integration

### 5.1 Export Change Payloads

All export-related mutations must flow through The Bin validation system.

```rust
/// Export-related changes submitted to The Bin
pub enum ExportChange {
    /// Create a new template
    CreateTemplate {
        name: String,
        template_type: TemplateType,
        template_content: String,
        config: Option<serde_json::Value>,
    },
    
    /// Update existing template
    UpdateTemplate {
        id: Uuid,
        changes: TemplateChanges,
    },
    
    /// Create a new destination
    CreateDestination {
        name: String,
        dest_type: DestinationType,
        config: serde_json::Value,
    },
    
    /// Update destination configuration
    UpdateDestination {
        id: Uuid,
        changes: DestinationChanges,
    },
    
    /// Execute export job
    CreateExportJob {
        selection_id: Uuid,
        template_id: Uuid,
        destination_id: Uuid,
        filename: String,
        user_metadata: serde_json::Value,
    },
}

#[derive(Debug, Clone)]
pub enum TemplateType {
    Python,
    Jinja2,
}

#[derive(Debug, Clone)]
pub enum DestinationType {
    Local,
    GitHub,
    TiDB,
    Drive,
    Custom,
}

#[derive(Debug, Clone)]
pub struct TemplateChanges {
    pub name: Option<String>,
    pub template_content: Option<String>,
    pub config: Option<serde_json::Value>,
    pub is_active: Option<bool>,
}

#[derive(Debug, Clone)]
pub struct DestinationChanges {
    pub name: Option<String>,
    pub config: Option<serde_json::Value>,
    pub is_active: Option<bool>,
}
```

### 5.2 Impact Analysis

For each export change type, The Bin computes impact:

```rust
pub struct ExportImpactReport {
    /// What files/records will be created or modified
    pub affected_resources: Vec<AffectedResource>,
    
    /// Estimated token cost for AI processing (if any)
    pub token_cost_estimate: u32,
    
    /// Risk level assessment
    pub risk_level: RiskLevel,
    
    /// Users who might be affected (e.g., shared destinations)
    pub affected_users: Vec<Uuid>,
    
    /// Specific impact details per change type
    pub details: ExportImpactDetails,
}

#[derive(Debug)]
pub enum RiskLevel {
    /// Local save - minimal risk
    Low,
    /// Database insert - medium risk
    Medium,
    /// GitHub push, Drive write - higher risk
    High,
    /// Template affecting multiple users - critical
    Critical,
}

#[derive(Debug)]
pub enum AffectedResource {
    LocalFile { path: String },
    GitHubFile { repo: String, path: String },
    DatabaseRecord { table: String, id: String },
    DriveFile { folder: String, filename: String },
}

impl ExportChange {
    pub fn analyze_impact(&self, ctx: &ValidationContext) -> ExportImpactReport {
        match self {
            ExportChange::CreateExportJob { destination_id, filename, .. } => {
                let dest = ctx.get_destination(destination_id);
                
                let risk_level = match dest.dest_type {
                    DestinationType::Local => RiskLevel::Low,
                    DestinationType::TiDB => RiskLevel::Medium,
                    DestinationType::GitHub | DestinationType::Drive => RiskLevel::High,
                    DestinationType::Custom => RiskLevel::High,
                };
                
                ExportImpactReport {
                    affected_resources: vec![dest.to_affected_resource(filename)],
                    token_cost_estimate: 0,  // Exports don't use AI tokens
                    risk_level,
                    affected_users: dest.get_affected_users(),
                    details: ExportImpactDetails::ExportJob {
                        destination_name: dest.name.clone(),
                        filename: filename.clone(),
                    }
                }
            },
            
            ExportChange::CreateTemplate { name, .. } => {
                ExportImpactReport {
                    affected_resources: vec![],
                    token_cost_estimate: 0,
                    risk_level: RiskLevel::Low,
                    affected_users: vec![],
                    details: ExportImpactDetails::TemplateCreate {
                        template_name: name.clone(),
                    }
                }
            },
            
            // ... other variants
        }
    }
}
```

### 5.3 Validation Rules

```rust
impl ExportChange {
    pub fn validate(&self, ctx: &ValidationContext) -> ValidationResult {
        let mut errors = Vec::new();
        let mut warnings = Vec::new();
        
        match self {
            ExportChange::CreateTemplate { name, template_type, template_content, .. } => {
                // Template must compile/parse
                if let Err(e) = validate_template(template_type, template_content) {
                    errors.push(ValidationError::new(
                        "TEMPLATE_INVALID",
                        format!("Template failed to compile: {}", e)
                    ));
                }
                
                // Name must be unique
                if ctx.template_exists(name) {
                    errors.push(ValidationError::new(
                        "TEMPLATE_NAME_EXISTS",
                        format!("Template '{}' already exists", name)
                    ));
                }
            },
            
            ExportChange::CreateDestination { dest_type, config, .. } => {
                // Configuration must be valid for type
                if let Err(e) = validate_dest_config(dest_type, config) {
                    errors.push(ValidationError::new(
                        "DEST_CONFIG_INVALID",
                        format!("Invalid configuration: {}", e)
                    ));
                }
            },
            
            ExportChange::CreateExportJob { template_id, destination_id, .. } => {
                // Template must exist
                if !ctx.template_exists_by_id(template_id) {
                    errors.push(ValidationError::new(
                        "TEMPLATE_NOT_FOUND",
                        format!("Template {} does not exist", template_id)
                    ));
                }
                
                // Destination must exist
                if !ctx.destination_exists_by_id(destination_id) {
                    errors.push(ValidationError::new(
                        "DESTINATION_NOT_FOUND",
                        format!("Destination {} does not exist", destination_id)
                    ));
                }
                
                // User must have permission for destination
                if !ctx.user_can_use_destination(destination_id) {
                    errors.push(ValidationError::new(
                        "PERMISSION_DENIED",
                        "You do not have permission to use this destination"
                    ));
                }
            },
            
            // ... other variants
        }
        
        ValidationResult {
            passed: errors.is_empty(),
            errors,
            warnings,
        }
    }
}
```

### 5.4 Accept/Reject Workflow

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                     THE BIN - EXPORT REVIEW                     â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                 â”‚
â”‚  âš¡ Pending Export                                              â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€                                              â”‚
â”‚  Template: yaml-markdown                                        â”‚
â”‚  Destination: GitHub (user/docs-repo)                           â”‚
â”‚  Filename: api-analysis.md                                      â”‚
â”‚                                                                 â”‚
â”‚  ðŸ“Š Impact Analysis                                             â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€                                              â”‚
â”‚  â€¢ Risk Level: HIGH (GitHub push)                               â”‚
â”‚  â€¢ File: docs/exports/api-analysis.md                           â”‚
â”‚  â€¢ Action: CREATE (new file)                                    â”‚
â”‚  â€¢ Token Cost: 0                                                â”‚
â”‚                                                                 â”‚
â”‚  âš ï¸ Warnings                                                    â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€                                                       â”‚
â”‚  â€¢ File will be publicly visible in repository                  â”‚
â”‚                                                                 â”‚
â”‚  [View Diff]  [View Content]                                    â”‚
â”‚                                                                 â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”                          â”‚
â”‚  â”‚ REJECT  â”‚              â”‚ ACCEPT  â”‚                          â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                          â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

All changes are logged for audit:

```sql
-- Bin action logged when export is accepted
INSERT INTO audit_logs (
    timestamp, user_id, action, resource_type, resource_id,
    status, metadata
) VALUES (
    NOW(),
    'user_abc',
    'bin_accept_export',
    'export_job',
    'job_xyz',
    'success',
    JSON_OBJECT(
        'template_id', 'tmpl_123',
        'destination_id', 'dest_456',
        'filename', 'api-analysis.md',
        'risk_level', 'high'
    )
);
```

---

## 6. Error Handling

### 6.1 Error Categories

```python
class ExportError(Exception):
    """Base class for export errors"""
    
    def __init__(
        self,
        code: str,
        message: str,
        recoverable: bool = False,
        details: Dict = None
    ):
        self.code = code
        self.message = message
        self.recoverable = recoverable
        self.details = details or {}
        super().__init__(message)


class TemplateError(ExportError):
    """Template-related errors"""
    pass


class ValidationError(ExportError):
    """Validation failures"""
    pass


class DestinationError(ExportError):
    """Destination-related errors"""
    pass


class BinError(ExportError):
    """The Bin validation/commit errors"""
    pass
```

### 6.2 Error Scenarios and Handling

| Stage | Error Type | Code | Message | Recovery |
|-------|-----------|------|---------|----------|
| Assembly | Empty selection | `EMPTY_SELECTION` | "No content selected for export" | Select content |
| Assembly | Message not found | `MESSAGE_NOT_FOUND` | "Selected message no longer exists" | Refresh and re-select |
| Transform | Template not found | `TEMPLATE_NOT_FOUND` | "Template '{name}' not found" | Select different template |
| Transform | Empty content | `EMPTY_CONTENT` | "Cannot export empty content" | Add content |
| Validation | Schema invalid | `SCHEMA_INVALID` | "Content does not match {template} schema" | Fix content or use different template |
| Validation | Bin blocked | `BIN_BLOCKED` | "{reason}" | Address guardrail violation |
| Publish | Connection failed | `CONNECTION_FAILED` | "Could not connect to {destination}" | Retry later |
| Publish | Auth expired | `AUTH_EXPIRED` | "Authentication expired for {destination}" | Re-authenticate |
| Publish | Rate limited | `RATE_LIMITED` | "Rate limit exceeded. Retry after {time}" | Wait and retry |
| Publish | Permission denied | `PERMISSION_DENIED` | "No write access to {location}" | Check permissions |

### 6.3 Rollback Strategy

```python
class RollbackManager:
    """Manages rollback for failed exports"""
    
    async def rollback_publish(
        self,
        job_id: str,
        destination: ExportDestination,
        location: str
    ):
        """Rollback a failed publish operation"""
        
        if isinstance(destination, GitHubDestination):
            # Revert commit or delete PR
            await self._rollback_github(destination, location)
        
        elif isinstance(destination, TiDBDestination):
            # Delete inserted record
            await self._rollback_tidb(destination, location)
        
        elif isinstance(destination, DriveDestination):
            # Delete uploaded file
            await self._rollback_drive(destination, location)
        
        elif isinstance(destination, LocalFileDestination):
            # Delete created file
            await self._rollback_local(destination, location)
        
        # Log rollback
        await self._log_rollback(job_id, location)
    
    async def _rollback_github(self, dest: GitHubDestination, location: str):
        """Revert GitHub commit or close PR"""
        # If location is a PR URL, close it
        # If location is a file URL, create revert commit
        pass
    
    async def _rollback_tidb(self, dest: TiDBDestination, location: str):
        """Delete database record"""
        # Extract record ID from location
        # DELETE FROM table WHERE id = ?
        pass
    
    async def _rollback_drive(self, dest: DriveDestination, location: str):
        """Move file to trash in Drive"""
        # Extract file ID
        # Call Drive API to trash file
        pass
    
    async def _rollback_local(self, dest: LocalFileDestination, location: str):
        """Delete local file"""
        import os
        if os.path.exists(location):
            os.unlink(location)
```

### 6.4 User-Friendly Error Messages

```python
ERROR_MESSAGES = {
    'EMPTY_SELECTION': {
        'title': 'Nothing Selected',
        'message': 'Please select some chat messages or blocks to export.',
        'action': 'Go back and make a selection'
    },
    'TEMPLATE_NOT_FOUND': {
        'title': 'Template Unavailable',
        'message': 'The selected template could not be found. It may have been deleted.',
        'action': 'Choose a different template'
    },
    'SCHEMA_INVALID': {
        'title': 'Format Error',
        'message': 'The content could not be formatted correctly. There may be special characters causing issues.',
        'action': 'Try a different template or simplify the content'
    },
    'CONNECTION_FAILED': {
        'title': 'Connection Failed',
        'message': 'Could not connect to the destination. Please check your internet connection.',
        'action': 'Retry or choose a different destination'
    },
    'AUTH_EXPIRED': {
        'title': 'Sign In Required',
        'message': 'Your authentication has expired. Please sign in again.',
        'action': 'Re-authenticate with the destination'
    },
    'RATE_LIMITED': {
        'title': 'Please Wait',
        'message': 'Too many requests. The destination has temporarily limited access.',
        'action': 'Wait a few minutes and try again'
    },
    'PERMISSION_DENIED': {
        'title': 'Access Denied',
        'message': 'You do not have permission to write to this location.',
        'action': 'Contact the owner or choose a different destination'
    }
}

def format_error_for_user(error: ExportError) -> Dict:
    """Convert error to user-friendly format"""
    template = ERROR_MESSAGES.get(error.code, {
        'title': 'Export Failed',
        'message': error.message,
        'action': 'Please try again'
    })
    
    return {
        'title': template['title'],
        'message': template['message'].format(**error.details),
        'action': template['action'],
        'recoverable': error.recoverable,
        'code': error.code
    }
```

### 6.5 Retry Mechanism

```python
import asyncio
from functools import wraps

def with_retry(
    max_attempts: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 30.0,
    recoverable_errors: tuple = (ConnectionError, TimeoutError)
):
    """Decorator for automatic retry with exponential backoff"""
    
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            last_error = None
            
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except recoverable_errors as e:
                    last_error = e
                    
                    if attempt < max_attempts - 1:
                        delay = min(
                            base_delay * (2 ** attempt),
                            max_delay
                        )
                        await asyncio.sleep(delay)
                except Exception as e:
                    # Non-recoverable error
                    raise
            
            # All retries exhausted
            raise DestinationError(
                code='MAX_RETRIES_EXCEEDED',
                message=f"Failed after {max_attempts} attempts: {last_error}",
                recoverable=False
            )
        
        return wrapper
    return decorator


# Usage
class GitHubDestination(ExportDestination):
    
    @with_retry(max_attempts=3, base_delay=2.0)
    async def save(self, filename: str, content: str) -> str:
        # ... implementation
        pass
```

---

## 7. Audit Trail

### 7.1 Export Jobs Table

```sql
CREATE TABLE export_jobs (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    session_id VARCHAR(36),
    template_id VARCHAR(36) NOT NULL,
    destination_id VARCHAR(36) NOT NULL,
    filename VARCHAR(255) NOT NULL,
    status ENUM('queued', 'running', 'completed', 'failed', 'rolled_back') NOT NULL DEFAULT 'queued',
    content_hash VARCHAR(64),           -- SHA-256 of exported content
    token_count INT,                     -- Tokens in exported content
    output_location TEXT,                -- URL, path, or record ID
    error_message TEXT,                  -- Error details if failed
    bin_transaction_id VARCHAR(36),      -- The Bin transaction
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    started_at TIMESTAMP(6),
    completed_at TIMESTAMP(6),
    
    -- Indexes
    INDEX idx_user (user_id),
    INDEX idx_session (session_id),
    INDEX idx_status (status),
    INDEX idx_created (created_at),
    INDEX idx_destination (destination_id),
    INDEX idx_template (template_id),
    
    -- Foreign keys
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (session_id) REFERENCES chat_sessions(id) ON DELETE SET NULL,
    FOREIGN KEY (template_id) REFERENCES export_templates(id) ON DELETE RESTRICT,
    FOREIGN KEY (destination_id) REFERENCES export_destinations(id) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 7.2 Query Examples

```sql
-- Show all exports to GitHub last week
SELECT 
    ej.id,
    ej.filename,
    ej.status,
    ej.output_location,
    ej.created_at,
    et.name AS template_name,
    ed.name AS destination_name
FROM export_jobs ej
JOIN export_templates et ON ej.template_id = et.id
JOIN export_destinations ed ON ej.destination_id = ed.id
WHERE ed.dest_type = 'github'
  AND ej.created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY)
  AND ej.user_id = ?
ORDER BY ej.created_at DESC;

-- Export statistics by destination
SELECT 
    ed.name AS destination,
    ed.dest_type,
    COUNT(*) AS total_exports,
    SUM(CASE WHEN ej.status = 'completed' THEN 1 ELSE 0 END) AS successful,
    SUM(CASE WHEN ej.status = 'failed' THEN 1 ELSE 0 END) AS failed,
    AVG(TIMESTAMPDIFF(SECOND, ej.started_at, ej.completed_at)) AS avg_duration_sec
FROM export_jobs ej
JOIN export_destinations ed ON ej.destination_id = ed.id
WHERE ej.user_id = ?
  AND ej.created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY ed.id, ed.name, ed.dest_type
ORDER BY total_exports DESC;

-- Failed exports requiring attention
SELECT 
    ej.id,
    ej.filename,
    ej.error_message,
    ej.created_at,
    ed.name AS destination
FROM export_jobs ej
JOIN export_destinations ed ON ej.destination_id = ed.id
WHERE ej.status = 'failed'
  AND ej.user_id = ?
  AND ej.created_at >= DATE_SUB(NOW(), INTERVAL 24 HOUR)
ORDER BY ej.created_at DESC;
```

### 7.3 Audit Integration

Export actions are also logged in the main `audit_logs` table:

```sql
-- Export started
INSERT INTO audit_logs (timestamp, user_id, action, resource_type, resource_id, status, metadata)
VALUES (NOW(), 'user_123', 'export_started', 'export_job', 'job_abc', 'in_progress',
        JSON_OBJECT('template', 'yaml-markdown', 'destination', 'github', 'filename', 'analysis.md'));

-- Export completed
INSERT INTO audit_logs (timestamp, user_id, action, resource_type, resource_id, status, metadata)
VALUES (NOW(), 'user_123', 'export_completed', 'export_job', 'job_abc', 'success',
        JSON_OBJECT('location', 'https://github.com/user/repo/blob/main/analysis.md', 'duration_ms', 1250));

-- Export failed
INSERT INTO audit_logs (timestamp, user_id, action, resource_type, resource_id, status, metadata)
VALUES (NOW(), 'user_123', 'export_failed', 'export_job', 'job_abc', 'error',
        JSON_OBJECT('error_code', 'AUTH_EXPIRED', 'destination', 'github'));
```

---

## 8. Performance Considerations

### 8.1 Async Processing

All pipeline stages run asynchronously to avoid blocking the UI:

```python
class ExportOrchestrator:
    """Main entry point for export operations"""
    
    async def start_export(
        self,
        selection: SelectionSet,
        template_id: str,
        destination_id: str,
        filename: str,
        metadata: Dict,
        user_id: str
    ) -> str:
        """Start export job asynchronously, return job ID"""
        
        job_id = str(uuid.uuid4())
        
        # Create job record immediately
        await self._create_job(job_id, selection, template_id, destination_id, filename, user_id)
        
        # Queue async processing
        asyncio.create_task(
            self._run_pipeline(job_id, selection, template_id, destination_id, filename, metadata, user_id)
        )
        
        return job_id
    
    async def _run_pipeline(self, job_id: str, ...):
        """Execute pipeline stages"""
        try:
            # Stage 1
            await self._update_progress(job_id, 1, 5, "Assembling content...")
            assembled = await self.assembly.execute(selection)
            
            # Stage 2
            await self._update_progress(job_id, 2, 5, "Applying template...")
            formatted = await self.transformation.execute(assembled, template_id, metadata)
            
            # Stage 3
            await self._update_progress(job_id, 3, 5, "Validating...")
            validation = await self.validation.execute(formatted, destination_id)
            
            if not validation.passed:
                await self._fail_job(job_id, validation.errors)
                return
            
            # Stage 4 (preview is synchronous in UI, skip here)
            await self._update_progress(job_id, 4, 5, "Publishing...")
            
            # Stage 5
            result = await self.publish.execute(
                formatted, validation, destination_id, filename, user_id
            )
            
            if result.success:
                await self._complete_job(job_id, result.location)
            else:
                await self._fail_job(job_id, [result.error])
                
        except Exception as e:
            await self._fail_job(job_id, [str(e)])
```

### 8.2 Progress Indicators

```python
@dataclass
class JobProgress:
    job_id: str
    current_stage: int
    total_stages: int
    stage_name: str
    percent_complete: int
    estimated_remaining_ms: Optional[int]

async def _update_progress(
    self,
    job_id: str,
    stage: int,
    total: int,
    message: str
):
    """Update job progress for UI consumption"""
    progress = JobProgress(
        job_id=job_id,
        current_stage=stage,
        total_stages=total,
        stage_name=message,
        percent_complete=int((stage / total) * 100),
        estimated_remaining_ms=None
    )
    
    # Publish via SSE or WebSocket
    await self.event_publisher.publish(
        f'export_progress:{job_id}',
        progress.__dict__
    )
```

### 8.3 Batch Exports

```python
async def batch_export(
    self,
    exports: List[ExportRequest],
    user_id: str
) -> BatchResult:
    """Export multiple files in one operation"""
    
    batch_id = str(uuid.uuid4())
    results = []
    
    for i, export in enumerate(exports):
        # Update batch progress
        await self._update_batch_progress(batch_id, i + 1, len(exports))
        
        try:
            job_id = await self.start_export(
                selection=export.selection,
                template_id=export.template_id,
                destination_id=export.destination_id,
                filename=export.filename,
                metadata=export.metadata,
                user_id=user_id
            )
            
            # Wait for completion (with timeout)
            result = await self._wait_for_job(job_id, timeout=30)
            results.append(result)
            
        except Exception as e:
            results.append(ExportResult(
                success=False,
                filename=export.filename,
                error=str(e)
            ))
    
    return BatchResult(
        batch_id=batch_id,
        total=len(exports),
        successful=sum(1 for r in results if r.success),
        failed=sum(1 for r in results if not r.success),
        results=results
    )
```

### 8.4 Rate Limiting

```python
from collections import defaultdict
import time

class RateLimiter:
    """Per-destination rate limiting"""
    
    LIMITS = {
        'github': {'requests': 5000, 'window': 3600},      # 5000/hour
        'drive': {'requests': 1000, 'window': 100},         # 1000/100s
        'tidb': {'requests': 10000, 'window': 60},          # 10000/min
        'local': {'requests': float('inf'), 'window': 1}    # No limit
    }
    
    def __init__(self):
        self._counters = defaultdict(list)
    
    async def check_and_consume(self, dest_type: str, dest_id: str) -> bool:
        """Check rate limit and consume if allowed"""
        limit = self.LIMITS.get(dest_type, self.LIMITS['local'])
        key = f"{dest_type}:{dest_id}"
        
        now = time.time()
        window_start = now - limit['window']
        
        # Clean old entries
        self._counters[key] = [
            t for t in self._counters[key] if t > window_start
        ]
        
        # Check limit
        if len(self._counters[key]) >= limit['requests']:
            return False
        
        # Consume
        self._counters[key].append(now)
        return True
    
    def get_retry_after(self, dest_type: str, dest_id: str) -> int:
        """Get seconds until rate limit resets"""
        limit = self.LIMITS.get(dest_type, self.LIMITS['local'])
        key = f"{dest_type}:{dest_id}"
        
        if not self._counters[key]:
            return 0
        
        oldest = min(self._counters[key])
        return max(0, int(oldest + limit['window'] - time.time()))
```

### 8.5 Caching

```python
class ContentCache:
    """Cache formatted content to avoid re-transformation"""
    
    def __init__(self, ttl_seconds: int = 300):
        self._cache: Dict[str, CacheEntry] = {}
        self._ttl = ttl_seconds
    
    def get_key(self, selection: SelectionSet, template_id: str) -> str:
        """Generate cache key from selection and template"""
        selection_hash = hashlib.md5(
            json.dumps(sorted(selection.message_ids)).encode()
        ).hexdigest()
        return f"{selection_hash}:{template_id}"
    
    def get(self, key: str) -> Optional[FormattedContent]:
        """Get cached content if still valid"""
        entry = self._cache.get(key)
        if entry and time.time() < entry.expires_at:
            return entry.content
        return None
    
    def set(self, key: str, content: FormattedContent):
        """Cache formatted content"""
        self._cache[key] = CacheEntry(
            content=content,
            expires_at=time.time() + self._ttl
        )
    
    def invalidate(self, session_id: str):
        """Invalidate all cache entries for a session"""
        # When chat content changes, invalidate relevant entries
        keys_to_delete = [
            k for k in self._cache.keys()
            if session_id in k
        ]
        for key in keys_to_delete:
            del self._cache[key]
```

---

## 9. Security Considerations

### 9.1 Credential Encryption

All sensitive credentials are encrypted at rest using AES-256-GCM:

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import base64
import os

class CryptoService:
    """Handles credential encryption/decryption"""
    
    def __init__(self, master_key: bytes):
        """Initialize with 256-bit master key"""
        if len(master_key) != 32:
            raise ValueError("Master key must be 256 bits")
        self._aesgcm = AESGCM(master_key)
    
    def encrypt(self, plaintext: str) -> str:
        """Encrypt and return base64-encoded ciphertext"""
        nonce = os.urandom(12)
        ciphertext = self._aesgcm.encrypt(
            nonce,
            plaintext.encode('utf-8'),
            None  # No additional data
        )
        # Prepend nonce to ciphertext
        return base64.b64encode(nonce + ciphertext).decode('ascii')
    
    def decrypt(self, encrypted: str) -> str:
        """Decrypt base64-encoded ciphertext"""
        data = base64.b64decode(encrypted)
        nonce = data[:12]
        ciphertext = data[12:]
        plaintext = self._aesgcm.decrypt(nonce, ciphertext, None)
        return plaintext.decode('utf-8')
```

### 9.2 Scope Validation

Users can only export to destinations they own or have explicit access to:

```python
async def validate_destination_access(
    user_id: str,
    destination_id: str,
    db: TiDBClient
) -> bool:
    """Check if user can access destination"""
    
    result = await db.query_one("""
        SELECT 
            d.id,
            d.created_by,
            COALESCE(a.access_level, 'none') AS shared_access
        FROM export_destinations d
        LEFT JOIN destination_access a ON d.id = a.destination_id AND a.user_id = %s
        WHERE d.id = %s AND d.is_active = TRUE
    """, (user_id, destination_id))
    
    if not result:
        return False
    
    # Owner always has access
    if result['created_by'] == user_id:
        return True
    
    # Check shared access
    return result['shared_access'] in ('write', 'admin')
```

### 9.3 Content Sanitization

Prevent XSS and injection attacks in templates:

```python
import re
import html

class ContentSanitizer:
    """Sanitize content to prevent injection attacks"""
    
    # Patterns that might indicate injection attempts
    DANGEROUS_PATTERNS = [
        r'<script[^>]*>.*?</script>',          # Script tags
        r'javascript:',                          # JS URLs
        r'on\w+\s*=',                            # Event handlers
        r'data:text/html',                       # Data URLs
    ]
    
    def sanitize_for_template(self, content: str) -> str:
        """Sanitize content before template application"""
        # HTML escape by default
        sanitized = html.escape(content)
        return sanitized
    
    def validate_template_output(self, output: str, template_type: str) -> bool:
        """Validate template output doesn't contain dangerous content"""
        
        for pattern in self.DANGEROUS_PATTERNS:
            if re.search(pattern, output, re.IGNORECASE | re.DOTALL):
                return False
        
        return True
    
    def sanitize_filename(self, filename: str) -> str:
        """Sanitize filename to prevent path traversal"""
        # Remove path separators
        filename = filename.replace('/', '_').replace('\\', '_')
        # Remove null bytes
        filename = filename.replace('\x00', '')
        # Limit to safe characters
        filename = re.sub(r'[^a-zA-Z0-9._-]', '_', filename)
        # Limit length
        return filename[:255]
```

### 9.4 Template Sandboxing

Custom templates execute in a restricted environment:

```python
from RestrictedPython import compile_restricted
from RestrictedPython.Guards import safe_builtins

class TemplateSandbox:
    """Sandbox for executing custom templates"""
    
    ALLOWED_BUILTINS = {
        **safe_builtins,
        'str': str,
        'int': int,
        'float': float,
        'list': list,
        'dict': dict,
        'len': len,
        'range': range,
        'enumerate': enumerate,
        'zip': zip,
        'sorted': sorted,
        'min': min,
        'max': max,
    }
    
    FORBIDDEN_NAMES = [
        '__import__',
        'eval',
        'exec',
        'compile',
        'open',
        'file',
        '__builtins__',
        'globals',
        'locals',
        'vars',
    ]
    
    def execute(self, code: str, context: Dict) -> Any:
        """Execute template code in sandbox"""
        
        # Compile with restrictions
        byte_code = compile_restricted(code, '<template>', 'exec')
        
        # Create restricted namespace
        namespace = {
            '__builtins__': self.ALLOWED_BUILTINS,
            'ExportTemplate': ExportTemplate,
            **context
        }
        
        # Execute
        exec(byte_code, namespace)
        
        return namespace
```

### 9.5 Audit Logging

All security-relevant actions are logged:

```python
SECURITY_EVENTS = [
    'destination_created',
    'destination_config_updated',
    'credential_accessed',
    'export_to_external',
    'template_executed',
    'access_denied',
]

async def log_security_event(
    event_type: str,
    user_id: str,
    resource_type: str,
    resource_id: str,
    details: Dict,
    db: TiDBClient
):
    """Log security-relevant event"""
    
    await db.execute("""
        INSERT INTO security_audit_logs (
            timestamp, event_type, user_id, resource_type, resource_id,
            ip_address, user_agent, details
        ) VALUES (
            NOW(), %s, %s, %s, %s, %s, %s, %s
        )
    """, (
        event_type,
        user_id,
        resource_type,
        resource_id,
        get_client_ip(),
        get_user_agent(),
        json.dumps(details)
    ))
```

---

## 10. API Specification

### 10.1 Export Jobs

#### Create Export Job

```
POST /api/export/jobs
```

**Request:**

```json
{
  "selection": {
    "session_id": "sess_abc123",
    "message_ids": ["msg_1", "msg_2", "msg_3"],
    "block_ids": null
  },
  "template_id": "yaml-markdown",
  "destination_id": "dest_github_123",
  "filename": "api-analysis.md",
  "metadata": {
    "title": "API Analysis Report",
    "author": "John Doe",
    "tags": ["api", "analysis", "q4"]
  }
}
```

**Response (202 Accepted):**

```json
{
  "job_id": "job_xyz789",
  "status": "queued",
  "created_at": "2026-01-02T15:30:00Z",
  "links": {
    "self": "/api/export/jobs/job_xyz789",
    "status": "/api/export/jobs/job_xyz789/status"
  }
}
```

#### Get Export Job

```
GET /api/export/jobs/{id}
```

**Response (200 OK):**

```json
{
  "id": "job_xyz789",
  "status": "completed",
  "template": {
    "id": "yaml-markdown",
    "name": "YAML + Markdown"
  },
  "destination": {
    "id": "dest_github_123",
    "name": "Docs Repository",
    "type": "github"
  },
  "filename": "api-analysis.md",
  "output_location": "https://github.com/user/docs/blob/main/api-analysis.md",
  "statistics": {
    "token_count": 1247,
    "char_count": 5892,
    "duration_ms": 1523
  },
  "created_at": "2026-01-02T15:30:00Z",
  "completed_at": "2026-01-02T15:30:02Z"
}
```

#### List Export Jobs

```
GET /api/export/jobs?status=completed&limit=20&offset=0
```

**Response (200 OK):**

```json
{
  "jobs": [
    {
      "id": "job_xyz789",
      "filename": "api-analysis.md",
      "status": "completed",
      "destination_type": "github",
      "created_at": "2026-01-02T15:30:00Z"
    }
  ],
  "pagination": {
    "total": 47,
    "limit": 20,
    "offset": 0,
    "has_more": true
  }
}
```

### 10.2 Templates

#### List Templates

```
GET /api/export/templates
```

**Response (200 OK):**

```json
{
  "templates": [
    {
      "id": "markdown",
      "name": "Markdown",
      "type": "builtin",
      "description": "Simple Markdown with title header",
      "file_extension": ".md"
    },
    {
      "id": "yaml-markdown",
      "name": "YAML + Markdown",
      "type": "builtin",
      "description": "Markdown with YAML front matter",
      "file_extension": ".md"
    },
    {
      "id": "tmpl_custom_123",
      "name": "My Custom Template",
      "type": "custom",
      "description": "Custom Jinja2 template",
      "file_extension": ".md",
      "created_by": "user_abc"
    }
  ]
}
```

#### Create Template

```
POST /api/export/templates
```

**Request:**

```json
{
  "name": "Technical Spec Template",
  "template_type": "jinja2",
  "template_content": "---\ntitle: {{ metadata.title }}\nversion: {{ metadata.version | default('1.0.0') }}\n---\n\n# {{ metadata.title }}\n\n{{ content }}",
  "config": {
    "required_fields": ["title"]
  }
}
```

**Response (201 Created):**

```json
{
  "id": "tmpl_new_456",
  "name": "Technical Spec Template",
  "type": "custom",
  "created_at": "2026-01-02T16:00:00Z"
}
```

### 10.3 Destinations

#### List Destinations

```
GET /api/export/destinations
```

**Response (200 OK):**

```json
{
  "destinations": [
    {
      "id": "local",
      "name": "Local Downloads",
      "type": "local",
      "builtin": true,
      "status": "active"
    },
    {
      "id": "dest_github_123",
      "name": "Docs Repository",
      "type": "github",
      "builtin": false,
      "status": "active",
      "config_summary": {
        "repo": "user/docs",
        "branch": "main"
      }
    }
  ]
}
```

#### Create Destination

```
POST /api/export/destinations
```

**Request:**

```json
{
  "name": "Team Wiki",
  "dest_type": "drive",
  "config": {
    "folder_id": "1abc123xyz",
    "oauth_credentials": "..."
  }
}
```

**Response (201 Created):**

```json
{
  "id": "dest_drive_789",
  "name": "Team Wiki",
  "type": "drive",
  "status": "active",
  "created_at": "2026-01-02T16:30:00Z"
}
```

#### Test Destination Connection

```
POST /api/export/destinations/{id}/test
```

**Response (200 OK):**

```json
{
  "success": true,
  "latency_ms": 234,
  "permissions": ["read", "write"]
}
```

### 10.4 Preview

#### Generate Preview

```
POST /api/export/preview
```

**Request:**

```json
{
  "selection": {
    "session_id": "sess_abc123",
    "message_ids": ["msg_1", "msg_2"]
  },
  "template_id": "yaml-markdown",
  "destination_id": "dest_github_123",
  "filename": "analysis.md",
  "metadata": {
    "title": "Analysis Report"
  }
}
```

**Response (200 OK):**

```json
{
  "content_preview": "---\ntitle: Analysis Report\nauthor: Unknown\n...",
  "filename": "analysis.md",
  "destination": {
    "name": "Docs Repository",
    "type": "github"
  },
  "is_update": false,
  "diff": null,
  "statistics": {
    "file_size_bytes": 2847,
    "token_count": 523,
    "line_count": 87
  },
  "validation": {
    "passed": true,
    "warnings": []
  }
}
```

### 10.5 Error Responses

All endpoints return consistent error responses:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Content does not match yaml-markdown schema",
    "details": {
      "field": "content",
      "expected": "Valid YAML front matter"
    },
    "recoverable": true
  }
}
```

**HTTP Status Codes:**

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 202 | Accepted (async job started) |
| 400 | Bad Request (validation failed) |
| 401 | Unauthorized |
| 403 | Forbidden (permission denied) |
| 404 | Not Found |
| 429 | Rate Limited |
| 500 | Internal Server Error |

---

## 11. Implementation Notes

### 11.1 Technology Stack

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Template Engine | Python (RestrictedPython) | Safe sandboxing, easy string manipulation |
| Destination Handlers | Go | Efficient async I/O, strong typing for API integrations |
| Pipeline Orchestrator | Go | Async processing, goroutine-based concurrency |
| Job Queue | Redis + Go workers | Reliable async job processing |
| API Layer | Go (Chi/Gin) | Performance, ecosystem maturity |
| Database | TiDB | HTAP capabilities, MySQL compatibility |
| Caching | Redis | Low latency, pub/sub for progress updates |

### 11.2 Service Architecture

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                        API Gateway (Go)                          â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                  â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”              â”‚
â”‚  â”‚   Export    â”‚  â”‚  Template   â”‚  â”‚ Destination â”‚              â”‚
â”‚  â”‚   Service   â”‚  â”‚  Service    â”‚  â”‚  Service    â”‚              â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜              â”‚
â”‚         â”‚                â”‚                â”‚                      â”‚
â”‚         â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                      â”‚
â”‚                          â”‚                                       â”‚
â”‚                   â”Œâ”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”                                â”‚
â”‚                   â”‚    Redis    â”‚                                â”‚
â”‚                   â”‚  (Job Queue)â”‚                                â”‚
â”‚                   â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜                                â”‚
â”‚                          â”‚                                       â”‚
â”‚                   â”Œâ”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”                                â”‚
â”‚                   â”‚   Worker    â”‚                                â”‚
â”‚                   â”‚   Pool      â”‚                                â”‚
â”‚                   â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜                                â”‚
â”‚                          â”‚                                       â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”              â”‚
â”‚  â”‚                       â”‚                       â”‚              â”‚
â”‚  â–¼                       â–¼                       â–¼              â”‚
â”‚ TiDB                  The Bin               Python Sandbox      â”‚
â”‚ (Data)                (Validation)          (Templates)         â”‚
â”‚                                                                  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 11.3 Testing Strategy

#### Unit Tests

```python
# Template validation
def test_yaml_template_validates_correct_output():
    template = YAMLFrontMatterTemplate()
    content = "# Test"
    metadata = {"title": "Test"}
    
    formatted = template.format(content, metadata)
    assert template.validate(formatted) is True

def test_yaml_template_rejects_invalid_yaml():
    template = YAMLFrontMatterTemplate()
    invalid = "---\ntitle: Missing closing\n# Content"
    
    assert template.validate(invalid) is False
```

#### Integration Tests

```python
# Full pipeline test
@pytest.mark.asyncio
async def test_full_export_pipeline():
    pipeline = ExportPipeline(test_config)
    
    # Create test selection
    selection = SelectionSet(
        session_id="test_session",
        message_ids=["msg_1", "msg_2"]
    )
    
    # Execute export
    result = await pipeline.export(
        selection=selection,
        template_id="markdown",
        destination_id="local",
        filename="test.md",
        metadata={"title": "Test Export"}
    )
    
    assert result.success is True
    assert os.path.exists(result.location)
```

#### E2E Tests

```python
# Full user flow test
async def test_export_to_github_e2e():
    async with TestClient(app) as client:
        # Create export job
        response = await client.post("/api/export/jobs", json={
            "selection": {"session_id": "sess_1", "message_ids": ["msg_1"]},
            "template_id": "yaml-markdown",
            "destination_id": "github_test",
            "filename": "test.md",
            "metadata": {"title": "E2E Test"}
        })
        
        assert response.status_code == 202
        job_id = response.json()["job_id"]
        
        # Poll for completion
        for _ in range(10):
            status_response = await client.get(f"/api/export/jobs/{job_id}")
            if status_response.json()["status"] == "completed":
                break
            await asyncio.sleep(1)
        
        assert status_response.json()["status"] == "completed"
        assert "github.com" in status_response.json()["output_location"]
```

### 11.4 Deployment Checklist

- [ ] Database migrations applied
- [ ] Redis configured and accessible
- [ ] Python sandbox environment isolated
- [ ] Master encryption key provisioned (never in code)
- [ ] Rate limits configured per environment
- [ ] Monitoring dashboards created
- [ ] Alert thresholds set for failed exports
- [ ] Backup strategy for export_jobs table
- [ ] Log rotation configured
- [ ] Integration tests passing against staging GitHub/Drive

---

## Appendix A: Database Schema Summary

```sql
-- Core tables
CREATE TABLE export_templates (...);      -- Template definitions
CREATE TABLE export_destinations (...);   -- Destination configurations
CREATE TABLE export_jobs (...);           -- Export job tracking
CREATE TABLE destination_access (...);    -- Shared destination permissions

-- Audit tables
CREATE TABLE audit_logs (...);            -- General audit trail
CREATE TABLE security_audit_logs (...);   -- Security-specific events
```

## Appendix B: Configuration Reference

```yaml
# export_config.yaml
export:
  pipeline:
    max_content_length: 1000000    # 1MB
    default_timeout_ms: 30000
    max_retries: 3
    
  templates:
    sandbox_enabled: true
    cache_ttl_seconds: 300
    
  destinations:
    rate_limits:
      github:
        requests: 5000
        window_seconds: 3600
      drive:
        requests: 1000
        window_seconds: 100
        
  security:
    encryption_algorithm: "AES-256-GCM"
    credential_ttl_hours: 24
```

---

**End of EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md**
