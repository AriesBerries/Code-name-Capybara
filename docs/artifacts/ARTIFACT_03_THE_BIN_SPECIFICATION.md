# The Bin: Synchronization Checkpoint Specification

**Document Type:** Core Architecture  
**Version:** 1.1.0  
**Date:** 2026-01-01T00:00:00Z  
**Dependencies:** TERMINOLOGY_GLOSSARY.md, CROSS_CUTTING_CONCERNS.md, ARTIFACT_05_DATA_MODEL.md, ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md

---

## Overview

**The Bin** is the heart of DevGuide Cockpit's synchronization guarantee. It is simultaneously:
1. **Data Structure** (Rust implementation)
2. **Conceptual Checkpoint** (validation moment)
3. **UI Component** (visual representation)

**Key Guarantee:** ALL changes flow through The Bin - no code path exists to bypass it.

**Version 1.1.0 Additions:**
- Metadata Validation Layer (Layer 0)
- Integration with Metadata Governance Dashboard
- Auto-fix suggestions for common metadata violations

---

## Implementation: Three Aspects

### 1. Data Structure (Rust SyncBuffer)

```rust
pub struct SyncBuffer<T> {
    pending: Vec<PendingChange<T>>,
    validated: Vec<ValidatedChange<T>>,
    rejected: Vec<RejectedChange<T>>,
    state: AtomicState,
}

pub struct PendingChange<T> {
    id: Uuid,
    timestamp: SystemTime,
    source: ChangeSource,
    payload: T,
    impact_analysis: Option<ImpactReport>,
}

pub struct ValidatedChange<T> {
    id: Uuid,
    original: PendingChange<T>,
    validation_results: Vec<GuardrailResult>,
    metadata_validation: MetadataValidationResult,  // NEW in v1.1.0
    approved_by: Option<UserId>,
    approved_at: Option<SystemTime>,
}
```

**Persistence:** Changes persisted to disk for crash recovery (`/var/lib/devguide/syncbuffer.dat`)

### 2. Validation Pipeline

The validation pipeline consists of 7 layers, executed in sequence. Metadata governance validation runs first (Layer 0) to ensure all subsequent validations operate on properly formatted data.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  THE BIN VALIDATION PIPELINE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  → Layer 0: METADATA GOVERNANCE (First Gate)           ← NEW v1.1.0 │
│      ├── Timestamp format validation (Zulu-only)                    │
│      ├── File naming validation (LPDS pattern)                      │
│      ├── Version string validation (SemVer 2.0.0)                   │
│      └── Block on ERROR-level violations                            │
│                                                                     │
│  → Layer 1: GUARDRAIL CHECKS                                        │
│      ├── System guardrails (non-negotiable)                         │
│      ├── Project guardrails (user-configured)                       │
│      └── AI guardrails (reasoning-based)                            │
│                                                                     │
│  → Layer 2: CAPABILITY VERIFICATION                                 │
│      └── Check if change requires capabilities user has granted     │
│                                                                     │
│  → Layer 3: DEPENDENCY IMPACT ANALYSIS                              │
│      ├── Identify affected dashboards                               │
│      ├── Estimate propagation cost (token budget, file writes)      │
│      └── Flag high-impact changes                                   │
│                                                                     │
│  → Layer 4: SPECIFICATION VALIDATION                                │
│      └── Config schema validation                                   │
│                                                                     │
│  → Layer 5: COMMUNITY REVIEW (if applicable)                        │
│      └── Template modifications require community approval          │
│                                                                     │
│  → Layer 6: FINAL COMMIT                                            │
│      └── Atomic database + Git commit                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Pipeline Short-Circuit Rules:**
- Layer 0 (Metadata) ERROR → Block immediately, skip remaining layers
- Layer 0 (Metadata) WARNING/INFO → Continue with warnings attached
- Layer 1 (Guardrail) CRITICAL → Block immediately
- All other layers → Accumulate results, final decision at end

### 3. UI Component (Elm)

**Appears in:** EVERY dashboard (consistent position)

```elm
type alias BinModel =
    { pendingCount : Int
    , validatedCount : Int
    , rejectedCount : Int
    , currentView : BinView
    , selectedChange : Maybe PendingChange
    , metadataViolations : List MetadataViolation  -- NEW in v1.1.0
    }

type BinView
    = Collapsed
    | PendingList
    | ValidatedList
    | RejectedList
    | DetailView PendingChange
    | MetadataViolationsView  -- NEW in v1.1.0

viewBin : BinModel -> Html Msg
viewBin model =
    div [ class "bin-container" ]
        [ viewBinHeader model
        , viewMetadataAlerts model.metadataViolations  -- NEW in v1.1.0
        , viewBinContent model
        ]
```

**Interaction Pattern:**
1. User sees badge notification: "3 pending changes"
2. **NEW:** If metadata violations exist, warning banner appears: "2 metadata issues require attention"
3. Clicks to expand → sees list with impact indicators
4. Selects change → sees validation results + impact analysis + metadata status
5. **NEW:** Can click "Auto-fix" for eligible violations
6. Accepts/rejects → propagation begins or change discarded

