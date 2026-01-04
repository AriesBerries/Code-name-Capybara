# TIER 3 COMPLETE EXECUTION GUIDE
## All Remaining Tasks with Copy-Paste Instructions

**Date:** 2026-01-02T20:30:00Z  
**Status:** âœ… READY TO EXECUTE  
**Context Used:** 51.7% (98.2K / 190K tokens)  
**Estimated Time:** 6-7 hours total

---

## âœ… WHAT'S BEEN COMPLETED

### Phase 1: Analysis & Planning (DONE)
- [x] Reviewed external Prompt Control Dashboard concept
- [x] Analyzed tri-surface chat workbench requirements
- [x] Corrected multi-agent architecture understanding
- [x] Reviewed visual mockups (images + video)
- [x] Created Python export framework
- [x] Identified database schema requirements
- [x] Documented Layout Edit Mode specification

### Phase 2: Supporting Documentation (DONE)
- [x] CORRECTION_SUMMARY.md
- [x] TRI_SURFACE_CORRECTED_ANALYSIS.md
- [x] TIER_3_FINAL_CHECKLIST.md
- [x] Python export framework code samples

---

## âš ï¸ WHAT REMAINS TO BE DONE

### Task 1: Update ARTIFACT_05 (Data Model v1.1.0 → v1.2.0)
**Status:** âš ï¸ NOT STARTED  
**Estimated Time:** 1 hour  
**Priority:** HIGH (required for all other tasks)

**What to add:**

```sql
-- 1. Chat sessions
CREATE TABLE chat_sessions (
  id VARCHAR(36) PRIMARY KEY,
  project_id VARCHAR(36),
  title VARCHAR(255),
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
  INDEX idx_project (project_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 2. Messages per surface
CREATE TABLE chat_messages (
  id VARCHAR(36) PRIMARY KEY,
  session_id VARCHAR(36) NOT NULL,
  surface VARCHAR(32) NOT NULL, -- inputs|reasoning|outputs
  agent_id VARCHAR(36), -- which agent user selected for this surface
  role VARCHAR(32) NOT NULL, -- user|assistant
  content LONGTEXT,
  metadata JSON, -- blocks, selections, etc.
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  FOREIGN KEY (session_id) REFERENCES chat_sessions(id) ON DELETE CASCADE,
  INDEX idx_session_surface (session_id, surface),
  INDEX idx_agent (agent_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 3. Session artifacts (database clipboard)
CREATE TABLE session_artifacts (
  id VARCHAR(36) PRIMARY KEY,
  session_id VARCHAR(36) NOT NULL,
  artifact_type VARCHAR(64) NOT NULL, -- prompt_draft|reasoning_result|selected_blocks
  content LONGTEXT,
  metadata JSON, -- provenance, token count, etc.
  created_from_surface VARCHAR(32), -- which surface created it
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  FOREIGN KEY (session_id) REFERENCES chat_sessions(id) ON DELETE CASCADE,
  INDEX idx_session_type (session_id, artifact_type),
  INDEX idx_surface (created_from_surface)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 4. Export templates (formatters)
CREATE TABLE export_templates (
  id VARCHAR(36) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  template_type VARCHAR(64), -- markdown|yaml_markdown|skill|custom
  template_content TEXT, -- Python script or template string
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  version INT NOT NULL DEFAULT 1,
  INDEX idx_type (template_type),
  INDEX idx_active (is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 5. Export destinations
CREATE TABLE export_destinations (
  id VARCHAR(36) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  destination_type VARCHAR(64), -- download|github|tidb|drive
  configuration JSON, -- destination-specific config
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  INDEX idx_type (destination_type),
  INDEX idx_active (is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 6. Export jobs (for tracking)
CREATE TABLE export_jobs (
  id VARCHAR(36) PRIMARY KEY,
  session_id VARCHAR(36) NOT NULL,
  selection_set_id VARCHAR(36),
  template_id VARCHAR(36),
  destination_id VARCHAR(36),
  status VARCHAR(32) NOT NULL, -- queued|running|completed|failed
  output_location TEXT, -- where file was saved
  error_message TEXT,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  completed_at TIMESTAMP(6),
  FOREIGN KEY (session_id) REFERENCES chat_sessions(id) ON DELETE CASCADE,
  FOREIGN KEY (template_id) REFERENCES export_templates(id),
  FOREIGN KEY (destination_id) REFERENCES export_destinations(id),
  INDEX idx_status (status),
  INDEX idx_session (session_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 7. User composer layouts (for Layout Edit Mode)
CREATE TABLE user_composer_layouts (
  id VARCHAR(36) PRIMARY KEY,
  user_id VARCHAR(36) NOT NULL,
  layout_name VARCHAR(255),
  configuration JSON NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  INDEX idx_user (user_id),
  INDEX idx_user_active (user_id, is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Also add to schema overview:**

```markdown
### Tri-Surface Chat Workbench Tables (7 tables)

