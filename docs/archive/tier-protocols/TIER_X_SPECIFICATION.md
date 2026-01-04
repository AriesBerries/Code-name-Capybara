# TIER X - Vision Synthesis & UX Specification
## External LLM Pre-Requisite Tasks (Complete Before TIER 3)

**Created:** 2026-01-02T21:00:00Z  
**Priority:** 🔥 CRITICAL - Must complete before TIER 3 execution  
**Purpose:** Synthesize project owner's vision into comprehensive specifications for external LLM  
**Estimated Time:** 8-12 hours (external LLM work)  
**Deliverables:** 5 comprehensive specification documents

---

## 📋 OVERVIEW

**What is TIER X?**

TIER X is a **pre-TIER 3 synthesis phase** where an external Large Language Model (GPT-4, Claude Opus, Gemini Pro, etc.) creates comprehensive specification documents based on:

1. Project owner's vision materials (uploaded documents)
2. Visual mockups (images + video + GIF)
3. Progressive disclosure philosophy
4. Layout Edit Mode requirements
5. Tri-surface chat workbench architecture

**Why complete this first?**

The external LLM will produce **detailed UX specifications** and **implementation blueprints** that become the **authoritative reference** for TIER 3 execution. This ensures:

✅ Project owner's vision is fully captured  
✅ UX patterns are comprehensively documented  
✅ Implementation teams have clear requirements  
✅ No ambiguity during TIER 3 execution  
✅ Consistent terminology across all artifacts

**Output:** 5 polished specification documents ready to hand to development teams

---

## 📦 TIER X DELIVERABLES

### 1. COMPOSER_UX_COMPLETE_SPECIFICATION.md
**Purpose:** Complete composer UX with all 7 progressive disclosure phases  
**Size:** 15-20 pages  
**Audience:** Frontend engineers, UX designers  

**Must Include:**
- All 7 progressive disclosure phases (detailed)
- Component inventory (every button, toggle, chip)
- Exact icon order and placement
- Keyboard shortcuts (complete list)
- Accessibility requirements (WCAG 2.1 AA)
- Animation specifications
- Responsive behavior
- Error states and edge cases
- Example user flows (5-7 scenarios)

---

### 2. LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md
**Purpose:** Full drag-and-drop customization system  
**Size:** 10-12 pages  
**Audience:** Frontend engineers, UX designers

**Must Include:**
- 3 Rails system (Left/Right/Secondary) with constraints
- Draggable component inventory
- Snap-to-slot behavior specification
- Keyboard controls (complete list)
- Invalid placement rules
- Hidden tray mechanism
- Save/Cancel/Reset behavior
- Layout persistence schema (JSON/YAML)
- Import/export functionality
- Default layouts (3-5 presets)
- Migration strategy for existing users

---

### 3. TRI_SURFACE_WORKBENCH_COMPLETE_SPECIFICATION.md
**Purpose:** Full tri-surface architecture with database-mediated communication  
**Size:** 20-25 pages  
**Audience:** Full-stack engineers, architects

**Must Include:**
- Surface switching UX (pills, sliding, collapsed states)
- Inputs Surface complete specification
  - Prompt Builder form fields
  - Active modules display
  - Agent selection UI
  - Save/load workflow
- Reasoning Surface complete specification
  - Chat message display
  - Checkbox selection system
  - Block-level selection (optional)
  - Selection counter
  - Composer controls integration
- Outputs Surface complete specification
  - Selection summary display
  - Format template UI
  - Preview panel
  - Destination buttons
  - Publish workflow
- Database artifact schema
- API endpoints (all CRUD operations)
- SSE real-time updates
- Error handling and edge cases
- Performance optimization strategies

---

### 4. EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md
**Purpose:** Complete export system with templates and destinations  
**Size:** 12-15 pages  
**Audience:** Backend engineers, DevOps

**Must Include:**
- Template formatter architecture
  - Base class interface
  - Built-in templates (Markdown, YAML, Skill, XML)
  - Custom template creation workflow
  - Template validation rules
- Destination handler architecture
  - Base class interface
  - Built-in destinations (Download, GitHub PR, TiDB, Drive)
  - Custom destination creation workflow
  - Authentication/authorization
- Export pipeline orchestration
  - Stage 1: Assembly
  - Stage 2: Transformation
  - Stage 3: Validation
  - Stage 4: Preview
  - Stage 5: Publish
- The Bin integration
- Error handling and rollback
- Audit trail requirements
- Performance considerations

---

### 5. PROMPT_CONTROL_HUB_COMPLETE_SPECIFICATION.md
**Purpose:** Unified Agents/Skills/Commands control system  
**Size:** 18-22 pages  
**Audience:** Full-stack engineers, product managers

**Must Include:**
- Hub UI/UX specification
  - Single entry point (icon placement)
  - Unified search across all types
  - Keyboard shortcut (Cmd/Ctrl+K)
- Agent Profiles
  - Structure and fields
  - Selection UI (radio buttons)
  - Default behaviors
  - Knowledge references
- Skills (Claude Skills + Custom)
  - Structure and fields
  - Selection UI (checkboxes)
  - Compatibility rules
  - Conflict warnings
- Command Macros
  - Structure and fields
  - Slash command integration
  - Parameter prompts
  - Execution modes (insert vs run)
- Transparent Stack visualization
  - Ordered layer display
  - Chip affordances (hover, remove, warnings)
  - Scope indicators
- Conflict detection system
  - Skill ↔ Skill conflicts
  - Agent ↔ Skill incompatibilities
  - Command ↔ Mode constraints
- Database schema (all tables)
- API endpoints (all operations)
- The Bin integration
- Version management

---

## 🎯 CONTEXT MATERIALS FOR EXTERNAL LLM

### Documents to Provide

1. **PROMPT_CONTROL_DASHBOARD_ENDUSER_UX_INTEGRATION.md** (uploaded)
   - End-user control surface concepts
   - Agent/Skill/Command/Stack model
   - Database schema proposals

2. **DEVGUIDE_TRI_SURFACE_CHAT_WORKBENCH_SPEC.md** (uploaded)
   - Tri-surface architecture
   - Selection model
   - Export pipeline
   - Data model additions