---

## Metadata Validation Layer (NEW in v1.1.0)

### Why Layer 0?

Metadata governance validation runs **first** in the pipeline for critical reasons:

1. **Temporal Integrity:** Downstream validations (guardrails, impact analysis) depend on accurate timestamps. Malformed timestamps can corrupt audit logs and break rollback mechanisms.

2. **File Resolution:** Dependency impact analysis relies on proper file naming. Non-compliant names can cause false negatives in dependency graphs.

3. **Version Tracking:** SemVer validation ensures version_history table integrity. Invalid versions break rollback capability.

4. **Fail Fast:** Metadata violations are cheap to detect (regex matching) but expensive to fix later. Blocking early prevents wasted computation.

### MetadataValidator Implementation

```rust
// bin/src/validators/metadata_validator.rs

use std::collections::HashMap;
use std::sync::Arc;
use tokio::sync::RwLock;
use regex::Regex;
use uuid::Uuid;
use chrono::{Utc, DateTime};

/// Cached metadata standard with pre-compiled regex
pub struct CachedStandard {
    pub id: String,
    pub name: String,
    pub category: MetadataCategory,
    pub compiled_regex: Regex,
    pub validation_message: String,
    pub enforcement_level: EnforcementLevel,
}

#[derive(Debug, Clone, PartialEq)]
pub enum MetadataCategory {
    Temporal,
    Nomenclature,
    Versioning,
}

#[derive(Debug, Clone, PartialEq)]
pub enum EnforcementLevel {
    Error,    // Blocks submission
    Warning,  // Allows submission with flag
    Info,     // Informational only
}

#[derive(Debug, Clone)]
pub struct MetadataViolation {
    pub id: Uuid,
    pub standard_id: String,
    pub standard_name: String,
    pub resource_type: String,
    pub resource_id: String,
    pub violation_value: String,
    pub suggested_fix: Option<String>,
    pub enforcement_level: EnforcementLevel,
    pub detected_at: DateTime<Utc>,
}

#[derive(Debug)]
pub struct MetadataValidationResult {
    pub is_valid: bool,
    pub errors: Vec<MetadataViolation>,
    pub warnings: Vec<MetadataViolation>,
    pub info: Vec<MetadataViolation>,
    pub validation_duration_ms: u64,
}

/// Central metadata validator with cached standards
pub struct MetadataValidator {
    /// Pre-compiled standards indexed by ID
    standards: Arc<RwLock<HashMap<String, CachedStandard>>>,
    /// Database client for loading standards and recording violations
    db: TiDBClient,
    /// Cache TTL in seconds (default: 300 = 5 minutes)
    cache_ttl_seconds: u64,
    /// Last cache refresh timestamp
    last_refresh: Arc<RwLock<DateTime<Utc>>>,
}

impl MetadataValidator {
    /// Initialize validator with database connection
    pub async fn new(db: TiDBClient) -> Result<Self, ValidatorError> {
        let validator = Self {
            standards: Arc::new(RwLock::new(HashMap::new())),
            db,
            cache_ttl_seconds: 300,
            last_refresh: Arc::new(RwLock::new(DateTime::UNIX_EPOCH)),
        };
        
        // Load standards on initialization
        validator.refresh_standards_cache().await?;
        
        Ok(validator)
    }
    
    /// Refresh standards from database if cache expired
    pub async fn refresh_standards_cache(&self) -> Result<(), ValidatorError> {
        let now = Utc::now();
        let last = *self.last_refresh.read().await;
        
        if (now - last).num_seconds() < self.cache_ttl_seconds as i64 {
            return Ok(()); // Cache still valid
        }
        
        // Query active standards from database
        let rows = self.db.query(
            "SELECT id, standard_name, category, pattern_regex, 
                    validation_message, enforcement_level 
             FROM metadata_standards 
             WHERE is_active = TRUE"
        ).await?;
        
        let mut new_cache = HashMap::new();
        for row in rows {
            let compiled = Regex::new(&row.pattern_regex)
                .map_err(|e| ValidatorError::InvalidRegex {
                    standard_id: row.id.clone(),
                    error: e.to_string(),
                })?;
            
            new_cache.insert(row.id.clone(), CachedStandard {
                id: row.id,
                name: row.standard_name,
                category: row.category.parse()?,
                compiled_regex: compiled,
                validation_message: row.validation_message,
                enforcement_level: row.enforcement_level.parse()?,
            });
        }
        
        *self.standards.write().await = new_cache;
        *self.last_refresh.write().await = now;
        
        Ok(())
    }
    
    /// Validate a Bin submission against all active metadata standards
    pub async fn validate_submission(&self, item: &BinItem) -> MetadataValidationResult {
        let start = std::time::Instant::now();
        let mut errors = Vec::new();
        let mut warnings = Vec::new();
        let mut info = Vec::new();
        
        // Ensure cache is fresh (non-blocking if within TTL)
        let _ = self.refresh_standards_cache().await;
        
        let standards = self.standards.read().await;
        
        // 1. Validate timestamps
        self.validate_timestamps(item, &standards, &mut errors, &mut warnings, &mut info);
        
        // 2. Validate file names
        self.validate_file_names(item, &standards, &mut errors, &mut warnings, &mut info);
        
        // 3. Validate version strings
        self.validate_versions(item, &standards, &mut errors, &mut warnings, &mut info);
        
        let duration_ms = start.elapsed().as_millis() as u64;
        let is_valid = errors.is_empty();
        
        MetadataValidationResult {
            is_valid,
            errors,
            warnings,
            info,
            validation_duration_ms: duration_ms,
        }
    }
    
    /// Validate all timestamp fields in the submission
    fn validate_timestamps(
        &self,
        item: &BinItem,
        standards: &HashMap<String, CachedStandard>,
        errors: &mut Vec<MetadataViolation>,
        warnings: &mut Vec<MetadataViolation>,
        info: &mut Vec<MetadataViolation>,
    ) {
        let timestamp_standards: Vec<_> = standards.values()
            .filter(|s| s.category == MetadataCategory::Temporal)
            .collect();
        
        for field in item.extract_timestamp_fields() {
            for standard in &timestamp_standards {
                if !standard.compiled_regex.is_match(&field.value) {
                    let violation = MetadataViolation {
                        id: Uuid::new_v4(),
                        standard_id: standard.id.clone(),
                        standard_name: standard.name.clone(),
                        resource_type: "timestamp".to_string(),
                        resource_id: field.name.clone(),
                        violation_value: field.value.clone(),
                        suggested_fix: self.suggest_timestamp_fix(&field.value),
                        enforcement_level: standard.enforcement_level.clone(),
                        detected_at: Utc::now(),
                    };
                    
                    match standard.enforcement_level {
                        EnforcementLevel::Error => errors.push(violation),
                        EnforcementLevel::Warning => warnings.push(violation),
                        EnforcementLevel::Info => info.push(violation),
                    }
                }
            }
        }
    }
    
    /// Validate file names against LPDS pattern
    fn validate_file_names(
        &self,
        item: &BinItem,
        standards: &HashMap<String, CachedStandard>,
        errors: &mut Vec<MetadataViolation>,
        warnings: &mut Vec<MetadataViolation>,
        info: &mut Vec<MetadataViolation>,
    ) {
        let nomenclature_standards: Vec<_> = standards.values()
            .filter(|s| s.category == MetadataCategory::Nomenclature)
            .collect();
        
        for file in item.modified_files() {
            // Apply file naming standard (nm-003)
            if let Some(standard) = nomenclature_standards.iter()
                .find(|s| s.id == "nm-003") 
            {
                if !standard.compiled_regex.is_match(&file.name) {
                    let violation = MetadataViolation {
                        id: Uuid::new_v4(),
                        standard_id: standard.id.clone(),
                        standard_name: standard.name.clone(),
                        resource_type: "file".to_string(),
                        resource_id: file.path.clone(),
                        violation_value: file.name.clone(),
                        suggested_fix: self.suggest_file_name_fix(&file.name),
                        enforcement_level: standard.enforcement_level.clone(),
                        detected_at: Utc::now(),
                    };
                    
                    match standard.enforcement_level {
                        EnforcementLevel::Error => errors.push(violation),
                        EnforcementLevel::Warning => warnings.push(violation),
                        EnforcementLevel::Info => info.push(violation),
                    }
                }
            }
        }
    }
    
    /// Validate version strings against SemVer
    fn validate_versions(
        &self,
        item: &BinItem,
        standards: &HashMap<String, CachedStandard>,
        errors: &mut Vec<MetadataViolation>,
        warnings: &mut Vec<MetadataViolation>,
        info: &mut Vec<MetadataViolation>,
    ) {
        let version_standards: Vec<_> = standards.values()
            .filter(|s| s.category == MetadataCategory::Versioning)
            .collect();
        
        if let Some(version) = item.version() {
            for standard in &version_standards {
                if !standard.compiled_regex.is_match(&version) {
                    let violation = MetadataViolation {
                        id: Uuid::new_v4(),
                        standard_id: standard.id.clone(),
                        standard_name: standard.name.clone(),
                        resource_type: "version".to_string(),
                        resource_id: item.id.to_string(),
                        violation_value: version.clone(),
                        suggested_fix: None, // Version fixes are risky, no auto-suggest
                        enforcement_level: standard.enforcement_level.clone(),
                        detected_at: Utc::now(),
                    };
                    
                    match standard.enforcement_level {
                        EnforcementLevel::Error => errors.push(violation),
                        EnforcementLevel::Warning => warnings.push(violation),
                        EnforcementLevel::Info => info.push(violation),
                    }
                }
            }
        }
    }
    
    /// Generate auto-fix suggestion for timestamp violations
    fn suggest_timestamp_fix(&self, value: &str) -> Option<String> {
        // Attempt to parse and convert to Zulu format
        if let Ok(parsed) = chrono::DateTime::parse_from_rfc3339(value) {
            let zulu = parsed.with_timezone(&Utc);
            return Some(zulu.format("%Y-%m-%dT%H:%M:%S%.6fZ").to_string());
        }
        
        // Try common non-Zulu formats
        let formats = [
            "%Y-%m-%d %H:%M:%S",           // 2026-01-01 14:30:45
            "%Y-%m-%dT%H:%M:%S",           // 2026-01-01T14:30:45 (no Z)
            "%Y-%m-%dT%H:%M:%S%z",         // 2026-01-01T14:30:45+00:00
            "%d/%m/%Y %H:%M:%S",           // 01/01/2026 14:30:45
        ];
        
        for fmt in formats {
            if let Ok(dt) = chrono::NaiveDateTime::parse_from_str(value, fmt) {
                let utc = DateTime::<Utc>::from_naive_utc_and_offset(dt, Utc);
                return Some(utc.format("%Y-%m-%dT%H:%M:%S%.6fZ").to_string());
            }
        }
        
        None
    }
    
    /// Generate auto-fix suggestion for file naming violations
    fn suggest_file_name_fix(&self, name: &str) -> Option<String> {
        // Extract components from common patterns
        let parts: Vec<&str> = name
            .trim_end_matches(|c: char| c == '.' || c.is_alphanumeric())
            .split(&['-', '_', '.'][..])
            .filter(|s| !s.is_empty())
            .collect();
        
        if parts.len() >= 2 {
            // Attempt to construct LPDS pattern: domain_type-variant.ext
            let extension = name.rsplit('.').next().unwrap_or("rs");
            let domain = parts.get(0).unwrap_or(&"module");
            let type_name = parts.get(1).unwrap_or(&"handler");
            
            return Some(format!(
                "{}_{}-default_proc-production_ver-1.0.0.{}",
                domain.to_lowercase(),
                type_name.to_lowercase(),
                extension
            ));
        }
        
        None
    }
    
    /// Record violations in the database for audit and dashboard display
    pub async fn record_violations(
        &self,
        violations: &[MetadataViolation],
    ) -> Result<(), ValidatorError> {
        if violations.is_empty() {
            return Ok(());
        }
        
        for violation in violations {
            self.db.execute(
                "INSERT INTO metadata_violations 
                 (id, standard_id, resource_type, resource_id, violation_value, 
                  suggested_fix, status, detected_at)
                 VALUES (?, ?, ?, ?, ?, ?, 'open', ?)",
                &[
                    &violation.id.to_string(),
                    &violation.standard_id,
                    &violation.resource_type,
                    &violation.resource_id,
                    &violation.violation_value,
                    &violation.suggested_fix,
                    &violation.detected_at.to_rfc3339(),
                ],
            ).await?;
        }
        
        Ok(())
    }
}
```