**Purpose:** Enable three-surface chat interface with independent AI agents, database-mediated artifact sharing, and export pipeline.

1. **chat_sessions** - Chat session metadata
2. **chat_messages** - Messages per surface with agent tracking
3. **session_artifacts** - Database "clipboard" for inter-surface data
4. **export_templates** - Formatter templates (Markdown, YAML, etc.)
5. **export_destinations** - Destination handlers (GitHub, TiDB, Drive)
6. **export_jobs** - Export pipeline tracking
7. **user_composer_layouts** - Layout Edit Mode configurations

**Total Project Tables:** 31 (was 24, now 31 with workbench)
```

**Copy-Paste Prompt for Execution:**

```
Please update ARTIFACT_05 (Data Model) from v1.1.0 to v1.2.0 by adding the 7 Tri-Surface Chat Workbench tables listed above. Add them to Section 6 (after extension tables) as a new section: "6. Tri-Surface Chat Workbench Tables". Update the table count summary at the top of the document. Maintain all Zulu timestamp conventions (TIMESTAMP(6) with Z suffix in examples).
```

---

### Task 2: Update ARTIFACT_04 (AI Layer)
**Status:** âš ï¸ NOT STARTED  
**Estimated Time:** 45 minutes  
**Priority:** HIGH

**What to add:**

```markdown
## 7. Surface-Specific Agent Management

### 7.1 Architecture

The Tri-Surface Chat Workbench allows users to select **independent AI agents** for each surface:
- **Input Surface**: Specialized in prompt engineering and instruction crafting
- **Reasoning Surface**: Main problem-solving and code generation
- **Output Surface**: Formatting and documentation generation

**Key Design:** No real-time coordination. Agents are completely independent. Data flows between surfaces via **database artifacts** (session_artifacts table).

### 7.2 Surface Agent Types

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum Surface {
    Inputs,
    Reasoning,
    Outputs,
}

#[derive(Debug, Clone)]
pub struct SurfaceAgent {
    pub surface: Surface,
    pub agent_profile_id: Uuid,
    pub provider: String, // claude|openai|google
    pub model: String, // claude-opus-4|gpt-4|gemini-pro
}

pub struct SurfaceAgentManager {
    pub input_agent: Option<SurfaceAgent>,
    pub reasoning_agent: Option<SurfaceAgent>,
    pub output_agent: Option<SurfaceAgent>,
}

impl SurfaceAgentManager {
    pub fn new() -> Self {
        Self {
            input_agent: None,
            reasoning_agent: None,
            output_agent: None,
        }
    }
    
    /// User selects agent for a specific surface
    pub fn set_agent_for_surface(&mut self, surface: Surface, agent: SurfaceAgent) {
        match surface {
            Surface::Inputs => self.input_agent = Some(agent),
            Surface::Reasoning => self.reasoning_agent = Some(agent),
            Surface::Outputs => self.output_agent = Some(agent),
        }
    }
    