3. **Progressive Disclosure Guide** (uploaded in text)
   - 7 phases specification
   - UX requirements per phase
   - Friendliness guidelines

4. **Layout Edit Mode Specification** (uploaded in text)
   - Drag-and-drop behavior
   - Rails system
   - Keyboard controls

5. **Visual Mockups** (uploaded images + video)
   - chat_workbench_reasoning.png
   - chat_workbench_advanced.png
   - chat_workbench_outputs.png
   - chat_workbench_inputs.png
   - chat_workbench_slide.gif
   - chat_workbench_slide.mp4

6. **Project Context** (from project files)
   - ARTIFACT_05 (Data Model v1.1.0)
   - ARTIFACT_04 (Unified AI Layer)
   - ARTIFACT_06 (Master Control Dashboard)
   - The Bin specification (ARTIFACT_03)
   - Tech stack constraints (Rust/Go/Elm, TiDB)

---

## 📝 COPY-PASTE PROMPTS FOR EXTERNAL LLM

### Prompt 1: COMPOSER_UX_COMPLETE_SPECIFICATION.md

```
I need you to create a comprehensive UX specification for a developer-focused chat composer with progressive disclosure features.

CONTEXT:
- Target audience: Software developers using an AI-powered development platform
- Key principle: "Narrow, learnable, and power-user fast"
- Progressive disclosure: 7 phases from basic to advanced

REFERENCE MATERIALS:
[Attach: Progressive Disclosure Guide from uploaded text]
[Attach: Layout Edit Mode Specification]
[Attach: All 6 visual mockups]

REQUIREMENTS:
Create a specification document titled "COMPOSER_UX_COMPLETE_SPECIFICATION.md" (15-20 pages) that includes:

1. **Executive Summary** (1 page)
   - Purpose and scope
   - Target audience
   - Key principles

2. **Progressive Disclosure Phases** (8-10 pages)
   For each of the 7 phases, provide:
   - Phase name and number
   - User capability unlocked
   - UI components added
   - UX requirements (with specifics)
   - Accessibility considerations
   - Example user flow
   - Success metrics
   
   Phase 1: Basic Chat (Send + Attach)
   Phase 2: Context Injection (@ mentions + chips)
   Phase 3: Slash Commands (/ palette)
   Phase 4: Mode Selector (Ask/Edit/Agent/Plan)
   Phase 5: Execution Toggles (Web + Fast/Deep)
   Phase 6: Agent Run Controls (Stop, progress, queue)
   Phase 7: IDE Safety Controls (Apply/Review + Checkpoints)

3. **Component Inventory** (2-3 pages)
   - Left Rail components (detailed specs)
   - Input area components
   - Right Rail components
   - Secondary strip components
   - Hidden/Advanced components
   - For each component:
     * Name and purpose
     * Visual appearance
     * Interaction behavior
     * States (default, hover, active, disabled)
     * Keyboard shortcuts
     * Accessibility labels

4. **Layout Specifications** (2 pages)
   - Default layout (narrow mode)
   - Advanced layout (power user mode)
   - Responsive breakpoints
   - Spacing and alignment rules

5. **Keyboard Shortcuts** (1 page)
   - Complete list of all shortcuts
   - Conflicts and resolution
   - Customization options

6. **Accessibility Requirements** (1 page)
   - WCAG 2.1 AA compliance checklist
   - Screen reader support
   - Keyboard navigation
   - Focus management
   - ARIA labels and roles

7. **Animation & Transitions** (1 page)
   - Timing and easing functions
   - Performance considerations
   - Reduced motion support

8. **Error States & Edge Cases** (1 page)
   - Input validation errors
   - Network failures
   - Rate limiting
   - Empty states
   - Loading states

9. **Example User Flows** (2-3 pages)
   - 5-7 complete scenarios from basic to advanced
   - Screenshots/wireframes at key steps
   - Expected outcomes

FORMAT:
- Markdown with clear headers
- Code blocks for schemas/examples
- Tables for component matrices
- Mermaid diagrams where helpful
- Copy-paste ready for engineering teams

TONE:
- Technical but readable
- Authoritative (this IS the spec)
- No ambiguity or "could" language
- Use "must", "will", "shall" for requirements

DELIVERABLE:
A single Markdown file that frontend engineers can use as THE source of truth for implementation, with no gaps or ambiguities.
```

---

### Prompt 2: LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md