### Validation Flow in Bin Submission Pipeline

```rust
// bin/src/pipeline.rs

impl BinPipeline {
    /// Execute full validation pipeline for a submission
    pub async fn validate(&self, item: &BinItem) -> PipelineResult {
        let mut result = PipelineResult::new();
        
        // ═══════════════════════════════════════════════════════════════
        // LAYER 0: METADATA GOVERNANCE (FIRST GATE)
        // ═══════════════════════════════════════════════════════════════
        let metadata_result = self.metadata_validator.validate_submission(item).await;
        
        // Record all violations for dashboard visibility
        let all_violations: Vec<_> = metadata_result.errors.iter()
            .chain(metadata_result.warnings.iter())
            .chain(metadata_result.info.iter())
            .cloned()
            .collect();
        
        if !all_violations.is_empty() {
            self.metadata_validator.record_violations(&all_violations).await?;
        }
        
        // Block on ERROR-level violations
        if !metadata_result.is_valid {
            return PipelineResult::Blocked {
                layer: ValidationLayer::MetadataGovernance,
                reason: "Metadata validation failed".to_string(),
                violations: metadata_result.errors,
                warnings: metadata_result.warnings,
            };
        }
        
        // Attach warnings for downstream display
        result.metadata_warnings = metadata_result.warnings;
        result.metadata_info = metadata_result.info;
        
        // ═══════════════════════════════════════════════════════════════
        // LAYER 1: GUARDRAIL CHECKS
        // ═══════════════════════════════════════════════════════════════
        let guardrail_result = self.validate_guardrails(item).await?;
        if guardrail_result.has_critical_violations() {
            return PipelineResult::Blocked {
                layer: ValidationLayer::Guardrails,
                reason: guardrail_result.blocking_reason(),
                violations: guardrail_result.violations,
                warnings: result.metadata_warnings,
            };
        }
        result.guardrail_results = guardrail_result;
        
        // ═══════════════════════════════════════════════════════════════
        // LAYER 2: CAPABILITY VERIFICATION
        // ═══════════════════════════════════════════════════════════════
        let capability_result = self.verify_capabilities(item).await?;
        if !capability_result.all_granted {
            return PipelineResult::Blocked {
                layer: ValidationLayer::Capabilities,
                reason: format!(
                    "Missing capabilities: {:?}",
                    capability_result.missing
                ),
                violations: vec![],
                warnings: result.metadata_warnings,
            };
        }
        
        // ═══════════════════════════════════════════════════════════════
        // LAYER 3: DEPENDENCY IMPACT ANALYSIS
        // ═══════════════════════════════════════════════════════════════
        let impact = self.analyze_impact(item).await?;
        result.impact_analysis = impact;
        
        // Continue to remaining layers...
        // (Layers 4-6 follow similar pattern)
        
        PipelineResult::Passed(result)
    }
}
```