    /// Get active agent for current surface
    pub fn get_agent_for_surface(&self, surface: Surface) -> Option<&SurfaceAgent> {
        match surface {
            Surface::Inputs => self.input_agent.as_ref(),
            Surface::Reasoning => self.reasoning_agent.as_ref(),
            Surface::Outputs => self.output_agent.as_ref(),
        }
    }
    
    /// Get all active agents
    pub fn get_all_agents(&self) -> Vec<&SurfaceAgent> {
        let mut agents = Vec::new();
        if let Some(ref agent) = self.input_agent {
            agents.push(agent);
        }
        if let Some(ref agent) = self.reasoning_agent {
            agents.push(agent);
        }
        if let Some(ref agent) = self.output_agent {
            agents.push(agent);
        }
        agents
    }
}
```

### 7.3 Database-Mediated Communication

**No orchestration required!** Surfaces communicate via `session_artifacts` table:

```rust
pub async fn save_artifact(
    db: &Database,
    session_id: Uuid,
    artifact_type: &str,
    content: &str,
    surface: Surface,
) -> Result<Uuid, Error> {
    let artifact_id = Uuid::new_v4();
    sqlx::query!(
        r#"
        INSERT INTO session_artifacts 
        (id, session_id, artifact_type, content, created_from_surface, created_at)
        VALUES (?, ?, ?, ?, ?, NOW(6))
        "#,
        artifact_id.to_string(),
        session_id.to_string(),
        artifact_type,
        content,
        format!("{:?}", surface).to_lowercase()
    )
    .execute(db)
    .await?;
    
    Ok(artifact_id)
}

pub async fn load_artifact(
    db: &Database,
    session_id: Uuid,
    artifact_type: &str,
) -> Result<Option<String>, Error> {
    let result = sqlx::query!(
        r#"
        SELECT content FROM session_artifacts
        WHERE session_id = ? AND artifact_type = ?
        ORDER BY created_at DESC
        LIMIT 1
        "#,
        session_id.to_string(),
        artifact_type
    )
    .fetch_optional(db)
    .await?;
    
    Ok(result.map(|r| r.content))
}
```

### 7.4 Token Tracking Per Surface

Track token usage separately per surface:

```rust
pub struct SurfaceTokenUsage {
    pub surface: Surface,
    pub input_tokens: i64,
    pub output_tokens: i64,
    pub total_cost_usd: f64,
}

pub async fn track_surface_usage(
    db: &Database,
    session_id: Uuid,
    surface: Surface,
    usage: SurfaceTokenUsage,
) -> Result<(), Error> {
    // Update token_usage table with surface-specific tracking
    sqlx::query!(
        r#"
        INSERT INTO token_usage 
        (id, session_id, surface, provider, model, input_tokens, output_tokens, total_cost, created_at)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, NOW(6))
        "#,
        Uuid::new_v4().to_string(),
        session_id.to_string(),
        format!("{:?}", surface).to_lowercase(),
        usage.provider,
        usage.model,
        usage.input_tokens,
        usage.output_tokens,
        usage.total_cost_usd
    )
    .execute(db)
    .await?;
    
    Ok(())
}
```

### 7.5 Integration with Existing AI Cortex

The Surface Agent Manager extends the existing AI Cortex infrastructure:

**Existing:**
- Multi-provider API support (Claude, OpenAI, Google)
- Token usage tracking
- Context window monitoring
- API configuration management

**New:**
- Per-surface agent selection
- Database-mediated artifact sharing
- Independent token budgets per surface
- No real-time coordination required

This maintains simplicity while enabling powerful multi-agent workflows.
```

**Copy-Paste Prompt for Execution:**