```
I need you to create a comprehensive specification for a drag-and-drop layout customization system for a developer chat interface.

CONTEXT:
- Users can reorder composer controls via drag-and-drop
- 3 Rails system constrains placement (Left/Right/Secondary)
- Keyboard-first workflow for IDE users
- Per-user persistence with import/export

REFERENCE MATERIALS:
[Attach: Layout Edit Mode Specification from uploaded text]
[Attach: Visual mockups showing composer layout]

REQUIREMENTS:
Create a specification document titled "LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md" (10-12 pages) that includes:

1. **Executive Summary** (1 page)
   - Feature purpose
   - User value proposition
   - Technical approach

2. **Edit Mode Toggle** (1 page)
   - Button placement options (with recommendation)
   - Visual design
   - States (on/off)
   - Keyboard shortcut
   - First-use onboarding

3. **Rails System** (2-3 pages)
   - Rail 1 (Left Cluster): Setup components
     * Draggable: Prompt Hub, Add/Attach, Mode Selector
     * Constraints: Must stay in left rail
   - Rail 2 (Right Cluster): Execution components
     * Draggable: Web toggle, Effort toggle
     * Fixed: Send/Stop button (not draggable)
     * Constraints: Must stay in right rail
   - Rail 3 (Secondary Strip): Optional row
     * Draggable: Token counter, Queue indicator, Project selector
     * Can be hidden entirely
   - Hidden Tray: Components user wants to hide
     * Drag to hide, drag back to reveal
   
   For each rail, specify:
   - Visual appearance in edit mode
   - Drop zone indicators
   - Insertion preview behavior
   - Constraint enforcement (with error messages)

4. **Draggable Components Inventory** (1-2 pages)
   Complete list of all components with:
   - Component name
   - Default rail
   - Draggable: Yes/No
   - Hideable: Yes/No
   - Constraints
   - Dependencies (what breaks if hidden)

5. **Drag-and-Drop Behavior** (2-3 pages)
   - Snap-to-slot mechanics (no pixel-perfect dragging)
   - Insertion indicator design
   - Invalid placement feedback
   - Drag handle design
   - Touch support (mobile/tablet)
   - Multi-select dragging (optional)

6. **Keyboard Controls** (1 page)
   Complete keyboard workflow:
   - Tab/Shift+Tab: Navigate components
   - Arrow keys: Move selected component
   - Space/Enter: Pick up / Drop
   - Esc: Cancel drag operation
   - Delete: Send to Hidden tray
   - Shortcuts for common actions

7. **Control Bar** (1 page)
   When edit mode is ON:
   - Save button (commits changes)
   - Cancel button (reverts to last saved)
   - Reset to Default button (factory reset)
   - Visual design
   - Confirmation dialogs

8. **Layout Persistence** (1-2 pages)
   - Database schema (user_composer_layouts table)
   - JSON structure for layout configuration
   - Save/Load workflow
   - Per-user vs per-device options
   - Sync across devices (optional)

9. **Import/Export** (1 page)
   - Export layout as JSON/YAML file
   - Import layout from file
   - Validation on import
   - Conflict resolution (what if component doesn't exist)
   - Community layout sharing (optional)

10. **Default Layouts** (1 page)
    - 3-5 preset layouts for common workflows:
      * Minimal (basic chat only)
      * Balanced (default, good for most users)
      * Power User (all controls visible)
      * Code Review (optimized for reviewing)
      * Documentation (optimized for writing docs)
    - One-click switching between presets

11. **Edge Cases & Error Handling** (1 page)
    - Component conflicts (two components want same slot)
    - Migration (what happens when new components are added in updates)
    - Corrupted layout data
    - Browser local storage limits
    - Layout reset if rendering breaks

12. **Implementation Notes** (1 page)
    - Recommended drag-and-drop library (if any)
    - Performance considerations
    - Browser compatibility
    - Testing strategy

FORMAT:
- Markdown with clear headers
- JSON/YAML examples for schemas
- ASCII diagrams for rail layouts
- Step-by-step workflows
- Copy-paste ready for engineering teams

TONE:
- Technical and precise
- Authoritative (this IS the spec)
- No ambiguity
- Implementation-focused

DELIVERABLE:
A single Markdown file that frontend engineers can use to implement drag-and-drop layout customization with zero ambiguity.
```

---

### Prompt 3: TRI_SURFACE_WORKBENCH_COMPLETE_SPECIFICATION.md

