# TIER 3 Completion Checklist - Visual Confirmation

**Date:** 2026-01-02T20:15:00Z  
**Status:** âœ… VISUALS REVIEWED - Ready to finalize TIER 3  
**Context Used:** 59.7% (113.5K / 190K tokens)

---

## âœ… WHAT I SEE IN YOUR VISUALS

### UI Components Confirmed

**From Images:**
1. **Three Surface Pills** (top center): INPUTS | REASONING | OUTPUTS
   - Active state: Orange highlight
   - Inactive states: Gray
   - Click to switch surfaces

2. **Collapsed Surface Indicators**
   - Left edge: Vertical "INPUTS" text
   - Right edge: Vertical "OUTPUTS" text
   - Shows which surfaces are available when collapsed

3. **Reasoning Surface (Default/Center)**
   - Chat conversation with checkboxes for selection
   - Message-level selection (orange checkboxes)
   - Composer at bottom with:
     - `+ Tools` button (left)
     - Input field: "Type @ to add context, / for commands"
     - `Web` toggle
     - `Fast` toggle
     - `Send` button (orange)
   - Context chips below composer: `@ PRD.md`, `@ prompt_spec.yaml`, `Skill: citations`, `Mode: Ask`

4. **Inputs Surface (Left)**
   - **Prompt Builder** with structured fields:
     - ROLE: "Senior engineer reviewer"
     - OBJECTIVE: "Draft a production-safe prompt stack"
     - CONSTRAINTS: "JSON output, no hallucinations"
     - EXAMPLES: "Add 1-2 few-shot pairs"
     - OUTPUT CONTRACT: "YAML front matter + Markdown body"
   - **Active Modules** chips at bottom
   - `Send to Reasoning →` button (orange)

5. **Outputs Surface (Right)**
   - **SELECTION**: "3 blocks • ~1.2k tokens"
   - **FORMAT**: Dropdown showing "Markdown + YAML front matter"
   - **AUTO-APPLY FORMAT AFTER RUN**: Toggle (on)
   - **PREVIEW**: Code preview box showing formatted output
   - **DESTINATION**: Buttons for `Download (.md)`, `GitHub PR`, `TiDB`, `Drive`
   - `Preview` button and `Publish →` button (orange)

6. **The Bin** (Bottom, Always Visible)
   - "2 Pending" indicator (orange)
   - "⚠️ Update formatter: markdown_yaml_v2 (Impact: High) [View Diff] [Accept] [Reject]"

7. **Token Counter** (Top Right)
   - "Token: 45K/100K"

---

## âœ… WHAT THIS CONFIRMS

### 1. Architecture = Simple (Database-Mediated)

**Flow I see in the UI:**
```
Inputs Surface:
  User fills prompt builder
  Clicks "Send to Reasoning →"
  → Saves to session_artifacts table

Reasoning Surface:
  Loads draft from database (optional)
  User chats with agent
  Selects messages with checkboxes
  → Saves selection to database

Outputs Surface:
  Loads selected blocks from database
  Applies template format
  Shows preview
  User clicks "Publish →"
  → Exports to destination
```

**No real-time orchestration!** Just save/load from database. âœ…

### 2. Complexity = MEDIUM (Confirmed)

**What needs to be built:**
- Three chat panels (Elm components)
- Sliding animation (CSS transitions)
- Checkbox selection system
- Template formatter system (Python)
- Destination connectors (Python)
- Database integration (save/load artifacts)

**NOT needed:**
- âŒ Agent-to-agent messaging
- âŒ Real-time coordination
- âŒ Complex orchestration protocol
- âŒ Token allocation across agents

### 3. Timeline = +2-3 Weeks (Confirmed)

**Week 8-9: Tri-Surface UI**
- Three panel layout with sliding transitions
- Surface switching logic
- Collapsed state indicators
- Agent selection per surface

**Week 10: Export System**
- Checkbox selection
- Format templates
- Destination handlers
- Publish pipeline

**Total Addition: 3 weeks** to Month 2-3 timeline

---

## ðŸ"‹ TIER 3 COMPLETION TASKS

### Task 1: Update ARTIFACT_05 (Data Model) âš ï¸ NOT DONE YET

**Add to existing schema:**