```
Please update ARTIFACT_04 (Unified AI Layer) by adding Section 7 "Surface-Specific Agent Management" as shown above. This extends the AI Cortex with tri-surface support. Add it after the existing sections. Update version from current to next MINOR version (e.g., if currently v1.1.0, make it v1.2.0). Maintain all existing content.
```

---

### Task 3: Update ARTIFACT_06 (Master Control Dashboard)
**Status:** âš ï¸ NOT STARTED  
**Estimated Time:** 1 hour  
**Priority:** HIGH

**What to add:**

```markdown
## 8. Tri-Surface Chat Workbench Module

### 8.1 Module Overview

The Tri-Surface Chat Workbench is a **universal chat module** embedded in all dashboards with three independent surfaces:

- **Inputs Surface** (left): Prompt engineering and instruction crafting
- **Reasoning Surface** (center): Main conversation and problem-solving
- **Outputs Surface** (right): Export formatting and publishing

**Default UX:** One surface open at a time (simple mode)  
**Advanced UX:** Multi-surface view with resizable columns

### 8.2 Elm Model Architecture

```elm
-- Surface types
type Surface
    = Inputs
    | Reasoning
    | Outputs

-- Layout modes
type LayoutMode
    = Simple  -- One surface at a time
    | Advanced  -- Multiple surfaces visible

-- Workbench model
type alias WorkbenchModel =
    { activeSurface : Surface
    , layoutMode : LayoutMode
    , sessionId : String
    , inputs : InputsModel
    , reasoning : ReasoningModel
    , outputs : OutputsModel
    , selection : SelectionModel
    , layoutConfig : LayoutConfig
    }

-- Inputs surface model
type alias InputsModel =
    { promptBuilder : PromptBuilderState
    , activeModules : List ModuleChip
    , agentId : Maybe String
    }

-- Reasoning surface model
type alias ReasoningModel =
    { messages : List ChatMessage
    , selectedMessages : Set String
    , composer : ComposerState
    , agentId : Maybe String
    }

-- Outputs surface model
type alias OutputsModel =
    { selectionSummary : SelectionSummary
    , selectedTemplate : Maybe TemplateId
    , preview : Maybe String
    , destination : Maybe DestinationId
    , agentId : Maybe String
    }

-- Selection tracking
type alias SelectionModel =
    { count : Int
    , totalTokens : Int
    , items : List SelectionItem
    }

-- Layout configuration (for Edit Mode)
type alias LayoutConfig =
    { leftRail : List ComposerControl
    , rightRail : List ComposerControl
    , secondaryStrip : List ComposerControl
    , hidden : List ComposerControl
    }
```

### 8.3 Surface Switching Logic

```elm
type Msg
    = SwitchSurface Surface
    | ToggleLayoutMode
    | SaveArtifact ArtifactType String
    | LoadArtifact ArtifactType
    | SelectMessage MessageId Bool
    | SelectBlock MessageId BlockId Bool
    | ExportSelection TemplateId DestinationId
    | UpdateLayoutConfig LayoutConfig
    | EnterLayoutEditMode
    | ExitLayoutEditMode

update : Msg -> WorkbenchModel -> ( WorkbenchModel, Cmd Msg )
update msg model =
    case msg of
        SwitchSurface surface ->
            ( { model | activeSurface = surface }
            , Cmd.none
            )
        
        SaveArtifact artifactType content ->
            ( model
            , saveArtifactToDb
                { sessionId = model.sessionId
                , artifactType = artifactType
                , content = content
                , surface = model.activeSurface
                }
            )
        
        LoadArtifact artifactType ->
            ( model
            , loadArtifactFromDb model.sessionId artifactType
            )
        
        SelectMessage messageId isSelected ->
            let
                newSelection =
                    if isSelected then
                        Set.insert messageId model.reasoning.selectedMessages
                    else
                        Set.remove messageId model.reasoning.selectedMessages
                
                updatedReasoning =
                    { reasoning | selectedMessages = newSelection }
            in
            ( { model | reasoning = updatedReasoning }
            , updateSelectionSummary newSelection
            )
        
        ExportSelection templateId destinationId ->
            ( model
            , createExportJob
                { sessionId = model.sessionId
                , selection = model.selection
                , templateId = templateId
                , destinationId = destinationId
                }
            )
        
        UpdateLayoutConfig config ->
            ( { model | layoutConfig = config }
            , saveLayoutConfig config
            )
        
        _ ->
            ( model, Cmd.none )