```
I need you to create a comprehensive specification for a three-surface chat workbench with independent AI agents and database-mediated communication.

CONTEXT:
- Three surfaces: Inputs (prompt crafting) | Reasoning (execution) | Outputs (packaging)
- Each surface has its own selectable AI agent
- Surfaces communicate via database (session_artifacts table), NOT real-time orchestration
- Default: One surface open at a time (simple mode)
- Advanced: Multi-surface view with resizable columns

REFERENCE MATERIALS:
[Attach: DEVGUIDE_TRI_SURFACE_CHAT_WORKBENCH_SPEC.md]
[Attach: All 6 visual mockups, especially the sliding animation GIF]
[Attach: ARTIFACT_05 database schema (existing)]

REQUIREMENTS:
Create a specification document titled "TRI_SURFACE_WORKBENCH_COMPLETE_SPECIFICATION.md" (20-25 pages) that includes:

1. **Executive Summary** (1-2 pages)
   - Feature purpose and value
   - Architecture overview (database-mediated, NOT orchestrated)
   - User mental model: "Three Chrome tabs with shared clipboard"

2. **Surface Switching System** (2-3 pages)
   - Three pills UI (INPUTS | REASONING | OUTPUTS)
     * Visual design
     * Active/inactive states
     * Click behavior
     * Keyboard shortcuts (Cmd+1/2/3)
   - Sliding panel behavior
     * Animation timing and easing
     * Left surface slides in (Reasoning compresses right)
     * Right surface slides in (Reasoning compresses left)
     * Center surface expands full width
   - Collapsed state indicators
     * Vertical "INPUTS" and "OUTPUTS" labels on edges
     * Click to expand behavior
   - Advanced mode: Multi-surface view
     * Resizable splitters between panels
     * Min/max width constraints
     * User preference persistence

3. **Inputs Surface Specification** (4-5 pages)
   Complete specification for prompt engineering surface:
   
   **3.1 Prompt Builder Form**
   - Structured fields:
     * ROLE (text input, 100 char limit)
     * OBJECTIVE (textarea, 500 char limit)
     * CONSTRAINTS (textarea, 500 char limit)
     * EXAMPLES (textarea, 1000 char limit, placeholder: "Add 1-2 few-shot pairs")
     * OUTPUT CONTRACT (textarea, 300 char limit)
   - Auto-save to browser localStorage (every 30s)
   - Clear/Reset button
   - Preview button (shows assembled prompt)
   
   **3.2 Active Modules Display**
   - Chips showing:
     * Selected Agent (e.g., "Agent: Product PM")
     * Active Skills (e.g., "Skill: Citations")
     * Active Template (if any)
     * Active Schema (if any)
   - Each chip has "X" to remove
   - Click chip to edit/configure
   
   **3.3 Send to Reasoning Button**
   - Orange button: "Send to Reasoning →"
   - Click behavior:
     * Assembles all fields into single prompt
     * Saves to session_artifacts table (artifact_type: 'prompt_draft')
     * Shows success toast: "Draft saved! Switch to Reasoning to use it."
     * Optional: Auto-switch to Reasoning surface
   
   **3.4 Agent Selection**
   - Dropdown or modal to select AI agent
   - Shows agent capabilities, default model, cost estimate
   - "Change Agent" button always visible

4. **Reasoning Surface Specification** (5-6 pages)
   Complete specification for main chat/execution surface:
   
   **4.1 Chat Message Display**
   - Message list (reverse chronological, newest at bottom)
   - Each message shows:
     * Avatar (user vs agent)
     * Timestamp (HH:MM format, tooltip shows full date)
     * Message content (Markdown rendered)
     * Checkbox for selection (orange when checked)
     * Optional: Block-level checkboxes (paragraph, code, list, table)
   - Virtualized scrolling for performance (large conversations)
   
   **4.2 Selection System**
   - Message-level checkbox (selects entire message)
   - Block-level checkboxes (optional, for granular selection)
   - Selected items highlighted with subtle background
   - Selection counter: "Selected: 3 messages • ~1.2k tokens"
   - Click counter to jump to Outputs surface
   - Clear All button
   
   **4.3 Load Draft from Inputs**
   - Button: "Load Draft from Inputs"
   - Click behavior:
     * Queries session_artifacts for artifact_type='prompt_draft'
     * Inserts into composer
     * Shows provenance: "Loaded from Inputs • [timestamp]"
   - Handles "no draft found" gracefully
   
   **4.4 Composer Controls**
   - (Integrated with COMPOSER_UX_COMPLETE_SPECIFICATION.md)
   - Input field with @ and / triggers
   - Context chips below input
   - Mode selector, Web/Effort toggles, Send button
   
   **4.5 Save Result Button**
   - Button: "Save Result"
   - Click behavior:
     * Saves last assistant message to session_artifacts (artifact_type='reasoning_result')
     * Shows success toast: "Result saved! Available in Outputs."
   
   **4.6 Agent Selection**
   - Dropdown to select AI agent (independent from Inputs agent)
   - Agent indicator always visible

5. **Outputs Surface Specification** (5-6 pages)
   Complete specification for export/packaging surface:
   
   **5.1 Load Result from Reasoning**
   - Button: "Load Result from Reasoning"
   - Click behavior:
     * Queries session_artifacts for artifact_type='reasoning_result'
     * Displays in content area
     * Shows provenance: "Loaded from Reasoning • [timestamp]"
   
   **5.2 Selection Summary**
   - Display: "SELECTION: 3 blocks • ~1.2k tokens"
   - Shows what's included in export bundle
   - Edit button to jump back to Reasoning for re-selection
   
   **5.3 Format Template Selector**
   - Dropdown: "FORMAT"
   - Options:
     * Markdown
     * Markdown + YAML front matter
     * SKILL.md
     * XML wrapper
     * Custom (user-created templates)
   - Each option shows preview of output structure
   
   **5.4 Auto-Apply Format Toggle**
   - Toggle: "AUTO-APPLY FORMAT AFTER RUN"
   - When ON: Automatically formats selection after every Reasoning run
   - When OFF: User must manually click "Apply Format"
   
   **5.5 Preview Panel**
   - Shows formatted output
   - Syntax highlighted
   - Line numbers
   - Copy button
   - Edit button (opens in modal for tweaks)
   
   **5.6 Destination Selector**
   - Buttons for each destination:
     * Download (.md) - saves to user's downloads folder
     * GitHub PR - creates PR in configured repo
     * TiDB - saves to database
     * Drive - saves to Google Drive (if connected)
   - Each button shows icon + label
   - Disabled state if destination not configured
   
   **5.7 Publish Button**
   - Orange button: "Publish →"
   - Click behavior:
     * Creates export job in export_jobs table
     * Runs through The Bin validation
     * Shows progress: "Publishing... • Step 2/5"
     * On success: Shows location (link or path)
     * On failure: Shows error with retry button
   
   **5.8 Agent Selection**
   - Dropdown to select AI agent for formatting assistance
   - Optional: Auto-formatting agent (Claude Haiku for speed)

6. **Database Schema** (3-4 pages)
   All tables needed for tri-surface workbench:
   
   - chat_sessions
   - chat_messages
   - session_artifacts (THE KEY TABLE for inter-surface communication)
   - export_templates
   - export_destinations
   - export_jobs
   - user_composer_layouts
   
   For each table, provide:
   - CREATE TABLE statement (MySQL/TiDB syntax)
   - Column descriptions
   - Indexes and foreign keys
   - Example data (3-5 rows)
   - Usage notes

7. **API Endpoints** (2-3 pages)
   All HTTP endpoints needed:
   
   **Sessions:**
   - GET /api/chat/sessions
   - POST /api/chat/sessions
   - GET /api/chat/sessions/{id}/messages
   - POST /api/chat/sessions/{id}/messages
   
   **Artifacts:**
   - POST /api/sessions/{id}/artifacts (save)
   - GET /api/sessions/{id}/artifacts/{type} (load)
   - GET /api/sessions/{id}/artifacts (list all)
   
   **Export:**
   - GET /api/export/templates
   - POST /api/export/templates (via Bin)
   - GET /api/export/destinations
   - POST /api/export/destinations (via Bin)
   - POST /api/export/jobs (create job)
   - GET /api/export/jobs/{id} (status)
   
   For each endpoint:
   - HTTP method and path
   - Request body schema (JSON)
   - Response schema (JSON)
   - Error codes
   - Example curl command

8. **SSE Real-Time Updates** (1 page)
   - Event stream: /api/sse/chat-workbench
   - Event types:
     * selection_updated
     * export_completed
     * export_failed
     * agent_switched
   - Event payload schemas
   - Client-side handling

9. **The Bin Integration** (1-2 pages)
   - All mutations go through The Bin
   - Change types:
     * CreateSession
     * SaveArtifact
     * CreateExportJob
     * UpdateTemplate
     * UpdateDestination
   - Impact analysis for each change type
   - Accept/Reject workflow

10. **Error Handling & Edge Cases** (1-2 pages)
    - Session not found
    - Artifact not found (no draft saved yet)
    - Export job failures (GitHub API down, etc.)
    - Network issues during save
    - Concurrent modifications (two users, same session)
    - Token limits exceeded
    - Validation errors

11. **Performance Optimization** (1 page)
    - Lazy loading surfaces (only load active surface)
    - Virtual scrolling for long message lists
    - Debounced selection updates
    - Cached artifact queries
    - Optimistic UI updates

12. **Implementation Notes** (1 page)
    - Elm architecture patterns
    - Go service structure
    - Database migration strategy
    - Testing approach (unit + integration)

FORMAT:
- Markdown with clear headers
- SQL for schemas
- JSON for API examples
- Mermaid diagrams for flows
- Tables for component specs

TONE:
- Comprehensive and authoritative
- Implementation-ready
- Zero ambiguity

DELIVERABLE:
A single Markdown file that full-stack engineers can use to implement the tri-surface workbench with complete clarity on all requirements.
```