---

## Error Messaging for Violations

### User-Facing Error Messages

Metadata violation messages are designed to be **actionable** without overwhelming users with technical details.

| Violation Type | User Message | Details (Expandable) |
|----------------|--------------|----------------------|
| Timestamp (ERROR) | "Timestamp format invalid" | "Value `2026-01-01 14:30:45` should be `2026-01-01T14:30:45Z` (ISO 8601 with Zulu suffix)" |
| File Name (WARNING) | "File name doesn't follow standard" | "Consider renaming `util.ts` to `dashboard_util-helper_proc-production_ver-1.0.0.ts`" |
| Version (ERROR) | "Version format invalid" | "Value `1.2` must follow Semantic Versioning (e.g., `1.2.0`)" |

### Error Message Structure

```rust
pub struct UserFacingError {
    pub severity: Severity,
    pub title: String,
    pub description: String,
    pub current_value: String,
    pub expected_format: String,
    pub suggested_fix: Option<String>,
    pub can_auto_fix: bool,
    pub documentation_link: String,
}

impl From<MetadataViolation> for UserFacingError {
    fn from(v: MetadataViolation) -> Self {
        Self {
            severity: match v.enforcement_level {
                EnforcementLevel::Error => Severity::Error,
                EnforcementLevel::Warning => Severity::Warning,
                EnforcementLevel::Info => Severity::Info,
            },
            title: get_friendly_title(&v.standard_name),
            description: get_friendly_description(&v),
            current_value: v.violation_value,
            expected_format: get_expected_format(&v.standard_id),
            suggested_fix: v.suggested_fix,
            can_auto_fix: is_safe_auto_fix(&v.standard_id),
            documentation_link: format!(
                "https://docs.devguide.io/metadata-standards#{}",
                v.standard_id
            ),
        }
    }
}

fn get_friendly_title(standard_name: &str) -> String {
    match standard_name {
        "api_timestamp_format" => "Timestamp Format Invalid".to_string(),
        "file_naming_lpds" => "File Name Non-Standard".to_string(),
        "semver_format" => "Version Format Invalid".to_string(),
        _ => format!("Metadata Standard: {}", standard_name),
    }
}

fn get_expected_format(standard_id: &str) -> String {
    match standard_id {
        "ts-001" => "2026-01-01T14:30:45.123456Z (ISO 8601 with Z suffix)".to_string(),
        "ts-002" => "20260101T143045Z (Basic ISO 8601, no punctuation)".to_string(),
        "nm-003" => "domain_type-variant_proc-stage_ver-semver.ext (LPDS)".to_string(),
        "vs-001" => "1.2.3 or 1.2.3-beta.1 (Semantic Versioning 2.0.0)".to_string(),
        _ => "See documentation".to_string(),
    }
}

fn is_safe_auto_fix(standard_id: &str) -> bool {
    match standard_id {
        "ts-001" | "ts-002" | "ts-003" => true,  // Timestamp conversion is safe
        "nm-003" => false,  // File rename needs user confirmation
        "vs-001" | "vs-002" => false,  // Version bump is risky
        _ => false,
    }
}
```