```

### 8.4 View Components

```elm
view : WorkbenchModel -> Html Msg
view model =
    div [ class "chat-workbench" ]
        [ viewHeader model
        , viewSurfacePills model.activeSurface
        , viewSurfacesContainer model
        , viewBinBar model.pendingChanges
        ]

viewSurfacePills : Surface -> Html Msg
viewSurfacePills activeSurface =
    div [ class "surface-pills" ]
        [ pill Inputs (activeSurface == Inputs)
        , pill Reasoning (activeSurface == Reasoning)
        , pill Outputs (activeSurface == Outputs)
        ]

pill : Surface -> Bool -> Html Msg
pill surface isActive =
    button
        [ class (if isActive then "pill active" else "pill")
        , onClick (SwitchSurface surface)
        ]
        [ text (surfaceName surface) ]

viewSurfacesContainer : WorkbenchModel -> Html Msg
viewSurfacesContainer model =
    div [ class "surfaces-container" ]
        [ collapsedLabel Inputs (model.activeSurface /= Inputs)
        , activeSurfaceView model
        , collapsedLabel Outputs (model.activeSurface /= Outputs)
        ]

activeSurfaceView : WorkbenchModel -> Html Msg
activeSurfaceView model =
    case model.activeSurface of
        Inputs ->
            viewInputsSurface model.inputs
        
        Reasoning ->
            viewReasoningSurface model.reasoning
        
        Outputs ->
            viewOutputsSurface model.outputs
```

### 8.5 Progressive Disclosure Features

**Phase 1: Basic Chat** (Week 8)
- Single-line composer with auto-grow
- Send button
- File attachment

**Phase 2: Context Injection** (Week 9)
- `@` mentions for files/folders
- Context chips
- Token counter

**Phase 3: Slash Commands** (Week 9)
- `/` command palette
- Command categories
- Parameter prompts

**Phase 4: Mode Selector** (Week 10)
- Ask/Edit/Agent/Plan modes
- Mode descriptions
- Default mode (Ask)

**Phase 5: Execution Toggles** (Week 10)
- Web toggle
- Fast/Deep effort toggle
- Status indicators

**Phase 6: Agent Run Controls** (Week 11)
- Send/Stop morphing
- Progress indicator
- Queue management

**Phase 7: Layout Edit Mode** (Week 11)
- Drag-and-drop customization
- Rail-based constraints
- Layout persistence

### 8.6 Layout Edit Mode Specification

**Edit Mode Toggle:**
- Button location: Top-right of chat header
- Label: "Edit Layout" or pencil icon
- Toggles between locked and editing states

**Draggable Rails:**

*Rail 1 - Left Cluster (Setup):*
- Prompt Control Hub icon
- Add/Attach (`+ Tools`)
- Mode selector (Ask/Edit/Agent/Plan)

*Rail 2 - Right Cluster (Execution):*
- Web toggle
- Fast/Deep toggle
- Send button (FIXED - not draggable)

*Rail 3 - Secondary Strip:*
- Token counter
- Queue indicator
- Project selector
- Hidden controls tray

**Edit Mode Behavior:**
- Buttons show drag handles
- Drop zones highlight
- Invalid placements show error
- Save/Cancel/Reset controls appear

**Persistence:**
```elm
type alias LayoutConfig =
    { userId : String
    , layoutName : String
    , configuration :
        { leftRail : List String
        , rightRail : List String
        , secondary : List String
        , hidden : List String
        }
    , isActive : Bool
    }