---

### Prompt 4: EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md

```
I need you to create a comprehensive specification for an export pipeline system that transforms chat selections into formatted files and publishes them to various destinations.

CONTEXT:
- Users select chat messages/blocks in Reasoning surface
- Outputs surface applies templates (Markdown, YAML, Skill, XML, custom)
- Validation ensures output meets requirements
- Publishing sends to destinations (Download, GitHub, TiDB, Drive)
- All mutations flow through "The Bin" validation system

REFERENCE MATERIALS:
[Attach: Python export framework from TRI_SURFACE_CORRECTED_ANALYSIS.md]
[Attach: Outputs surface mockup image]
[Attach: The Bin specification from ARTIFACT_03]

REQUIREMENTS:
Create a specification document titled "EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md" (12-15 pages) that includes:

1. **Executive Summary** (1 page)
   - Purpose: Transform and publish chat output
   - Architecture: 5-stage pipeline
   - Integration: The Bin for all mutations

2. **Pipeline Architecture** (2 pages)
   
   **Stage 1: Assembly**
   - Input: Selection set (message IDs + block IDs)
   - Process:
     * Fetch messages from database
     * Order by timestamp
     * Preserve provenance (who said what, when)
     * Calculate token count
   - Output: AssembledContent object
   
   **Stage 2: Transformation**
   - Input: AssembledContent + Template
   - Process:
     * Apply template formatter
     * Inject metadata
     * Handle edge cases (empty content, special chars)
   - Output: FormattedContent object
   
   **Stage 3: Validation**
   - Input: FormattedContent
   - Process:
     * Schema validation (YAML front matter, XML tags)
     * Content rules (max length, required fields)
     * The Bin pre-flight check
   - Output: ValidationResult (pass/fail + errors)
   
   **Stage 4: Preview**
   - Input: FormattedContent
   - Process:
     * Render for user review
     * Generate diff if updating existing file
     * Show metadata and stats
   - Output: PreviewRendered object
   
   **Stage 5: Publish**
   - Input: FormattedContent + Destination
   - Process:
     * Execute destination handler
     * Track in export_jobs table
     * The Bin final validation
     * Rollback mechanism if failure
   - Output: PublishResult (success + location, or failure + error)

3. **Template Formatter System** (3-4 pages)
   
   **3.1 Base Class Architecture**
   ```python
   class ExportTemplate(ABC):
       @abstractmethod
       def format(content: str, metadata: dict) -> str
       def validate(formatted: str) -> bool
       def add_metadata(content: str, metadata: dict) -> str
   ```
   
   **3.2 Built-in Templates**
   
   *MarkdownTemplate:*
   - Format: Plain Markdown
   - Metadata: Title + Author + Date as header
   - Use case: General documentation
   
   *YAMLFrontMatterTemplate:*
   - Format: YAML front matter + Markdown body
   - Metadata: Full YAML block with all fields
   - Validation: YAML syntax check
   - Use case: Static site generators, documentation systems
   
   *SkillFileTemplate:*
   - Format: DevGuide SKILL.md format
   - Metadata: Name, description, version, created timestamp
   - Validation: Required sections present
   - Use case: Claude/DevGuide skill files
   
   *XMLWrapperTemplate:*
   - Format: XML document with structured sections
   - Metadata: XML attributes and meta tags
   - Validation: Tag balance, well-formed XML
   - Use case: System prompts, structured data exchange
   
   **3.3 Custom Template Creation**
   - User uploads Python script or Jinja2 template
   - Template registry (stored in export_templates table)
   - Validation on upload
   - Sandbox execution (security)
   - Version management

4. **Destination Handler System** (3-4 pages)
   
   **4.1 Base Class Architecture**
   ```python
   class ExportDestination(ABC):
       @abstractmethod
       def save(filename: str, content: str) -> str  # returns location
       def validate_config() -> bool
       def test_connection() -> bool
   ```
   
   **4.2 Built-in Destinations**
   
   *LocalFileDestination:*
   - Saves to browser Downloads folder (web) or specified path (desktop)
   - Filename sanitization
   - Overwrite prompts
   - Use case: Quick local saves
   
   *GitHubDestination:*
   - Creates PR or commits directly
   - Configuration:
     * Repository (user/repo)
     * Branch
     * Path within repo
     * GitHub token (encrypted)
   - Validation:
     * Token has write access
     * Repo exists
     * Path is valid
   - Behavior:
     * Create/update file
     * Commit message: Templated
     * PR title/description: Generated from metadata
   - Use case: Documentation updates, code snippets
   
   *TiDBDestination:*
   - Saves to database table
   - Configuration:
     * Table name
     * Column mappings
     * Connection credentials (encrypted)
   - Validation:
     * Table exists
     * Schema matches
   - Behavior:
     * INSERT or UPDATE based on primary key
     * Timestamp tracking
   - Use case: Knowledge base, artifact storage
   
   *DriveDestination:*
   - Saves to Google Drive
   - Configuration:
     * Folder ID
     * OAuth credentials
   - Validation:
     * User has write access
     * Folder exists
   - Behavior:
     * Create/update file
     * Set permissions
   - Use case: Team documentation, shared resources
   
   **4.3 Custom Destination Creation**
   - User provides connection config
   - Destination registry (stored in export_destinations table)
   - Test connection before saving
   - Retry logic for transient failures

5. **The Bin Integration** (2 pages)
   
   **5.1 Change Payloads**
   ```rust
   pub enum ExportChange {
       CreateTemplate { name, template_content, ... },
       UpdateTemplate { id, changes, ... },
       CreateDestination { name, dest_type, config, ... },
       UpdateDestination { id, changes, ... },
       CreateExportJob { selection_id, template_id, dest_id, ... },
   }
   ```
   
   **5.2 Impact Analysis**
   For each change type:
   - What files/records will be touched
   - Token cost estimate
   - Risk level (e.g., GitHub push = high, local save = low)
   - Affected users (who else might see this export)
   
   **5.3 Validation Rules**
   - Template must compile/parse successfully
   - Destination configuration must be valid
   - Export job must reference existing template + destination
   - User has permission for destination
   
   **5.4 Accept/Reject Workflow**
   - User reviews impact in The Bin UI
   - Accept: Change is executed
   - Reject: Change is discarded
   - All changes logged for audit

6. **Error Handling** (1-2 pages)
   - Template formatting errors
   - Validation failures (schema, content rules)
   - Destination connection failures (GitHub API down, Drive auth expired)
   - Partial success (5 of 10 files published)
   - Rollback strategy for each stage
   - User-friendly error messages
   - Retry mechanisms (exponential backoff)

7. **Audit Trail** (1 page)
   - export_jobs table tracks all exports
   - Fields:
     * Job ID
     * Session ID
     * Template used
     * Destination used
     * Status (queued, running, completed, failed)
     * Output location (URL, path, record ID)
     * Error message (if failed)
     * Created/completed timestamps
   - Queryable history: "Show me all exports to GitHub last week"

8. **Performance Considerations** (1 page)
   - Async processing (don't block UI)
   - Progress indicators (Step X of Y)
   - Batch exports (multiple files in one job)
   - Rate limiting (respect GitHub API limits, etc.)
   - Caching (re-use formatted content if selection unchanged)

9. **Security Considerations** (1 page)
   - Credential encryption (GitHub tokens, DB passwords)
   - Scope validation (user can't export to others' repos)
   - Content sanitization (prevent XSS in templates)
   - Audit logging (who exported what, when)

10. **API Specification** (1-2 pages)
    All endpoints:
    - POST /api/export/jobs
    - GET /api/export/jobs/{id}
    - GET /api/export/templates
    - POST /api/export/templates
    - GET /api/export/destinations
    - POST /api/export/destinations
    
    Include request/response schemas and examples.

11. **Implementation Notes** (1 page)
    - Python for formatters (sandboxed execution)
    - Go for destination handlers
    - Async job processing (queue system)
    - Testing strategy

FORMAT:
- Markdown with clear headers
- Python code for template/destination classes
- JSON for API schemas
- Flowcharts for pipeline stages

TONE:
- Implementation-focused
- Security-conscious
- Clear and unambiguous

DELIVERABLE:
A single Markdown file that backend engineers can use to implement the complete export pipeline with full clarity.
```