---

## Auto-Fix Strategy

### Safe Auto-Fixes (Apply Automatically)

| Violation | Auto-Fix | Safety Rationale |
|-----------|----------|------------------|
| `ts-001`: Non-Zulu timestamp | Convert to UTC with Z suffix | Deterministic conversion, no data loss |
| `ts-002`: Punctuated file timestamp | Remove punctuation, add Z | Deterministic format change |
| `ts-003`: Missing timezone | Assume UTC, add Z suffix | Safe assumption for server-side times |

### Suggested Fixes (User Confirmation Required)

| Violation | Suggestion | Why User Must Confirm |
|-----------|------------|----------------------|
| `nm-003`: Non-LPDS file name | Propose LPDS-compliant name | Renames affect imports, tests, documentation |
| `vs-001`: Invalid SemVer | Show corrected format | Version has semantic meaning |

### Risky (No Auto-Fix)

| Violation | No Auto-Fix | Risk |
|-----------|-------------|------|
| `vs-002`: Missing build metadata | None | Could overwrite intentional build tags |
| Branch naming | None | Git history integrity |

### Auto-Fix Implementation

```rust
// bin/src/autofix.rs

pub struct AutoFixer {
    metadata_validator: Arc<MetadataValidator>,
}

#[derive(Debug, Clone)]
pub struct AutoFixResult {
    pub original: String,
    pub fixed: String,
    pub standard_id: String,
    pub confidence: FixConfidence,
}

#[derive(Debug, Clone)]
pub enum FixConfidence {
    High,      // Safe to apply automatically
    Medium,    // Show preview, require one-click confirm
    Low,       // Show preview, require explicit confirmation
}

impl AutoFixer {
    /// Attempt to auto-fix violations in a Bin submission
    pub async fn fix_submission(
        &self,
        item: &mut BinItem,
        violations: &[MetadataViolation],
    ) -> Vec<AutoFixResult> {
        let mut results = Vec::new();
        
        for violation in violations {
            if let Some(suggested_fix) = &violation.suggested_fix {
                let confidence = self.assess_fix_confidence(&violation.standard_id);
                
                match confidence {
                    FixConfidence::High => {
                        // Apply automatically
                        if self.apply_fix(item, violation, suggested_fix) {
                            results.push(AutoFixResult {
                                original: violation.violation_value.clone(),
                                fixed: suggested_fix.clone(),
                                standard_id: violation.standard_id.clone(),
                                confidence,
                            });
                        }
                    }
                    FixConfidence::Medium | FixConfidence::Low => {
                        // Return as suggestion only
                        results.push(AutoFixResult {
                            original: violation.violation_value.clone(),
                            fixed: suggested_fix.clone(),
                            standard_id: violation.standard_id.clone(),
                            confidence,
                        });
                    }
                }
            }
        }
        
        results
    }
    
    fn assess_fix_confidence(&self, standard_id: &str) -> FixConfidence {
        match standard_id {
            "ts-001" | "ts-002" | "ts-003" => FixConfidence::High,
            "nm-003" => FixConfidence::Medium,
            _ => FixConfidence::Low,
        }
    }
    
    fn apply_fix(
        &self,
        item: &mut BinItem,
        violation: &MetadataViolation,
        fix: &str,
    ) -> bool {
        match violation.resource_type.as_str() {
            "timestamp" => {
                item.update_timestamp_field(&violation.resource_id, fix);
                true
            }
            _ => false,
        }
    }
}
```