```sql
-- Chat sessions
CREATE TABLE chat_sessions (
  id VARCHAR(36) PRIMARY KEY,
  project_id VARCHAR(36),
  title VARCHAR(255),
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE
);

-- Messages per surface
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
  INDEX idx_session_surface (session_id, surface)
);

-- Session artifacts (database clipboard)
CREATE TABLE session_artifacts (
  id VARCHAR(36) PRIMARY KEY,
  session_id VARCHAR(36) NOT NULL,
  artifact_type VARCHAR(64) NOT NULL, -- prompt_draft|reasoning_result|selected_blocks
  content LONGTEXT,
  metadata JSON, -- provenance, token count, etc.
  created_from_surface VARCHAR(32), -- which surface created it
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  FOREIGN KEY (session_id) REFERENCES chat_sessions(id) ON DELETE CASCADE,
  INDEX idx_session_type (session_id, artifact_type)
);

-- Export templates (formatters)
CREATE TABLE export_templates (
  id VARCHAR(36) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  template_type VARCHAR(64), -- markdown|yaml_markdown|skill|custom
  template_content TEXT, -- Python script or template string
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  version INT NOT NULL DEFAULT 1
);

-- Export destinations
CREATE TABLE export_destinations (
  id VARCHAR(36) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  destination_type VARCHAR(64), -- download|github|tidb|drive
  configuration JSON, -- destination-specific config
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6)
);

-- Export jobs (for tracking)
CREATE TABLE export_jobs (
  id VARCHAR(36) PRIMARY KEY,
  session_id VARCHAR(36) NOT NULL,
  selection_set_id VARCHAR(36),
  template_id VARCHAR(36),
  destination_id VARCHAR(36),
  status VARCHAR(32) NOT NULL, -- queued|running|completed|failed
  output_location TEXT, -- where file was saved
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  completed_at TIMESTAMP(6),
  FOREIGN KEY (session_id) REFERENCES chat_sessions(id) ON DELETE CASCADE,
  INDEX idx_status (status)
);
```

**Total New Tables: 6**  
**Total Project Tables: 18 (existing) + 6 (new) = 24 tables**

**Action Required:** âš ï¸ Add these to ARTIFACT_05 in TIER 3 execution

---

### Task 2: Update ARTIFACT_04 (AI Layer) âš ï¸ NOT DONE YET

**Add surface-specific agent management:**

```rust
// Surface agent configuration
pub struct SurfaceAgent {
    pub surface: Surface, // Inputs | Reasoning | Outputs
    pub agent_profile_id: Uuid,
    pub provider: String, // claude|openai|google
    pub model: String, // claude-opus-4|gpt-4|gemini-pro
}

// Simple agent manager (no orchestration!)
pub struct SurfaceAgentManager {
    pub input_agent: Option<SurfaceAgent>,
    pub reasoning_agent: Option<SurfaceAgent>,
    pub output_agent: Option<SurfaceAgent>,
}

impl SurfaceAgentManager {
    // User selects agent for a surface
    pub fn set_agent_for_surface(&mut self, surface: Surface, agent: SurfaceAgent) {
        match surface {
            Surface::Inputs => self.input_agent = Some(agent),
            Surface::Reasoning => self.reasoning_agent = Some(agent),
            Surface::Outputs => self.output_agent = Some(agent),
        }
    }
    
    // Get active agent for current surface
    pub fn get_agent_for_surface(&self, surface: Surface) -> Option<&SurfaceAgent> {
        match surface {
            Surface::Inputs => self.input_agent.as_ref(),
            Surface::Reasoning => self.reasoning_agent.as_ref(),
            Surface::Outputs => self.output_agent.as_ref(),
        }
    }
}
```

**Action Required:** âš ï¸ Add these to ARTIFACT_04 in TIER 3 execution

---

### Task 3: Update ARTIFACT_06 (Master Control) âš ï¸ NOT DONE YET

**Add Tri-Surface Chat Workbench module:**

```elm
-- Surface type
type Surface
    = Inputs
    | Reasoning
    | Outputs

-- Workbench model
type alias WorkbenchModel =
    { activeSurface : Surface
    , inputsModel : InputsModel
    , reasoningModel : ReasoningModel
    , outputsModel : OutputsModel
    , sessionId : String
    }

-- Surface switching
type Msg
    = SwitchSurface Surface
    | SaveArtifact ArtifactType String
    | LoadArtifact ArtifactType
    | SelectMessage MessageId
    | ExportSelection TemplateId DestinationId

-- View with sliding panels
view : WorkbenchModel -> Html Msg
view model =
    div [ class "chat-workbench" ]
        [ surfacePills model.activeSurface
        , div [ class "surfaces-container" ]
            [ collapsedLabel Inputs (model.activeSurface /= Inputs)
            , activeSurfaceView model
            , collapsedLabel Outputs (model.activeSurface /= Outputs)
            ]
        , binBar model.pendingChanges
        ]
```