---

### Prompt 5: PROMPT_CONTROL_HUB_COMPLETE_SPECIFICATION.md

```
I need you to create a comprehensive specification for a unified Prompt Control Hub that manages Agents, Skills, Commands, and Stack visualization.

CONTEXT:
- Unifies multiple prompt control paradigms (GPTs/Gems style Agents, Claude Skills, Slash Commands)
- Single entry point (icon + keyboard shortcut)
- Transparent Stack shows active layers
- Conflict detection (skill vs skill, agent vs skill, etc.)
- All managed assets are versioned and flow through The Bin

REFERENCE MATERIALS:
[Attach: PROMPT_CONTROL_DASHBOARD_ENDUSER_UX_INTEGRATION.md]
[Attach: Progressive Disclosure Guide (Hub is part of Phase 2)]
[Attach: Visual mockups showing Active Modules chips]

REQUIREMENTS:
Create a specification document titled "PROMPT_CONTROL_HUB_COMPLETE_SPECIFICATION.md" (18-22 pages) that includes:

1. **Executive Summary** (1-2 pages)
   - Purpose: Unified control for AI behavior customization
   - Mental model: "Select WHO + HOW + DO"
     * Agents = Who (single select, changes voice + defaults)
     * Skills = How (multi select, standards/modules)
     * Commands = What (execute workflow)
   - Integration with tri-surface workbench
   - Relationship to "Prompt Control Dashboard" (admin interface)

2. **Hub Entry Point** (2 pages)
   
   **2.1 Icon Placement**
   - Location: First in left rail (before + Add, Mode selector)
   - Visual design: Icon that suggests "layers" or "stack"
   - Hover tooltip: "Prompt Control (Cmd+K)"
   - Badge: Shows count if anything active beyond defaults
   
   **2.2 Keyboard Shortcut**
   - Cmd+K (Mac) / Ctrl+K (Windows/Linux)
   - Opens Hub modal
   - Closes on Esc or click outside
   - Tab navigation within modal
   
   **2.3 First-Use Onboarding**
   - Tooltip on first click: "Control how Claude responds"
   - Tour of sections (Agents, Skills, Commands)
   - Can dismiss: "Don't show again"

3. **Hub Modal UI** (3-4 pages)
   
   **3.1 Layout**
   - Modal: 600px wide, 500px tall, centered
   - Header:
     * Title: "Prompt Control"
     * Unified search bar
     * Close button (X)
   - Body: 3 tabs
     * Agents
     * Skills
     * Commands
   - Footer:
     * Active Stack preview (chips)
     * Apply/Cancel buttons
   
   **3.2 Unified Search**
   - Search across all types (Agents, Skills, Commands)
   - Fuzzy matching
   - Results grouped by type
   - Keyboard navigation (arrow keys, Enter to select)
   - Recent searches
   
   **3.3 Tabs**
   Each tab has:
   - List of available items
   - Selection mechanism (radio for Agents, checkboxes for Skills)
   - Item details (expand on click)
   - Conflict warnings (if incompatible selections)

4. **Agent Profiles** (4-5 pages)
   
   **4.1 Data Structure**
   ```yaml
   AgentProfile:
     id: uuid
     name: string
     description: string
     instruction_summary: string  # tooltip
     instructions: string  # full system prompt
     knowledge_refs:
       - { kind: "drive_doc"|"github_repo"|"url", ref: string }
     tool_policy:
       allowed_tools: [string]
       web_default: "on"|"off"
     output_style:
       format_contract_id: uuid | null
       tone: "concise"|"friendly"|"formal"
     defaults:
       mode: "Ask"|"Edit"|"Agent"|"Plan"
       effort: "fast"|"deep"
       skills_on_by_default: [skill_id]
   ```
   
   **4.2 Agent Selection UI**
   - Radio buttons (only one active at a time)
   - Each agent shows:
     * Name and short description
     * Icon/avatar
     * Indicator of what it changes (tools, knowledge, tone)
   - Preview button: Shows full instructions
   - Default agent: "General Assistant" (balanced, no special behaviors)
   
   **4.3 Agent Examples**
   - General Assistant (default)
   - Code Reviewer (strict, focused on code quality)
   - Documentation Writer (detailed, structured)
   - Creative Writer (expressive, fewer constraints)
   - Data Analyst (tool-heavy, precise)
   
   **4.4 Agent Management** (admin)
   - Create custom agents
   - Version agents
   - Share with team (if in project)
   - Community marketplace (optional)

5. **Skills** (4-5 pages)
   
   **5.1 Data Structure**
   ```yaml
   SkillModule:
     id: uuid
     name: slug
     description: string
     instructions: string
     triggers:
       - { pattern: string, scope: "message"|"thread"|"project" }
     constraints:
       output_format: "json"|"markdown"|null
       citation_required: bool
     compatibility:
       conflicts_with: [skill_id]
       compatible_agents: [agent_id] | null
   ```
   
   **5.2 Skill Selection UI**
   - Checkboxes (multi-select)
   - Each skill shows:
     * Name and description
     * Indicator if it changes output format
     * Warning if incompatible with current agent
   - Toggle all on/off
   - Favorites (pin frequently used)
   
   **5.3 Built-in Skills** (examples)
   - Citations Required (enforces source attribution)
   - Structured Output (enforces JSON/YAML)
   - Code Standards (enforces style guides)
   - Accessibility Checker (for UI/UX work)
   - Security Scanner (for code review)
   
   **5.4 Claude Skills Integration**
   - Use Claude's native skills where available:
     * docx, pdf, pptx, xlsx
     * web-artifacts-builder
     * product-self-knowledge
   - Custom skills for DevGuide-specific workflows
   
   **5.5 Conflict Detection**
   - Example: "Citations Required" + "Code Standards" both define output format
   - Warning: "Two skills conflict. Choose one:"
   - Resolution: User picks which to keep active
   - Rules stored in prompt_conflicts table

6. **Command Macros** (3-4 pages)
   
   **6.1 Data Structure**
   ```yaml
   CommandMacro:
     id: uuid
     slash: "/summarize"
     name: string
     description: string
     category: "Write"|"Code"|"Research"|"Transform"
     mode_requirement: ["Agent"] | null
     execution:
       kind: "insert_scaffold"|"execute_workflow"
       requires_confirmation: bool
     parameters_schema: jsonschema
     template: string
     output_contract_id: uuid | null
     tool_hints:
       allowed_tools: [string]
   ```
   
   **6.2 Command Selection UI**
   - Action list (click to run or insert)
   - Categories:
     * Write (drafts, docs, emails)
     * Code (scaffolds, refactors, tests)
     * Research (summaries, comparisons)
     * Transform (format conversions, extract data)
   - Each command shows:
     * Slash trigger
     * Name and description
     * What it produces
     * Required mode (if any)
   - Recent commands (last 5 used)
   
   **6.3 Command Examples** (DevGuide-specific)
   - /validate - Run metadata validation
   - /review - Code review workflow
   - /scaffold - Generate project structure
   - /comply - Check governance compliance
   - /sync - Update knowledge base
   - /extract - Pull data from documents
   - /compare - Compare two items
   
   **6.4 Slash Command Integration**
   - Typing "/" in composer also triggers commands
   - Hub provides discoverable list
   - Both UIs reference same command registry
   
   **6.5 Command Parameters**
   - If command needs params:
     * Inline chips appear in composer
     * Or small popover for complex params
   - Example: /scaffold requires "project type"

7. **Transparent Stack Visualization** (2-3 pages)
   
   **7.1 Stack Layers (Ordered)**
   1. Base system defaults
   2. Project rules (if in a project)
   3. Selected Agent
   4. Selected Skills (multiple)
   5. Selected Command (if queued)
   
   **7.2 Stack Display**
   - Shown as chips in Hub footer
   - Also shown below composer in main UI
   - Each chip:
     * Layer icon + name
     * Hover: Shows instruction summary
     * X button: Remove layer (except base)
     * Warning badge: If conflict
   
   **7.3 Stack Scope**
   - one_message: Applies only to next message
   - thread: Applies to entire conversation
   - project: Applies to all conversations in project
   - User can change scope per layer

8. **Conflict Detection System** (2-3 pages)
   
   **8.1 Conflict Types**
   - Skill ↔ Skill: Both define output format
   - Agent ↔ Skill: Agent forbids tool that skill requires
   - Command ↔ Mode: Command requires Edit mode but user in Ask mode
   - Command ↔ Skill: Command output conflicts with skill constraint
   
   **8.2 Conflict Storage**
   ```sql
   CREATE TABLE prompt_conflicts (
     id VARCHAR(36) PRIMARY KEY,
     kind VARCHAR(32) NOT NULL,  -- 'skill-skill'|'agent-skill'|...
     left_id VARCHAR(36) NOT NULL,
     right_id VARCHAR(36) NOT NULL,
     severity VARCHAR(16) NOT NULL,  -- 'low'|'medium'|'high'|'critical'
     message TEXT NOT NULL,
     resolution_hint TEXT
   );
   ```
   
   **8.3 Conflict UI**
   - Warning badge on conflicting chips
   - Hover: Shows conflict message
   - Resolution options:
     * Deactivate one of the conflicting items
     * Accept risk (proceed anyway)
   - Critical conflicts: Block execution until resolved

9. **Database Schema** (2 pages)
   - agents table
   - agent_versions table
   - commands table
   - command_versions table
   - prompt_stacks table (saved configurations)
   - prompt_stack_versions table
   - prompt_conflicts table
   
   For each, provide CREATE TABLE + indexes.

10. **API Endpoints** (2 pages)
    - GET /api/agents
    - POST /api/agents (via Bin)
    - GET /api/commands
    - POST /api/commands (via Bin)
    - GET /api/stacks
    - POST /api/stacks (via Bin)
    - GET /api/conflicts
    
    Include request/response schemas.

11. **The Bin Integration** (1-2 pages)
    - All agent/command/stack changes go through Bin
    - Impact analysis shows:
      * Which conversations affected
      * Token cost changes
      * Conflict warnings
    - Accept/Reject workflow

12. **Saved Stack Configurations** (1-2 pages)
    - User can save current stack as preset
    - Name: "Code Review Setup", "Documentation Mode", etc.
    - One-click load saved stack
    - Export/import stacks (YAML)
    - Share stacks with team

13. **Integration with Tri-Surface Workbench** (1 page)
    - Each surface can have different agent selected
    - Skills apply per-surface or globally (user choice)
    - Commands available in all surfaces (via / trigger)
    - Stack visualization shows per-surface layers

14. **Implementation Notes** (1 page)
    - Elm for Hub modal UI
    - Go for API + conflict detection
    - Rust for Bin integration
    - Version everything (agents, commands, stacks)

FORMAT:
- Markdown with headers
- YAML for schemas
- SQL for database
- JSON for API examples
- Mermaid for conflict detection flow

TONE:
- Comprehensive
- Product-focused (this is end-user UX)
- Implementation-ready

DELIVERABLE:
A single Markdown file that product + engineering teams can use to implement the unified Prompt Control Hub.
```