---

## Integration with Metadata Governance Dashboard

### Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    METADATA GOVERNANCE DATA FLOW                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────┐                                           │
│   │ metadata_standards  │◄────── Admin creates/edits standards      │
│   │ (Source of Truth)   │                                           │
│   └──────────┬──────────┘                                           │
│              │                                                      │
│              ▼ Cache on startup (5-min TTL)                         │
│   ┌─────────────────────┐                                           │
│   │  MetadataValidator  │◄────── The Bin loads standards            │
│   │  (Rust WASM)        │                                           │
│   └──────────┬──────────┘                                           │
│              │                                                      │
│              ▼ Validate every submission                            │
│   ┌─────────────────────┐                                           │
│   │ BinItem Submission  │                                           │
│   └──────────┬──────────┘                                           │
│              │                                                      │
│              ▼ On violation                                         │
│   ┌─────────────────────┐                                           │
│   │ metadata_violations │────────► Dashboard displays violations    │
│   │ (Audit Trail)       │                                           │
│   └──────────┬──────────┘                                           │
│              │                                                      │
│              ▼ User action                                          │
│   ┌─────────────────────┐                                           │
│   │  Auto-Fix / Manual  │────────► Update status to 'resolved'      │
│   └─────────────────────┘                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### API Endpoints for Dashboard Integration

```go
// api/handlers/metadata_violations.go

// GET /api/v1/metadata/violations
// Returns open violations for current project
func GetMetadataViolations(c *gin.Context) {
    projectID := c.GetString("project_id")
    status := c.DefaultQuery("status", "open")
    
    violations, err := db.Query(`
        SELECT v.id, v.standard_id, s.standard_name, v.resource_type,
               v.resource_id, v.violation_value, v.suggested_fix,
               s.enforcement_level, v.detected_at
        FROM metadata_violations v
        JOIN metadata_standards s ON v.standard_id = s.id
        WHERE v.status = ? AND v.resource_id LIKE ?
        ORDER BY v.detected_at DESC
    `, status, projectID+"%")
    
    c.JSON(200, violations)
}

// POST /api/v1/metadata/violations/:id/resolve
// Marks a violation as resolved (manual fix)
func ResolveViolation(c *gin.Context) {
    violationID := c.Param("id")
    userID := c.GetString("user_id")
    
    db.Exec(`
        UPDATE metadata_violations 
        SET status = 'resolved', 
            resolved_at = CURRENT_TIMESTAMP(6),
            resolved_by = ?
        WHERE id = ?
    `, userID, violationID)
    
    c.JSON(200, gin.H{"status": "resolved"})
}

// POST /api/v1/metadata/violations/:id/suppress
// Suppresses a violation (false positive)
func SuppressViolation(c *gin.Context) {
    violationID := c.Param("id")
    userID := c.GetString("user_id")
    reason := c.PostForm("reason")
    
    // Log suppression for audit
    db.Exec(`
        UPDATE metadata_violations 
        SET status = 'suppressed', 
            resolved_at = CURRENT_TIMESTAMP(6),
            resolved_by = ?
        WHERE id = ?
    `, userID, violationID)
    
    // Audit log entry
    db.Exec(`
        INSERT INTO audit_logs (timestamp, user_id, action, resource_type, 
                               resource_id, status, metadata)
        VALUES (CURRENT_TIMESTAMP(6), ?, 'suppress_violation', 'metadata_violation',
                ?, 'success', ?)
    `, userID, violationID, fmt.Sprintf(`{"reason": "%s"}`, reason))
    
    c.JSON(200, gin.H{"status": "suppressed"})
}

// POST /api/v1/metadata/violations/:id/autofix
// Applies auto-fix to a violation
func ApplyAutoFix(c *gin.Context) {
    violationID := c.Param("id")
    
    violation, _ := db.QueryRow(`
        SELECT suggested_fix, resource_type, resource_id 
        FROM metadata_violations WHERE id = ?
    `, violationID)
    
    if violation.SuggestedFix == "" {
        c.JSON(400, gin.H{"error": "No auto-fix available"})
        return
    }
    
    // Apply the fix (implementation depends on resource_type)
    err := applyFixToResource(violation)
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }
    
    // Mark as resolved
    db.Exec(`
        UPDATE metadata_violations 
        SET status = 'resolved', resolved_at = CURRENT_TIMESTAMP(6)
        WHERE id = ?
    `, violationID)
    
    c.JSON(200, gin.H{"status": "fixed", "new_value": violation.SuggestedFix})
}
```