saveLayoutConfig : LayoutConfig -> Cmd Msg
saveLayoutConfig config =
    Http.post
        { url = "/api/composer/layouts"
        , body = Http.jsonBody (encodeLayoutConfig config)
        , expect = Http.expectJson LayoutConfigSaved layoutConfigDecoder
        }
```

### 8.7 SSE Integration

Real-time updates for:
- Export job status
- Selection changes
- Agent switching
- Layout updates

```elm
subscriptions : WorkbenchModel -> Sub Msg
subscriptions model =
    Sub.batch
        [ sseEvents "chat-workbench" handleSSEEvent
        , sseEvents "export-jobs" handleExportEvent
        ]

handleSSEEvent : SSEEvent -> Msg
handleSSEEvent event =
    case event.eventType of
        "selection_updated" ->
            UpdateSelection (decodeSelection event.data)
        
        "export_completed" ->
            ExportCompleted (decodeExportResult event.data)
        
        "agent_switched" ->
            AgentSwitched (decodeAgentSwitch event.data)
        
        _ ->
            NoOp
```

### 8.8 Integration with Master Control Dashboard

The Tri-Surface Chat Workbench appears as a module in all dashboards:

```elm
type DashboardModule
    = MetadataGovernancePanel
    | AICortexManagementPanel
    | ExtensionOrchestrationPanel
    | TriSurfaceChatWorkbench  -- NEW
    | ...

renderModule : DashboardModule -> Model -> Html Msg
renderModule module model =
    case module of
        TriSurfaceChatWorkbench ->
            Html.map WorkbenchMsg (Workbench.view model.workbench)
        
        _ ->
            ...
```

### 8.9 Accessibility

- **Keyboard shortcuts:**
  - `Cmd/Ctrl + 1/2/3`: Switch to Inputs/Reasoning/Outputs
  - `Cmd/Ctrl + E`: Toggle Edit Layout Mode
  - `Cmd/Ctrl + S`: Save current work
  
- **ARIA labels:**
  - Surface pills: `role="tablist"`
  - Active surface: `aria-selected="true"`
  - Checkboxes: `aria-label="Select message for export"`
  
- **Focus management:**
  - Focus trapped in modals
  - Tab order follows visual layout
  - Skip links for keyboard users

### 8.10 Performance Considerations

- **Lazy loading:** Load surface content only when activated
- **Virtual scrolling:** For large message lists
- **Debounced selection:** Update selection summary after 300ms
- **Optimized renders:** React only to changed props
```

**Copy-Paste Prompt for Execution:**

```
Please update ARTIFACT_06 (Master Control Dashboard) by adding Section 8 "Tri-Surface Chat Workbench Module" as shown above. This adds the workbench as a universal module available in all dashboards. Add it after existing sections. Update version to next MINOR version. Include all Elm code examples and integration patterns.
```

---

### Task 4: Update ARTIFACT_19 (Implementation Plan) - MAIN TASK
**Status:** âš ï¸ NOT STARTED  
**Estimated Time:** 4-5 hours (first pass) + 1.5-2 hours (second pass)  
**Priority:** âš¡ CRITICAL - This is the TIER 3 deliverable

**What to do:**

This is the main TIER 3 task. You need to:

1. Review ALL updated artifacts (05, 04, 06)
2. Review visual mockups (images + video)
3. Synthesize into cohesive 3.5-month timeline
4. Add tri-surface workbench to Month 2-3
5. Add Layout Edit Mode to Week 11
6. Update resource allocation
7. Update budget projections
8. Add risk assessment

**Copy-Paste Prompt for First Pass:**