---

## ✅ ACCEPTANCE CRITERIA

Each deliverable must meet these standards:

### 1. Completeness
- ✅ All sections from prompt are present
- ✅ No "TBD" or "TODO" placeholders
- ✅ All code examples are complete and runnable
- ✅ All schemas are valid (SQL, JSON, YAML)

### 2. Clarity
- ✅ Unambiguous language ("must", "will", not "could", "should")
- ✅ Technical precision (specific numbers, formats, constraints)
- ✅ No assumptions left unstated
- ✅ Edge cases explicitly handled

### 3. Implementability
- ✅ Frontend engineers can code directly from spec
- ✅ Backend engineers can code directly from spec
- ✅ DevOps can deploy from spec
- ✅ QA can test from spec (testable requirements)

### 4. Consistency
- ✅ Terminology consistent across all 5 docs
- ✅ Database schemas align across docs
- ✅ API contracts align across docs
- ✅ No contradictions between documents

### 5. Format
- ✅ Markdown with proper headers (H1-H4)
- ✅ Code blocks with syntax highlighting
- ✅ Tables for structured data
- ✅ Diagrams where helpful (Mermaid or ASCII)
- ✅ Page count within target range

### 6. Tone
- ✅ Authoritative (this IS the spec, not suggestions)
- ✅ Technical but readable
- ✅ Implementation-focused
- ✅ Copy-paste ready for engineering