### User Workflow for Resolving Violations

1. **Discovery:** User submits change via The Bin
2. **Validation:** MetadataValidator runs, finds violations
3. **Notification:** 
   - ERROR violations → Submission blocked, user sees error banner
   - WARNING violations → Submission allowed, warning badge shown
4. **Resolution Options:**
   - **Auto-Fix Button** (if available): One-click apply suggested fix
   - **Manual Fix:** User edits source, resubmits
   - **Suppress:** False positive, mark as suppressed with reason
5. **Completion:** Violation status updated in `metadata_violations` table
6. **Dashboard Update:** Real-time compliance score recalculated

---

## Guardrail System

### System Guardrails (Non-Negotiable)

```yaml
system_guardrails:
  - id: no_plaintext_secrets
    description: "No plaintext passwords, API keys, or tokens in config"
    severity: CRITICAL
    auto_reject: true
    
  - id: wcag_aa_compliance
    description: "All UI changes must pass WCAG AA accessibility"
    severity: HIGH
    auto_reject: false  # Warning only
    
  - id: performance_budget
    description: "Dashboard load time must be < 1s"
    severity: MEDIUM
    auto_reject: false
```

### Project Guardrails (User-Configured)

```yaml
project_guardrails:
  - id: test_coverage_minimum
    description: "Test coverage must be ≥ 80%"
    severity: HIGH
    threshold: 80
    auto_reject: true
    
  - id: no_deployment_without_review
    description: "Deployment requires at least 1 code review approval"
    severity: CRITICAL
    auto_reject: true
```

### AI Guardrails (Reasoning-Based)

```yaml
ai_guardrails:
  - id: single_responsibility_check
    description: "AI validates if change violates SRP"
    prompt: "Does this change make a module/function responsible for >1 thing?"
    severity: MEDIUM
    auto_reject: false
```

---

## Dependency Impact Analysis

**Algorithm:**

```rust
pub fn analyze_impact(change: &PendingChange) -> ImpactReport {
    let affected_dashboards = find_dependent_dashboards(change);
    let token_cost = estimate_ai_token_consumption(&affected_dashboards);
    let file_writes = count_file_modifications(&affected_dashboards);
    let criticality = assess_criticality(change);
    
    ImpactReport {
        affected_dashboards,
        estimated_token_cost: token_cost,
        estimated_file_writes: file_writes,
        criticality_level: criticality,
        recommendation: compute_recommendation(token_cost, file_writes, criticality),
    }
}
```

**Recommendation Logic:**

| Token Cost | File Writes | Criticality | Recommendation |
|------------|-------------|-------------|----------------|
| < 1K | < 10 | LOW | Auto-approve safe |
| 1K-10K | 10-100 | MEDIUM | Review suggested |
| > 10K | > 100 | HIGH | Manual review required |

---

## Propagation Mechanism

**After User Accepts Change:**

```
1. Mark change as VALIDATED in SyncBuffer
2. Acquire propagation lock (prevents concurrent propagations)
3. Execute propagation plan:
   a. Update source artifact
   b. Trigger AI regeneration (if needed)
   c. Update dependent artifacts
   d. Commit to Git (if enabled)
   e. Persist to TiDB
4. Release propagation lock
5. Send SSE notification to all active dashboards
6. Update Bin UI: move from "pending" to "completed"
```

**Rollback Capability:**

```rust
pub fn rollback_propagation(change_id: Uuid) -> Result<(), RollbackError> {
    // Retrieve propagation record
    let propagation = get_propagation_record(change_id)?;
    
    // Git revert (if change was committed)
    if let Some(commit_sha) = propagation.git_commit {
        git_revert(commit_sha)?;
    }
    
    // TiDB transaction rollback
    rollback_database_transaction(propagation.transaction_id)?;
    
    // Restore IndexedDB cache
    restore_cached_state(propagation.previous_state)?;
    
    Ok(())
}
```