**Action Required:** âš ï¸ Add these to ARTIFACT_06 in TIER 3 execution

---

### Task 4: Update ARTIFACT_19 (Implementation Plan) âš ï¸ MAIN TASK

**This is the TIER 3 primary deliverable.**

**Additions to Month 2:**

**Week 8-9: Tri-Surface Chat UI** (NEW - 2 weeks)
- Day 36-37: Surface switching system
  - Three panel layout (Elm)
  - Sliding transitions (CSS)
  - Surface pills component
  - Collapsed state indicators
  
- Day 38-39: Inputs Surface
  - Prompt Builder form
  - Structured fields (Role, Objective, Constraints, etc.)
  - Active modules display
  - "Send to Reasoning" button with database save
  
- Day 40-41: Reasoning Surface  
  - Chat conversation display
  - Message-level checkboxes
  - Block-level selection (optional)
  - Selection counter
  - Composer with @ and / triggers
  
- Day 42-43: Outputs Surface
  - Selection summary display
  - Format template dropdown
  - Preview panel
  - Destination button group
  - Publish workflow
  
- Day 44-45: Database Integration
  - SessionArtifactManager (Python)
  - Save/load operations
  - Artifact provenance tracking

**Week 10: Export System** (NEW - 1 week)
- Day 46-47: Template Formatters
  - MarkdownTemplate class
  - YAMLFrontMatterTemplate class
  - SkillFileTemplate class
  - Custom template support
  
- Day 48-49: Destination Handlers
  - LocalFileDestination (download)
  - GitHubDestination (PR creation)
  - TiDBDestination (database save)
  - DriveDestination (if enabled)
  
- Day 50: Export Pipeline
  - ExportPipeline orchestrator
  - Validation system
  - Error handling
  - The Bin integration

**Week 11: Progressive Disclosure Features** (NEW - 1 week)
- Day 51: Layout Edit Mode
  - Drag-and-drop customization
  - Rail-based constraints
  - Layout persistence
  
- Day 52: Advanced Composer Controls
  - Mode selector (Ask/Edit/Agent/Plan)
  - Web/Effort toggles
  - Context chips management
  
- Day 53-54: Polish & Testing
  - Keyboard shortcuts
  - Accessibility (ARIA labels, focus management)
  - Animation performance
  - Cross-browser testing

**Total Addition: 3 weeks** (Week 8-11)

**Action Required:** âœ… This is the MAIN TIER 3 task - Update ARTIFACT_19 with revised timeline

---

### Task 5: Create Supporting Documentation âš ï¸ NOT DONE YET

**New artifacts to create:**

1. **COMPOSER_UX_SPECIFICATION.md**
   - Progressive disclosure phases (1-7)
   - Layout Edit Mode specification
   - Keyboard shortcuts
   - Accessibility requirements
   - Component inventory

2. **PYTHON_EXPORT_FRAMEWORK.md**
   - Template base classes
   - Destination handlers
   - Export pipeline
   - Usage examples
   - (Already created in corrected analysis - can reference)

3. **TRI_SURFACE_UI_PATTERNS.md**
   - Sliding panel animations
   - Surface switching logic
   - Collapsed state indicators
   - Token counter integration

**Action Required:** âœ… Create these as reference documentation

---

## ðŸŽ¯ LAYOUT EDIT MODE SPECIFICATION

### Feature: Drag-and-Drop Composer Customization

**User Request:** "Add a Layout Edit Mode that lets users re-order the composer controls"

**Implementation in TIER 3:**

#### 1. Edit Layout Button
- **Location:** Top-right of chat panel header (near settings)
- **Label:** "Edit Layout" or pencil icon
- **Toggle:** ON = editing, OFF = locked

#### 2. Draggable Rails

**Rail 1 - Left Cluster (Setup):**
- Prompt Control Hub icon (optional position)
- Add/Attach (`+ Tools`) (optional position)
- Mode selector (Ask/Edit/Agent/Plan) (optional position)

**Rail 2 - Right Cluster (Execution):**
- Web toggle (draggable)
- Fast/Deep toggle (draggable)
- Send button (FIXED - not draggable)

**Rail 3 - Secondary Strip (Optional):**
- Token counter
- Queue indicator
- Project selector
- Hidden controls tray

#### 3. Edit Mode UX

**When user clicks "Edit Layout":**
- UI enters reorder state
- Buttons show drag handles
- Drop zones highlight
- Control bar appears:
  - `Save` button
  - `Cancel` button
  - `Reset to Default` button