```
Please update ARTIFACT_19 (DevGuide Cockpit Implementation Plan) from v2.0 to v2.1.0 by integrating the Tri-Surface Chat Workbench feature into the 3-month timeline.

CONTEXT TO REVIEW:
1. ARTIFACT_05 v1.2.0 (7 new tables for chat workbench)
2. ARTIFACT_04 (Surface agent management)
3. ARTIFACT_06 (Tri-surface workbench module)
4. Visual mockups showing UI design
5. Layout Edit Mode specification
6. Progressive Disclosure phases (1-7)

CHANGES REQUIRED:

Month 2 Updates:
- Add Week 8-9: Tri-Surface Chat UI (2 weeks)
  - Day 36-37: Surface switching system (3 pills, sliding panels, collapsed indicators)
  - Day 38-39: Inputs Surface (Prompt Builder with structured fields)
  - Day 40-41: Reasoning Surface (Chat with checkboxes, selection counter)
  - Day 42-43: Outputs Surface (Selection summary, format dropdown, preview, destinations)
  - Day 44-45: Database integration (save/load artifacts via session_artifacts table)

Month 3 Updates:
- Add Week 10: Export System (1 week)
  - Day 46-47: Template Formatters (Markdown, YAML, Skill templates)
  - Day 48-49: Destination Handlers (Download, GitHub PR, TiDB, Drive)
  - Day 50: Export Pipeline (validation, The Bin integration)

- Add Week 11: Progressive Disclosure Features (1 week)
  - Day 51: Layout Edit Mode (drag-and-drop customization, rail constraints)
  - Day 52: Advanced Composer Controls (Mode selector, Web/Effort toggles)
  - Day 53-54: Polish & Testing (keyboard shortcuts, accessibility, animations)

Resources:
- Same 7 engineers (no hiring needed)
- Complexity: MEDIUM (not ULTRA HIGH - database-mediated, not orchestrated)

Budget:
- MVP: $180/month (+$30 from baseline)
- Production: $1,600/month (+$300 from baseline)

Timeline:
- Original: 3 months
- Updated: 3.5 months (3 weeks added for tri-surface workbench)

Success Metrics:
- Add: User adoption of surface switching
- Add: Export template usage
- Add: Layout customization engagement

Use TIER_3_IMPLEMENTATION_PROTOCOL.md as reference for structure. Document all timeline changes in TIMELINE_SYNTHESIS_NOTES.md (create this file). Maintain Month 1 unchanged (Metadata + AI Cortex + Extensions).
```

**Copy-Paste Prompt for Second Pass (Validation):**

```
Please validate ARTIFACT_19 v2.1.0 using CONTINUATION_HANDOFF_ARTIFACT_19_SECOND_PASS.md checklist:

VALIDATION CHECKS:
1. Completeness: All 8 artifacts represented in timeline
   - ARTIFACT_05 (Data Model) - 7 new tables deployed Week 8
   - ARTIFACT_04 (AI Layer) - Surface agents integrated Week 8
   - ARTIFACT_06 (Master Control) - Workbench module Week 8-11
   - ARTIFACT_03 (The Bin) - Export validation Week 10
   - ARTIFACT_13 (Security) - Unchanged (Month 1-2)
   - ARTIFACT_16 (Tech Stack) - Unchanged (Month 1)
   - ARTIFACT_20 (Metadata Governance) - Unchanged (Month 1)
   - NEW: Tri-Surface Workbench (Week 8-11)

2. Timeline Realism: Tasks fit in allocated time
   - 2 weeks for UI development realistic? (6 engineers)
   - 1 week for export system realistic?
   - 1 week for progressive disclosure realistic?
   - Buffer periods included?

3. Resource Allocation: 7 engineers properly utilized
   - Frontend: 2 Elm engineers on UI (Week 8-11)
   - Backend: 2 Go engineers on APIs (Week 8-11)
   - Core: 1 Rust engineer on The Bin (Week 10)
   - Infrastructure: 2 DevOps engineers on monitoring
   - No overallocation?

4. Dependency Verification: Correct prerequisite order
   - Database tables before UI (Week 8 Day 36 before Day 38)
   - API Layer before Export (Week 8-9 before Week 10)
   - Export before Polish (Week 10 before Week 11)

5. Risk Assessment: 3 new systems covered
   - Tri-Surface UI complexity risk
   - Export system integration risk
   - Layout Edit Mode adoption risk
   - Mitigation strategies documented?

6. Success Metrics: New systems measurable
   - Surface switching adoption
   - Export usage rates
   - Layout customization engagement

Create TIMELINE_VALIDATION_REPORT.md with findings. Make corrections if needed and update to v2.1.1 if changes required.
```