**Rollback Window:** 30 days (configurable)

---

## Performance Requirements

| Operation | Target | Max | Measurement |
|-----------|--------|-----|-------------|
| Add to Bin | < 10ms | 50ms | Rust microbenchmark |
| **Metadata Validation** | < 50ms | 100ms | Regex matching (v1.1.0) |
| Guardrail Validation | < 150ms | 400ms | E2E test |
| Full Validation | < 200ms | 500ms | E2E test |
| Propagation | < 1s | 3s | Integration test |

**Optimization Strategies (v1.1.0):**
- Pre-compiled regex patterns cached in memory
- Parallel validation using Rust async tasks
- Standards cache with 5-minute TTL (avoids DB round-trips)
- Timeout at 100ms for metadata validation (fail open with warning)

---

## Error Handling

**Validation Errors:**

```rust
pub enum BinError {
    GuardrailViolation { rule: String, details: String },
    CapabilityDenied { required: Capability, reason: String },
    DependencyConflict { conflicting_modules: Vec<String> },
    TokenBudgetExceeded { limit: u32, requested: u32 },
    PropagationFailed { stage: PropagationStage, error: String },
    // NEW in v1.1.0
    MetadataViolation { violations: Vec<MetadataViolation> },
    MetadataValidatorUnavailable { reason: String },
}
```

**User-Facing Error Messages:**

| Error Type | Message | Recovery Action |
|------------|---------|-----------------|
| GuardrailViolation | "Change violates: {rule}" | Modify change to comply |
| CapabilityDenied | "Action requires: {capability}" | Grant capability in settings |
| TokenBudgetExceeded | "Change would use {tokens} tokens (limit: {limit})" | Reduce scope or wait for reset |
| **MetadataViolation** | "Metadata validation failed: {count} issues" | Fix violations or use auto-fix (v1.1.0) |

---

## Offline Behavior

**Offline Mode:**
- Changes queue in IndexedDB
- Validation deferred until online
- User sees "Pending Sync" indicator
- **NEW (v1.1.0):** Basic metadata validation runs locally using cached standards

**Sync on Reconnect:**
```
1. Retrieve queued changes from IndexedDB
2. Submit to Go API in batch
3. API validates each change (including metadata layer)
4. Return results (validated/rejected)
5. Clear IndexedDB queue
6. Update UI
```

---

## Audit Trail

**Every Bin action logged:**

```sql
INSERT INTO audit_logs (
    timestamp, user_id, action, resource_type, resource_id, 
    status, metadata
) VALUES (
    CURRENT_TIMESTAMP(6), 'user_123', 'bin_accept_change', 'dashboard_config', 'cfg_abc',
    'success', '{"change_id": "change_xyz", "impact": "3 dashboards", "metadata_warnings": 2}'
);
```

**NEW in v1.1.0:** Metadata violation actions also logged:
- `metadata_violation_detected`
- `metadata_violation_autofix_applied`
- `metadata_violation_manually_resolved`
- `metadata_violation_suppressed`

---

## Integration with AI Layer

**AI Participation in Validation:**

```rust
pub async fn validate_with_ai(change: &PendingChange) -> Result<AIValidationResult, AIError> {
    let context = build_context_for_ai(change);
    let prompt = format!(
        "Analyze this change for potential issues:\n{}\n\n\
         Consider: single responsibility, maintainability, security",
        context
    );
    
    let ai_response = ai_provider.reason(prompt, TokenBudget::from(4000)).await?;
    parse_validation_result(ai_response)
}
```

**AI Result Integration:**
- AI warnings shown in Bin detail view
- User can override AI recommendation
- AI rationale logged for audit

---

## Open Questions

1. ~~**Batch Validation:** Should users be able to accept multiple changes at once?~~
   - **Resolved (v1.1.0):** Yes, with aggregated metadata validation.
2. **Undo Depth:** 30-day rollback sufficient or configurable?
3. **Conflict Resolution:** How to handle concurrent changes to same artifact?
4. **NEW (v1.1.0):** Should metadata validation timeout fail-open or fail-closed?
   - Current implementation: Fail-open with warning (prioritizes user productivity)

---

## Changelog

### v1.1.0 (2026-01-01T00:00:00Z)

**Added:**
- Metadata Validation Layer (Layer 0 in pipeline)
- MetadataValidator struct with cached standards
- Integration with `metadata_standards` and `metadata_violations` tables
- Auto-fix suggestions for timestamp violations
- User-facing error messages with actionable suggestions
- API endpoints for violation management
- Performance requirements for metadata validation
- Audit logging for metadata violation actions

**Changed:**
- Validation pipeline now has 7 layers (was 4)
- BinModel includes `metadataViolations` field
- ValidatedChange includes `metadata_validation` field

**Performance:**
- Metadata validation target: <50ms (p99)
- Standards cache TTL: 5 minutes

---

**End of THE_BIN_SPECIFICATION.md v1.1.0**