**Drag behavior:**
- Snap-to-slot ordering (not pixel-perfect)
- Live insertion indicator
- Invalid placements show error: "Send is fixed"
- Option to drag to "Hidden" tray

**Keyboard controls:**
- Arrow keys: Move selected item
- Enter: Pick up / drop
- Esc: Cancel

#### 4. Layout Persistence

**Storage:**
```json
{
  "user_id": "user-123",
  "layout_config": {
    "left_rail": ["hub", "mode", "attach"],
    "right_rail": ["web", "effort"],
    "secondary": ["tokens", "queue"],
    "hidden": []
  },
  "created_at": "2026-01-02T20:00:00Z"
}
```

**Database table:**
```sql
CREATE TABLE user_composer_layouts (
  id VARCHAR(36) PRIMARY KEY,
  user_id VARCHAR(36) NOT NULL,
  layout_name VARCHAR(255),
  configuration JSON NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  INDEX idx_user (user_id)
);
```

#### 5. Implementation Timeline

**Add to Week 11 (Day 51):**
- Draggable component system (Elm)
- Rail constraints logic
- Layout persistence to database
- Reset to default function
- Import/export layout JSON

**Estimated effort:** 1 day

---

## âœ… SECOND PASS CHECKLIST

### What's Been Done

- [x] Initial TIER 1 & 2 analysis
- [x] Prompt Control Dashboard evaluation (deferred post-MVP)
- [x] Tri-Surface architecture correction (database-mediated)
- [x] Python export framework provided
- [x] Visual confirmation of UI design

### What Still Needs to Be Done for TIER 3

- [ ] **ARTIFACT_05 update** - Add 6 new database tables
- [ ] **ARTIFACT_04 update** - Add surface agent management
- [ ] **ARTIFACT_06 update** - Add tri-surface workbench module
- [ ] **ARTIFACT_19 update** - MAIN TASK - Revised 3.5 month timeline
  - [ ] Add Week 8-9: Tri-Surface UI (2 weeks)
  - [ ] Add Week 10: Export System (1 week)
  - [ ] Add Week 11: Progressive Disclosure + Layout Edit Mode (1 week)
  - [ ] Update resource allocation
  - [ ] Update budget projections
  - [ ] Add risk assessment for new UI complexity
- [ ] **Supporting docs** - Create UX specifications
- [ ] **Validation** - Second pass review of ARTIFACT_19

### Estimated Time to Complete TIER 3

**First Pass (ARTIFACT_19 synthesis):** 4-5 hours
- Review all visual materials
- Integrate tri-surface requirements
- Update Month 2-3 timeline
- Add Layout Edit Mode specification
- Budget and resource updates

**Second Pass (Validation):** 1.5-2 hours
- Verify all visuals represented
- Check timeline realism
- Validate resource allocation
- Confirm feasibility

**Total TIER 3 Time:** 5.5-7 hours (slightly longer due to visual integration)

---

## ðŸ'° UPDATED COST ESTIMATE (With Visuals Confirmed)

### MVP Phase (Month 1-3.5)
- Infrastructure: $100/month
- Clock Sync: $20/month
- AI Agents (3 surfaces, but users run 1 at a time): $60/month
- **Total: $180/month**

### Production Phase (Month 4+)
- Infrastructure: $1,100/month
- Monitoring: $150/month
- AI Scaling (3 surfaces): $350/month
- **Total: $1,600/month**

**No change from corrected estimate!** âœ…

---

## ðŸš€ READY TO EXECUTE TIER 3

**Status:** âœ… All prerequisites met
- âœ… Visuals reviewed and understood
- âœ… Architecture confirmed (database-mediated, simple)
- âœ… Timeline impact confirmed (+3 weeks)
- âœ… Layout Edit Mode specified
- âœ… Python framework provided
- âœ… Database schema designed

**Next Step:** Execute TIER 3 with UltraThink mode

**Deliverables:**
1. Updated ARTIFACT_05 (6 new tables)
2. Updated ARTIFACT_04 (surface agent management)
3. Updated ARTIFACT_06 (tri-surface workbench)
4. Updated ARTIFACT_19 (3.5 month timeline)
5. Supporting documentation (UX specs)

**Ready to proceed?** Say the word and I'll execute the full TIER 3 update!

---

*Generated: 2026-01-02T20:15:00Z*  
*Status: âœ… READY TO EXECUTE*  
*Context: 59.7% used (113.5K / 190K tokens)*