---

## 📊 TIER X EXECUTION PLAN

### Step 1: Prepare Context Package (30 minutes)
- Gather all reference materials
- Upload to external LLM (GPT-4, Claude Opus, etc.)
- Create context document with project background

### Step 2: Execute Prompts (8-10 hours)
Execute each of the 5 prompts in sequence:

1. **Prompt 1** → COMPOSER_UX_COMPLETE_SPECIFICATION.md (2 hours)
2. **Prompt 2** → LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md (1.5 hours)
3. **Prompt 3** → TRI_SURFACE_WORKBENCH_COMPLETE_SPECIFICATION.md (2.5 hours)
4. **Prompt 4** → EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md (1.5 hours)
5. **Prompt 5** → PROMPT_CONTROL_HUB_COMPLETE_SPECIFICATION.md (2 hours)

**Note:** Use extended thinking mode (if available) for each prompt

### Step 3: Review & Validate (1-2 hours)
- Check each document against acceptance criteria
- Cross-reference for consistency
- Fill any gaps
- Fix any contradictions

### Step 4: Package for TIER 3 (30 minutes)
- Add documents to project knowledge base
- Create index/table of contents
- Mark as "authoritative references" for TIER 3
- Hand off to TIER 3 execution team

**Total Time: 10-13 hours**

---

## 🎯 HANDOFF TO TIER 3

Once TIER X is complete, TIER 3 execution can begin with:

✅ **5 comprehensive specification documents** as authoritative references  
✅ **No ambiguity** in requirements  
✅ **All UX patterns** documented  
✅ **All database schemas** defined  
✅ **All API contracts** specified  
✅ **Complete implementation guidance**

TIER 3 execution becomes **straightforward implementation** of well-defined requirements, not discovery and design work.

---

## 📁 DELIVERABLE STRUCTURE

```
/TIER_X_SPECIFICATIONS/
├── README.md (this document)
├── COMPOSER_UX_COMPLETE_SPECIFICATION.md (15-20 pages)
├── LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md (10-12 pages)
├── TRI_SURFACE_WORKBENCH_COMPLETE_SPECIFICATION.md (20-25 pages)
├── EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md (12-15 pages)
├── PROMPT_CONTROL_HUB_COMPLETE_SPECIFICATION.md (18-22 pages)
└── CONTEXT_MATERIALS/
    ├── PROMPT_CONTROL_DASHBOARD_ENDUSER_UX_INTEGRATION.md
    ├── DEVGUIDE_TRI_SURFACE_CHAT_WORKBENCH_SPEC.md
    ├── Progressive_Disclosure_Guide.md
    ├── Layout_Edit_Mode_Spec.md
    └── Visual_Mockups/
        ├── chat_workbench_reasoning.png
        ├── chat_workbench_advanced.png
        ├── chat_workbench_outputs.png
        ├── chat_workbench_inputs.png
        ├── chat_workbench_slide.gif
        └── chat_workbench_slide.mp4
```

**Total Package Size:** ~100 pages of comprehensive specifications

---

## ✅ SUCCESS CRITERIA FOR TIER X

TIER X is complete when:

1. ✅ All 5 specification documents exist and are complete
2. ✅ Each document passes all 6 acceptance criteria
3. ✅ Documents are internally consistent (no contradictions)
4. ✅ External LLM reviewer confirms implementability
5. ✅ Project owner approves vision capture
6. ✅ Documents are added to project knowledge base
7. ✅ TIER 3 team confirms they have everything needed

**At that point, TIER 3 execution can begin with confidence.**

---

**Generated:** 2026-01-02T21:00:00Z  
**Status:** âœ… READY FOR EXECUTION  
**Estimated Completion:** 10-13 hours (external LLM work)  
**Next Step:** Execute Prompt 1 with external LLM

---

*This is TIER X - Complete this FIRST before beginning TIER 3 execution.*