---

### Task 5: Create Supporting Documentation
**Status:** âš ï¸ NOT STARTED  
**Estimated Time:** 1 hour  
**Priority:** MEDIUM

**Documents to create:**

1. **COMPOSER_UX_SPECIFICATION.md**
   - Progressive disclosure phases (1-7)
   - Component inventory
   - Keyboard shortcuts
   - Accessibility requirements

2. **LAYOUT_EDIT_MODE_SPECIFICATION.md**
   - Drag-and-drop behavior
   - Rail constraints
   - Persistence model
   - Keyboard controls

3. **EXPORT_PIPELINE_SPECIFICATION.md**
   - Template formatter interface
   - Destination handler interface
   - Validation rules
   - The Bin integration

**Copy-Paste Prompt:**

```
Please create three supporting specification documents for the Tri-Surface Chat Workbench:

1. COMPOSER_UX_SPECIFICATION.md - Full progressive disclosure specification
2. LAYOUT_EDIT_MODE_SPECIFICATION.md - Drag-and-drop customization spec
3. EXPORT_PIPELINE_SPECIFICATION.md - Export system technical spec

Use the content from the uploaded documents and my analysis as source material. Format as technical specifications with:
- Purpose and scope
- Requirements (functional + non-functional)
- Architecture diagrams (Markdown/ASCII)
- Implementation notes
- Testing criteria

Save to /mnt/user-data/outputs/
```

---

## âš¡ EXECUTION ORDER

### Session 1: Database & Architecture (2 hours)
1. Task 1: Update ARTIFACT_05 (1 hour)
2. Task 2: Update ARTIFACT_04 (45 minutes)
3. Task 3: Update ARTIFACT_06 (1 hour)

### Session 2: Implementation Plan - First Pass (4-5 hours)
4. Task 4 Part 1: Update ARTIFACT_19 first pass (4-5 hours)
   - Use UltraThink mode
   - Review all updated artifacts
   - Synthesize timeline
   - Create TIMELINE_SYNTHESIS_NOTES.md

### Session 3: Validation - Second Pass (2 hours)
5. Task 4 Part 2: Validate ARTIFACT_19 (1.5-2 hours)
   - Check completeness
   - Verify timeline realism
   - Validate resources
   - Create TIMELINE_VALIDATION_REPORT.md
   - Apply corrections if needed

### Session 4: Documentation (1 hour)
6. Task 5: Create supporting specs (1 hour)

**TOTAL TIME: 9-11 hours across 4 sessions**

---

## ðŸ" SUMMARY

**What's Complete:** âœ…
- Analysis and planning
- Python export framework
- Visual review
- Architecture correction
- Supporting documentation

**What Remains:** âš ï¸
- Update 3 artifacts (05, 04, 06) - 2.75 hours
- Update ARTIFACT_19 (first pass) - 4-5 hours
- Validate ARTIFACT_19 (second pass) - 1.5-2 hours
- Create supporting specs - 1 hour

**Total Remaining: 9-11 hours**

**Ready to begin:** YES âœ…  
**Context available:** 48.3% (91.8K / 190K tokens)  
**Confidence level:** HIGH

---

*Generated: 2026-01-02T20:30:00Z*  
*Status: âœ… READY TO EXECUTE*  
*Next Step: Begin Session 1 (Tasks 1-3)*
