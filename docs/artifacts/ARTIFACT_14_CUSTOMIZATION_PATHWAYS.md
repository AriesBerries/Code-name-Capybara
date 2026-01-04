# DevGuide Cockpit: Customization Pathways

**Document Type:** Governance & Security (Tier 4)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** TEMPLATE_SYSTEM_SPECIFICATION.md, DASHBOARD_TEMPLATE_CATALOG.md

---

## Executive Summary

This artifact specifies **how users customize dashboards and workflows** in DevGuide Cockpit without writing code. It defines:
- Configuration-based customization (YAML overlays)
- Module configuration options
- Workflow adaptation patterns
- AI-assisted customization vs manual configuration
- Visual dashboard builder (drag-and-drop)

**Key Principle:** Customize through configuration, not code modification

---

## Customization Hierarchy

```
Template (Immutable)
    ↓
Configuration Overlay (User-Editable)
    ↓
Runtime Behavior (Dynamically Adjusted)
```

**Example:**
```yaml
# Template default: max 5 features
template:
  dashboards:
    inception-001:
      max_features: 5

# User override: increase to 10
config_overrides:
  dashboards:
    inception-001:
      max_features: 10
```

---

## Customization Pathways

### 1. AI-Assisted Customization

**When to Use:** User wants to customize but doesn't know how

**Workflow:**
```
User: "I want to track bugs in my project"
AI: "I can add a Bug Tracker module to your Quality Dashboard. 
     It will include:
     • Bug entry form (title, severity, assignee)
     • Status tracker (open, in progress, resolved)
     • Priority sorting
     
     Sound good?"
User: "Yes, and I want email notifications"
AI: "Added! Configuring email notifications...
     What's your email server? (SMTP/SendGrid/AWS SES)"
User: "SendGrid"
AI: [Generates config]
     "Done! Bug tracker added with SendGrid integration."
```

**Implementation:**
```go
// api/customization/ai_assistant.go
func (ca *CustomizationAssistant) ProcessRequest(ctx context.Context, userRequest string, projectID string) (*CustomizationPlan, error) {
    // Use AI to understand intent
    intent, err := ca.aiLayer.AnalyzeIntent(ctx, AIRequest{
        Prompt: fmt.Sprintf(`
User request: %s
Project context: %s

What customization does the user want?
Respond in JSON:
{
  "dashboard": "quality-001",
  "action": "add_module",
  "module_type": "bug_tracker",
  "config": {...}
}
`, userRequest, ca.getProjectContext(projectID)),
        MaxTokens: 1000,
    })
    
    // Generate customization plan
    plan := ca.generatePlan(intent)
    
    // Show user preview
    return plan, nil
}
```

---

### 2. Manual Configuration Editing

**When to Use:** User knows exactly what to change

**UI: Configuration Editor**
```elm
viewConfigEditor : Config -> Html Msg
viewConfigEditor config =
    div [ class "config-editor" ]
        [ h3 [] [ text "Dashboard Configuration" ]
        , label []
            [ text "Max Features:"
            , input
                [ type_ "number"
                , value (String.fromInt config.maxFeatures)
                , onInput UpdateMaxFeatures
                , Html.Attributes.min "1"
                , Html.Attributes.max "20"
                ] []
            ]
        , label []
            [ input [ type_ "checkbox", checked config.requireSuccessMetrics, onCheck ToggleSuccessMetrics ] []
            , text "Require success metrics"
            ]
        , button [ onClick SaveConfig ] [ text "Save Changes" ]
        ]
```

---

### 3. Visual Dashboard Builder

**When to Use:** User wants to customize layout

**Features:**
- Drag-and-drop module placement
- Resize modules
- Hide/show optional sections
- Reorder workflow steps

**Implementation:**
```elm
-- Drag-and-drop builder
type Msg
    = StartDrag ModuleId
    | DragOver Position
    | Drop ModuleId Position

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        StartDrag moduleId ->
            ( { model | dragging = Just moduleId }
            , Cmd.none
            )
        
        DragOver position ->
            ( { model | dropTarget = Just position }
            , Cmd.none
            )
        
        Drop moduleId position ->
            let
                updatedLayout = moveModule moduleId position model.layout
            in
            ( { model 
                | layout = updatedLayout
                , dragging = Nothing
                , dropTarget = Nothing
              }
            , saveLayoutToServer updatedLayout
            )

viewDashboardBuilder : Model -> Html Msg
viewDashboardBuilder model =
    div [ class "dashboard-builder" ]
        [ div [ class "module-palette" ]
            (List.map viewModulePalette model.availableModules)
        , div 
            [ class "canvas"
            , onDragOver (\pos -> DragOver pos)
            ] 
            (List.map viewPlacedModule model.layout)
        ]
```

---

## Module Configuration Options

### Standard Modules

#### Checklist Module

```yaml
module:
  type: "checklist"
  config:
    allow_custom_items: true
    require_completion: false
    progress_tracking: true
    categories:
      - "Pre-Development"
      - "Development"
      - "Post-Development"
```

**Customization UI:**
- Add/remove checklist items
- Reorder items
- Mark items as required/optional
- Define custom categories

---

#### Code Snippet Module

```yaml
module:
  type: "code_snippet"
  config:
    languages: ["elm", "go", "rust", "python"]
    syntax_highlighting: true
    allow_execution: false  # Safety
    max_snippet_size_kb: 100
```

**Customization:**
- Enable/disable execution
- Add more languages
- Configure snippet templates

---

#### Decision Log Module

```yaml
module:
  type: "decision_log"
  config:
    require_rationale: true
    require_alternatives: true
    min_rationale_length: 100
    adr_format: "MADR"  # or "Y-Statements"
```

**Customization:**
- Change ADR format
- Add custom fields (e.g., "Stakeholders Consulted")
- Configure approval workflow

---

## Workflow Adaptation Patterns

### Pattern 1: Add Optional Step

**Use Case:** User wants to add code review step

**Before:**
```
Code Written → Testing → Deployment
```

**After:**
```
Code Written → Code Review → Testing → Deployment
```

**Implementation:**
```yaml
workflow_modifications:
  - action: "insert_step"
    after: "code_written"
    step:
      id: "code_review"
      name: "Code Review"
      required: true
      module: "code_review_module"
      config:
        reviewers_required: 2
        approval_threshold: "all"
```

---

### Pattern 2: Skip Optional Dashboard

**Use Case:** User doesn't need Security Dashboard

**Implementation:**
```yaml
config_overrides:
  dashboards:
    security-001:
      enabled: false  # Skip this dashboard
```

**UI Indicator:**
```elm
-- Dashboard appears grayed out in Master Control
viewDashboardCard : Dashboard -> Html Msg
viewDashboardCard dashboard =
    div 
        [ class (if dashboard.enabled then "dashboard-card" else "dashboard-card disabled") ]
        [ h3 [] [ text dashboard.name ]
        , if dashboard.enabled then
            button [ onClick (OpenDashboard dashboard.id) ] [ text "Open" ]
          else
            span [ class "disabled-label" ] [ text "Disabled" ]
        ]
```

---

### Pattern 3: Reorder Dashboards

**Use Case:** User wants to do security before deployment

**Before Sequence:**
```
1. Inception
2. Architecture
3. Tech Stack
4. Quality
5. Deployment
6. Security
```

**After Reordering:**
```
1. Inception
2. Architecture
3. Tech Stack
4. Security  ← Moved up
5. Quality
6. Deployment
```

**Implementation:**
```yaml
config_overrides:
  dashboard_sequence:
    - inception-001
    - architecture-001
    - techstack-001
    - security-001  # Reordered
    - quality-001
    - deployment-001
```

---

## AI-Assisted vs Manual Configuration

### Comparison

| Aspect | AI-Assisted | Manual Configuration |
|--------|-------------|---------------------|
| **Speed** | Fast (seconds) | Slower (minutes) |
| **Ease** | Natural language | Requires YAML knowledge |
| **Precision** | ~90% accurate | 100% accurate |
| **Use Case** | Common customizations | Edge cases, complex config |
| **Learning Curve** | None | Moderate |

### When to Use AI

✓ Adding common modules  
✓ Changing simple settings  
✓ Getting started quickly  
✓ Exploring possibilities

### When to Use Manual

✓ Fine-tuning exact values  
✓ Complex multi-step changes  
✓ Version-controlled config  
✓ Scripted deployments

---

## Customization Limits

### What Can Be Customized

✓ Module configurations  
✓ Dashboard layout  
✓ Workflow sequences  
✓ Optional dashboards (enable/disable)  
✓ Guardrail thresholds (with rationale)  
✓ AI prompts  
✓ Color schemes  
✓ Keyboard shortcuts

### What Cannot Be Customized

✗ Template source code (immutable)  
✗ Baseline guardrails (system-wide)  
✗ Required dashboards (enforced by template)  
✗ Capability requirements (security)  
✗ Core validation logic  
✗ The Bin behavior (synchronization guarantee)

---

## Customization Storage

### Configuration File

```yaml
# .devguide/customizations.yaml
project_id: "proj-123"
template_id: "vibe-coding-v1"
template_version: "1.0.0"

customizations:
  dashboards:
    inception-001:
      max_features: 10  # Override default 5
      custom_fields:
        - name: "target_market"
          type: "text"
          required: true
    
    security-001:
      enabled: true  # Was optional, now required
      strict_mode: true
  
  modules:
    checklist:
      custom_items:
        - "Accessibility audit"
        - "Load testing"
  
  guardrails:
    max_service_count:
      value: 5  # Override default 3
      rationale: "Complex e-commerce requires more services"
  
  ai_settings:
    monthly_token_budget: 2000000  # Override default 1M

metadata:
  created_at: "2026-01-01T10:00:00Z"
  last_modified: "2026-01-15T14:30:00Z"
  modified_by: "user-456"
```

---

## Testing Customizations

### Preview Mode

**Before Applying:**
```elm
viewCustomizationPreview : CustomizationPlan -> Html Msg
viewCustomizationPreview plan =
    div [ class "preview-modal" ]
        [ h2 [] [ text "Preview Changes" ]
        , div [ class "before-after" ]
            [ div [ class "before" ]
                [ h3 [] [ text "Current" ]
                , viewCurrentConfig plan.currentConfig
                ]
            , div [ class "after" ]
                [ h3 [] [ text "After Applying" ]
                , viewNewConfig plan.newConfig
                ]
            ]
        , div [ class "actions" ]
            [ button [ onClick ApplyCustomization ] [ text "Apply" ]
            , button [ onClick CancelCustomization ] [ text "Cancel" ]
            ]
        ]
```

### Rollback Capability

```go
// api/customization/manager.go
func (cm *CustomizationManager) ApplyCustomization(ctx context.Context, projectID string, plan CustomizationPlan) error {
    // Create snapshot before applying
    snapshot, err := cm.createSnapshot(ctx, projectID)
    if err != nil {
        return err
    }
    
    // Apply customization
    err = cm.applyPlan(ctx, projectID, plan)
    if err != nil {
        // Rollback on failure
        cm.restoreSnapshot(ctx, snapshot)
        return err
    }
    
    // Store rollback point (30-day retention)
    return cm.saveRollbackPoint(ctx, projectID, snapshot)
}
```

---

## Customization Examples

### Example 1: Add Email Notifications

**User Request:** "Send me email when deployment completes"

**AI-Generated Config:**
```yaml
config_overrides:
  dashboards:
    deployment-001:
      notifications:
        enabled: true
        channels:
          - type: "email"
            recipients: ["user@example.com"]
            events: ["deployment_success", "deployment_failure"]
```

---

### Example 2: Custom Checklist

**User Request:** "Add accessibility items to my checklist"

**Manual Configuration:**
```yaml
modules:
  checklist:
    custom_items:
      - text: "Run WAVE accessibility scan"
        category: "Quality Assurance"
        required: true
      - text: "Test with screen reader"
        category: "Quality Assurance"
        required: true
      - text: "Verify keyboard navigation"
        category: "Quality Assurance"
        required: false
```

---

### Example 3: Reorder Workflow

**User Request:** "I want to do testing before deployment"

**Visual Builder:**
- Drag "Quality Gates" dashboard
- Drop before "Deployment Orchestrator"
- System updates sequence automatically

---

## Open Questions

1. Should customizations be shareable between users? (Suggest: Yes, as templates)
2. Maximum customization file size? (Suggest: 1 MB)
3. Should AI suggest customizations proactively? (Suggest: Yes, based on usage patterns)

---

**End of CUSTOMIZATION_PATHWAYS.md**
