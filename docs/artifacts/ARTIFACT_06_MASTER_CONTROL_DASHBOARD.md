# Master Control Dashboard Specification

**Document Type:** Core Architecture  
**Version:** 1.1.0  
**Date:** 2026-01-01T00:00:00Z  
**Dependencies:** THE_BIN_SPECIFICATION.md, TERMINOLOGY_GLOSSARY.md, METADATA_GOVERNANCE_DASHBOARD.md

---

## Overview

The **Master Control Dashboard** is the **orchestration layer** for DevGuide Cockpit, providing:
- Drag-and-drop dashboard selection
- Version management interface
- High-impact change visualization
- Accept/reject propagation workflow
- The Bin integration (always visible)
- **Metadata Governance monitoring (NEW v1.1.0)**
- **AI Cortex status overview (NEW v1.1.0)**

**Key Principle:** Master Control is NOT just another dashboard - it's the meta-layer that manages all other dashboards.

---

## UI Layout

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Master Control Dashboard                            [Token: 85K/100K]      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Active Dashboards (Drag to Reorder)                                  │  │
│  │                                                                       │  │
│  │  [1] Project Inception      ┊  [2] Requirements                       │  │
│  │  [3] Architecture Design    ┊  [4] Implementation                     │  │
│  │  [5] Testing & QA          ┊  [6] Deployment                         │  │
│  │                                                                       │  │
│  │  [+ Add Dashboard from Template]                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────┐  ┌──────────────────────────────────────────┐ │
│  │  AI Cortex Panel         │  │  Metadata Governance Panel               │ │
│  │                          │  │                                          │ │
│  │  Context Window:         │  │  Compliance Rate:                        │ │
│  │  ██████████░░░ 67%       │  │  ████████████░ 98.2%                      │ │
│  │                          │  │                                          │ │
│  │  Token Budget:           │  │  Open Violations: 3                      │ │
│  │  45K / 100K remaining    │  │  • 2 WARNING (naming)                    │ │
│  │                          │  │  • 1 INFO (timestamp)                    │ │
│  │  Provider Status:        │  │                                          │ │
│  │  ✓ Claude-3.5 Online     │  │  Clock Drift: +2ms ✓                     │ │
│  │  ✓ GPT-4 Online          │  │  Last Audit: 2026-01-01T12:00:00Z        │ │
│  │                          │  │                                          │ │
│  │  [Manage AI Settings]    │  │  [View Metadata Dashboard]               │ │
│  └──────────────────────────┘  └──────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  The Bin                                                [3 Pending]   │  │
│  │                                                                       │  │
│  │  ⚠ High Impact: Update API spec (affects 4 dashboards)               │  │
│  │    Estimated: 8.5K tokens, 32 file writes                            │  │
│  │    Metadata: ✓ Compliant                                             │  │
│  │    [View Details] [Accept] [Reject]                                  │  │
│  │                                                                       │  │
│  │  ℹ Medium Impact: Add test coverage guardrail                        │  │
│  │    Metadata: ⚠ 1 naming warning                                      │  │
│  │    [View Details] [Accept] [Reject]                                  │  │
│  │                                                                       │  │
│  │  ✓ Low Impact: Update README formatting                              │  │
│  │    Metadata: ✓ Compliant                                             │  │
│  │    [View Details] [Accept] [Reject]                                  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Version History                                                      │  │
│  │                                                                       │  │
│  │  v1.3.2 (Current)  2026-01-01T12:34:00Z                              │  │
│  │  v1.3.1            2025-12-28T09:15:00Z  [Rollback]                  │  │
│  │  v1.3.0            2025-12-22T14:22:00Z  [View Changes]              │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Features

### 1. Dashboard Selection (Drag-and-Drop)

**Elm Implementation:**

```elm
type Msg
    = DragStart DashboardId
    | DragOver DashboardId
    | Drop DashboardId
    | AddDashboardFromTemplate

type alias Model =
    { dashboards : List Dashboard
    , draggingId : Maybe DashboardId
    , dropTarget : Maybe DashboardId
    }

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        DragStart id ->
            ( { model | draggingId = Just id }, Cmd.none )
        
        DragOver id ->
            ( { model | dropTarget = Just id }, Cmd.none )
        
        Drop targetId ->
            case model.draggingId of
                Just draggedId ->
                    let
                        reordered = reorderDashboards draggedId targetId model.dashboards
                    in
                    ( { model 
                        | dashboards = reordered
                        , draggingId = Nothing
                        , dropTarget = Nothing 
                      }
                    , saveDashboardOrder reordered
                    )
                
                Nothing ->
                    ( model, Cmd.none )
```

**Visual Feedback:**
- Dragged dashboard shows ghost image
- Drop target highlighted with blue border
- Smooth animation on reorder

---

### 2. Impact Assessment Workflow

**Change Classification:**

| Impact Level | Criteria | Visual Indicator |
|--------------|----------|------------------|
| **Critical** | Affects 5+ dashboards OR requires >20K tokens | 🔴 Red badge |
| **High** | Affects 3-4 dashboards OR requires 10K-20K tokens | 🟠 Orange badge |
| **Medium** | Affects 1-2 dashboards OR requires 5K-10K tokens | 🟡 Yellow badge |
| **Low** | Single dashboard, <5K tokens | 🟢 Green badge |

**Detail View:**

```elm
viewImpactDetails : PendingChange -> Html Msg
viewImpactDetails change =
    div [ class "impact-details" ]
        [ h3 [] [ text "Impact Analysis" ]
        , div [ class "metric" ]
            [ span [ class "label" ] [ text "Affected Dashboards:" ]
            , span [ class "value" ] [ text (String.fromInt change.affectedCount) ]
            , ul [] (List.map viewAffectedDashboard change.affectedDashboards)
            ]
        , div [ class "metric" ]
            [ span [ class "label" ] [ text "Estimated Token Cost:" ]
            , span [ class "value" ] [ text (formatTokens change.tokenCost) ]
            ]
        , div [ class "metric" ]
            [ span [ class "label" ] [ text "File Modifications:" ]
            , span [ class "value" ] [ text (String.fromInt change.fileWrites) ]
            ]
        , div [ class "guardrail-results" ]
            [ h4 [] [ text "Guardrail Checks" ]
            , ul [] (List.map viewGuardrailResult change.validationResults)
            ]
        , viewMetadataComplianceStatus change.metadataStatus
        , div [ class "actions" ]
            [ button [ onClick (AcceptChange change.id) ] [ text "Accept & Propagate" ]
            , button [ onClick (RejectChange change.id) ] [ text "Reject" ]
            ]
        ]
```

---

### 3. Accept/Reject Workflow

**Accept Path:**

```
1. User clicks "Accept & Propagate"
2. Confirm dialog: "This will modify 4 dashboards and use 8.5K tokens. Continue?"
3. User confirms
4. Progress indicator shown
5. Go API processes propagation:
   a. Mark change as VALIDATED in sync_buffer
   b. Execute propagation plan
   c. Update affected dashboards
   d. Commit to Git (if enabled)
   e. Send SSE notification
6. Success message: "Change propagated successfully"
7. Bin UI updates: move to "Completed" section
```

**Reject Path:**

```
1. User clicks "Reject"
2. Optional: "Reason for rejection?" (text input)
3. Change marked as REJECTED
4. Removed from Bin UI
5. Audit log entry created
```

**Elm Code:**

```elm
type Msg
    = AcceptChange ChangeId
    | RejectChange ChangeId
    | ConfirmPropagation ChangeId
    | CancelPropagation

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        AcceptChange changeId ->
            ( { model | confirmingChangeId = Just changeId }, Cmd.none )
        
        ConfirmPropagation changeId ->
            ( { model | confirmingChangeId = Nothing, propagating = Just changeId }
            , Http.post
                { url = "/api/bin/accept/" ++ changeId
                , body = Http.emptyBody
                , expect = Http.expectJson PropagationCompleted propagationDecoder
                }
            )
        
        RejectChange changeId ->
            ( model
            , Http.post
                { url = "/api/bin/reject/" ++ changeId
                , body = Http.emptyBody
                , expect = Http.expectWhatever ChangeRejected
                }
            )
```

---

### 4. Version Management

**Version Schema:**
- Semantic versioning (MAJOR.MINOR.PATCH)
- Auto-increment PATCH on every propagation
- Manual MINOR/MAJOR bumps via UI

**Version History Display:**

```elm
viewVersionHistory : List Version -> Html Msg
viewVersionHistory versions =
    div [ class "version-history" ]
        [ h3 [] [ text "Version History" ]
        , ul [] (List.map viewVersion versions)
        ]

viewVersion : Version -> Html msg
viewVersion version =
    li [ class (if version.current then "current" else "") ]
        [ span [ class "version-number" ] [ text version.number ]
        , span [ class "timestamp" ] [ text (formatTimestamp version.createdAt) ]
        , if version.current then
            text "(Current)"
          else
            button [ onClick (RollbackToVersion version.id) ] [ text "Rollback" ]
        ]
```

**Rollback Mechanism:**

```rust
pub async fn rollback_to_version(
    project_id: &str, 
    version_id: &str
) -> Result<(), RollbackError> {
    // Retrieve version snapshot
    let snapshot = db.get_version_snapshot(version_id).await?;
    
    // Restore all dashboards to snapshot state
    for dashboard_config in snapshot.dashboards {
        db.update_dashboard(dashboard_config).await?;
    }
    
    // Restore modules
    for module_config in snapshot.modules {
        db.update_module(module_config).await?;
    }
    
    // Git revert if needed
    if let Some(commit_sha) = snapshot.git_commit {
        git_revert_to(commit_sha).await?;
    }
    
    // Create audit log entry
    audit_log("version_rollback", project_id, version_id).await?;
    
    Ok(())
}
```

**Rollback Window:** 30 days (configurable)

---

### 5. The Bin Integration

**Position:** Fixed bottom panel (always visible)  
**Collapse/Expand:** User can toggle via keyboard shortcut (Ctrl+B)

**Badge Notification:**

```elm
viewBinBadge : Int -> Html Msg
viewBinBadge pendingCount =
    if pendingCount > 0 then
        div [ class "bin-badge", onClick ToggleBin ]
            [ text ("The Bin (" ++ String.fromInt pendingCount ++ ")")
            , span [ class "pulse-animation" ] []
            ]
    else
        div [ class "bin-badge-empty" ] [ text "The Bin (0)" ]
```

**Real-Time Updates (SSE):**

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch
        [ binUpdates BinChangeReceived  -- SSE subscription
        , Time.every 30000 RefreshDashboards  -- Fallback polling
        , metadataGovernanceUpdates MetadataStatusReceived  -- NEW v1.1.0
        ]
```

---

### 6. Template Selection

**Add Dashboard Workflow:**

```
1. User clicks "+ Add Dashboard from Template"
2. Modal shows template catalog:
   - Vibe Coding (6 dashboards)
   - Data Science (7 dashboards)
   - Note Taking (4 dashboards)
   - Community Templates (sorted by downloads)
3. User selects template
4. AI-assisted configuration wizard:
   a. "What's the project name?"
   b. "Which guardrails to enable?"
   c. "Connect to GitHub repository?"
5. Dashboard created and added to profile
6. Opens in new window
```

**Elm Modal:**

```elm
viewTemplateSelector : Model -> Html Msg
viewTemplateSelector model =
    div [ class "modal-overlay", onClick CloseModal ]
        [ div [ class "modal-content", stopPropagationOn "click" (Decode.succeed (NoOp, True)) ]
            [ h2 [] [ text "Select Dashboard Template" ]
            , div [ class "template-grid" ]
                (List.map viewTemplateCard model.availableTemplates)
            ]
        ]

viewTemplateCard : Template -> Html Msg
viewTemplateCard template =
    div [ class "template-card", onClick (SelectTemplate template.id) ]
        [ h3 [] [ text template.name ]
        , p [] [ text template.description ]
        , span [ class "download-count" ] 
            [ text (String.fromInt template.downloads ++ " downloads") ]
        ]
```

---

## Metadata Governance Panel (NEW v1.1.0)

### 7. Metadata Governance Status

The Metadata Governance Panel provides real-time monitoring of compliance with system-wide metadata standards defined in `ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md`.

**Data Types:**

```elm
-- Metadata Governance Status Type
type alias MetadataGovernanceStatus =
    { complianceRate : Float          -- 0.0 to 100.0 (percentage)
    , openViolations : Int            -- Count of unresolved violations
    , errorCount : Int                -- ERROR-level violations (blocking)
    , warningCount : Int              -- WARNING-level violations (non-blocking)
    , infoCount : Int                 -- INFO-level violations (advisory)
    , clockDriftMs : Int              -- Current clock drift in milliseconds
    , clockSyncStatus : ClockStatus   -- synchronized | drifting | error
    , lastAuditTime : String          -- ISO 8601 with Z suffix
    }

type ClockStatus
    = Synchronized
    | Drifting
    | ClockError

-- Individual violation for display
type alias MetadataViolation =
    { id : String
    , standardId : String
    , standardName : String
    , resourceType : String
    , resourceId : String
    , violationValue : String
    , suggestedFix : String
    , severity : ViolationSeverity
    , detectedAt : String             -- ISO 8601 with Z suffix
    }

type ViolationSeverity
    = Error
    | Warning
    | Info
```

**Panel View Component:**

```elm
metadataGovernancePanel : MetadataGovernanceStatus -> Html Msg
metadataGovernancePanel status =
    div [ class "metadata-governance-panel" ]
        [ div [ class "panel-header" ]
            [ h3 [] [ text "Metadata Governance" ]
            , clockSyncIndicator status.clockSyncStatus status.clockDriftMs
            ]
        , div [ class "panel-body" ]
            [ complianceGauge status.complianceRate
            , violationsSummary status.openViolations status.errorCount status.warningCount status.infoCount
            , lastAuditTimestamp status.lastAuditTime
            ]
        , div [ class "panel-footer" ]
            [ button 
                [ class "secondary-button"
                , onClick ViewMetadataDashboard 
                ] 
                [ text "View Dashboard" ]
            ]
        ]

-- Compliance rate gauge (visual progress bar)
complianceGauge : Float -> Html Msg
complianceGauge rate =
    let
        colorClass =
            if rate >= 95.0 then "gauge-success"
            else if rate >= 80.0 then "gauge-warning"
            else "gauge-danger"
    in
    div [ class "compliance-gauge" ]
        [ div [ class "gauge-label" ] [ text "Compliance Rate:" ]
        , div [ class ("gauge-bar " ++ colorClass) ]
            [ div 
                [ class "gauge-fill"
                , style "width" (String.fromFloat rate ++ "%")
                ] 
                []
            ]
        , div [ class "gauge-value" ] [ text (String.fromFloat rate ++ "%") ]
        ]

-- Violations summary with severity breakdown
violationsSummary : Int -> Int -> Int -> Int -> Html Msg
violationsSummary total errors warnings infos =
    div [ class "violations-summary" ]
        [ div [ class "violations-total" ]
            [ text ("Open Violations: " ++ String.fromInt total) ]
        , if total > 0 then
            ul [ class "violations-breakdown" ]
                [ if errors > 0 then
                    li [ class "violation-error" ] 
                        [ text (String.fromInt errors ++ " ERROR (blocking)") ]
                  else text ""
                , if warnings > 0 then
                    li [ class "violation-warning" ] 
                        [ text (String.fromInt warnings ++ " WARNING") ]
                  else text ""
                , if infos > 0 then
                    li [ class "violation-info" ] 
                        [ text (String.fromInt infos ++ " INFO") ]
                  else text ""
                ]
          else
            div [ class "all-compliant" ] [ text "✓ All standards met" ]
        ]

-- Clock drift indicator with status icon
clockSyncIndicator : ClockStatus -> Int -> Html Msg
clockSyncIndicator status driftMs =
    let
        (iconClass, statusText) =
            case status of
                Synchronized ->
                    ("clock-sync", "✓ " ++ String.fromInt driftMs ++ "ms")
                Drifting ->
                    ("clock-drift", "⚠ " ++ String.fromInt driftMs ++ "ms drift")
                ClockError ->
                    ("clock-error", "✗ Sync error")
    in
    div [ class ("clock-indicator " ++ iconClass) ]
        [ span [ class "clock-icon" ] [ text "🕐" ]
        , span [ class "clock-status" ] [ text statusText ]
        ]

-- Last audit timestamp (always in Zulu format)
lastAuditTimestamp : String -> Html Msg
lastAuditTimestamp timestamp =
    div [ class "last-audit" ]
        [ span [ class "label" ] [ text "Last Audit: " ]
        , span [ class "timestamp" ] [ text timestamp ]
        ]
```

**Navigation Message:**

```elm
type Msg
    = -- ... existing messages ...
    | ViewMetadataDashboard
    | MetadataStatusReceived MetadataGovernanceStatus
    | RefreshMetadataStatus

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        ViewMetadataDashboard ->
            ( model
            , Navigation.pushUrl model.key "/dashboards/metadata-governance"
            )
        
        MetadataStatusReceived status ->
            ( { model | metadataGovernance = status }
            , Cmd.none
            )
        
        RefreshMetadataStatus ->
            ( model
            , fetchMetadataGovernanceStatus
            )
```

---

### 8. Metadata Governance Data Fetching

**API Endpoint Integration:**

```elm
-- Fetch metadata governance status from Go API
fetchMetadataGovernanceStatus : Cmd Msg
fetchMetadataGovernanceStatus =
    Http.get
        { url = "/api/metadata/governance/status"
        , expect = Http.expectJson MetadataStatusReceived metadataStatusDecoder
        }

-- JSON decoder for metadata status
metadataStatusDecoder : Decoder MetadataGovernanceStatus
metadataStatusDecoder =
    Decode.map8 MetadataGovernanceStatus
        (Decode.field "compliance_rate" Decode.float)
        (Decode.field "open_violations" Decode.int)
        (Decode.field "error_count" Decode.int)
        (Decode.field "warning_count" Decode.int)
        (Decode.field "info_count" Decode.int)
        (Decode.field "clock_drift_ms" Decode.int)
        (Decode.field "clock_sync_status" clockStatusDecoder)
        (Decode.field "last_audit_time" Decode.string)

clockStatusDecoder : Decoder ClockStatus
clockStatusDecoder =
    Decode.string
        |> Decode.andThen
            (\str ->
                case str of
                    "synchronized" -> Decode.succeed Synchronized
                    "drifting" -> Decode.succeed Drifting
                    "error" -> Decode.succeed ClockError
                    _ -> Decode.fail ("Unknown clock status: " ++ str)
            )

-- Fetch open violations with details
fetchOpenViolations : Cmd Msg
fetchOpenViolations =
    Http.get
        { url = "/api/metadata/violations?status=open&limit=10"
        , expect = Http.expectJson ViolationsReceived (Decode.list violationDecoder)
        }

violationDecoder : Decoder MetadataViolation
violationDecoder =
    Decode.map9 MetadataViolation
        (Decode.field "id" Decode.string)
        (Decode.field "standard_id" Decode.string)
        (Decode.field "standard_name" Decode.string)
        (Decode.field "resource_type" Decode.string)
        (Decode.field "resource_id" Decode.string)
        (Decode.field "violation_value" Decode.string)
        (Decode.field "suggested_fix" Decode.string)
        (Decode.field "severity" severityDecoder)
        (Decode.field "detected_at" Decode.string)

severityDecoder : Decoder ViolationSeverity
severityDecoder =
    Decode.string
        |> Decode.andThen
            (\str ->
                case str of
                    "error" -> Decode.succeed Error
                    "warning" -> Decode.succeed Warning
                    "info" -> Decode.succeed Info
                    _ -> Decode.fail ("Unknown severity: " ++ str)
            )
```

**Go API Handler (Reference):**

```go
// Handler: GET /api/metadata/governance/status
func (h *Handler) GetMetadataGovernanceStatus(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Query metadata_violations table for open violations
    var openViolations []struct {
        Severity string
        Count    int
    }
    err := h.db.QueryContext(ctx, `
        SELECT 
            ms.enforcement_level as severity,
            COUNT(*) as count
        FROM metadata_violations mv
        JOIN metadata_standards ms ON mv.standard_id = ms.id
        WHERE mv.status = 'open'
        GROUP BY ms.enforcement_level
    `).Scan(&openViolations)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    // Calculate compliance rate
    var totalResources, compliantResources int
    h.db.QueryRowContext(ctx, `
        SELECT 
            COUNT(DISTINCT resource_id) as total,
            COUNT(DISTINCT CASE WHEN status != 'open' THEN resource_id END) as compliant
        FROM metadata_violations
    `).Scan(&totalResources, &compliantResources)
    
    complianceRate := 100.0
    if totalResources > 0 {
        complianceRate = float64(compliantResources) / float64(totalResources) * 100
    }
    
    // Query clock_sync_status table
    var clockStatus struct {
        Status         string
        OffsetMicros   int64
        LastSyncAt     time.Time
    }
    h.db.QueryRowContext(ctx, `
        SELECT status, offset_microseconds, last_sync_at
        FROM clock_sync_status
        WHERE component_name = 'api_server'
        ORDER BY last_sync_at DESC
        LIMIT 1
    `).Scan(&clockStatus.Status, &clockStatus.OffsetMicros, &clockStatus.LastSyncAt)
    
    response := map[string]interface{}{
        "compliance_rate":   complianceRate,
        "open_violations":   sumViolations(openViolations),
        "error_count":       getCountBySeverity(openViolations, "error"),
        "warning_count":     getCountBySeverity(openViolations, "warning"),
        "info_count":        getCountBySeverity(openViolations, "info"),
        "clock_drift_ms":    clockStatus.OffsetMicros / 1000,
        "clock_sync_status": clockStatus.Status,
        "last_audit_time":   time.Now().UTC().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

---

## AI Cortex Panel (NEW v1.1.0)

### 9. AI Cortex Status

The AI Cortex Panel provides visibility into AI provider status and token usage.

**Data Types:**

```elm
type alias AICortexStatus =
    { contextUsagePercent : Float     -- 0.0 to 100.0
    , tokensUsed : Int                -- Current session tokens
    , tokenBudget : Int               -- Maximum allowed tokens
    , activeProvider : String         -- "claude-3.5" | "gpt-4" | etc.
    , providerStatuses : List ProviderStatus
    }

type alias ProviderStatus =
    { name : String
    , status : ProviderOnlineStatus
    , latencyMs : Int
    }

type ProviderOnlineStatus
    = Online
    | Degraded
    | Offline
```

**Panel Component:**

```elm
aiCortexPanel : AICortexStatus -> Html Msg
aiCortexPanel status =
    div [ class "ai-cortex-panel" ]
        [ div [ class "panel-header" ]
            [ h3 [] [ text "AI Cortex" ]
            ]
        , div [ class "panel-body" ]
            [ contextWindowGauge status.contextUsagePercent
            , tokenBudgetDisplay status.tokensUsed status.tokenBudget
            , providerStatusList status.providerStatuses
            ]
        , div [ class "panel-footer" ]
            [ button 
                [ class "secondary-button"
                , onClick ManageAISettings 
                ] 
                [ text "Manage AI Settings" ]
            ]
        ]

contextWindowGauge : Float -> Html Msg
contextWindowGauge usagePercent =
    div [ class "context-gauge" ]
        [ div [ class "gauge-label" ] [ text "Context Window:" ]
        , div [ class "gauge-bar" ]
            [ div 
                [ class "gauge-fill"
                , style "width" (String.fromFloat usagePercent ++ "%")
                ] 
                []
            ]
        , div [ class "gauge-value" ] [ text (String.fromFloat usagePercent ++ "%") ]
        ]

tokenBudgetDisplay : Int -> Int -> Html Msg
tokenBudgetDisplay used budget =
    div [ class "token-budget" ]
        [ div [ class "budget-label" ] [ text "Token Budget:" ]
        , div [ class "budget-value" ] 
            [ text (formatTokens used ++ " / " ++ formatTokens budget ++ " remaining") ]
        ]

providerStatusList : List ProviderStatus -> Html Msg
providerStatusList providers =
    div [ class "provider-statuses" ]
        [ div [ class "status-label" ] [ text "Provider Status:" ]
        , ul [ class "provider-list" ]
            (List.map viewProviderStatus providers)
        ]

viewProviderStatus : ProviderStatus -> Html Msg
viewProviderStatus provider =
    let
        statusIcon =
            case provider.status of
                Online -> "✓"
                Degraded -> "⚠"
                Offline -> "✗"
    in
    li [ class ("provider-item " ++ statusClass provider.status) ]
        [ span [ class "status-icon" ] [ text statusIcon ]
        , span [ class "provider-name" ] [ text provider.name ]
        ]

statusClass : ProviderOnlineStatus -> String
statusClass status =
    case status of
        Online -> "status-online"
        Degraded -> "status-degraded"
        Offline -> "status-offline"
```

---

## Token Budget Display

**Position:** Top-right corner  
**Format:** Progress bar + remaining count

```elm
viewTokenBudget : TokenBudget -> Html Msg
viewTokenBudget budget =
    div [ class "token-budget-widget" ]
        [ div [ class "label" ] [ text "Session Token Budget" ]
        , div [ class "progress-bar" ]
            [ div 
                [ class "progress-fill"
                , style "width" (String.fromFloat (toFloat budget.used / toFloat budget.limit * 100) ++ "%")
                ] 
                []
            ]
        , div [ class "count" ] 
            [ text (formatTokens (budget.limit - budget.used) ++ " remaining") ]
        ]
```

**Warning Threshold:** Alert when 90% consumed

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl+B | Toggle The Bin |
| Ctrl+T | Open Template Selector |
| Ctrl+V | Open Version History |
| Ctrl+M | Toggle Metadata Governance Panel (NEW v1.1.0) |
| Ctrl+1-9 | Switch to dashboard 1-9 |
| Ctrl+Enter | Accept selected change |
| Ctrl+Delete | Reject selected change |

---

## SSE Update Flow

**Server (Go API):**

```go
func (h *Handler) StreamBinUpdates(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    updates := make(chan BinUpdate)
    h.binService.Subscribe(r.Context(), updates)
    
    for {
        select {
        case update := <-updates:
            data, _ := json.Marshal(update)
            fmt.Fprintf(w, "data: %s\n\n", data)
            w.(http.Flusher).Flush()
        
        case <-r.Context().Done():
            return
        }
    }
}

// NEW v1.1.0: Stream metadata governance updates
func (h *Handler) StreamMetadataUpdates(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    updates := make(chan MetadataGovernanceUpdate)
    h.metadataService.Subscribe(r.Context(), updates)
    
    for {
        select {
        case update := <-updates:
            data, _ := json.Marshal(update)
            fmt.Fprintf(w, "event: metadata\ndata: %s\n\n", data)
            w.(http.Flusher).Flush()
        
        case <-r.Context().Done():
            return
        }
    }
}
```

**Client (Elm):**

```elm
port binUpdates : (String -> msg) -> Sub msg
port metadataGovernanceUpdates : (String -> msg) -> Sub msg

-- In JavaScript (ports.js)
app.ports.binUpdates.send = function(callback) {
    const eventSource = new EventSource('/api/bin/stream');
    eventSource.onmessage = function(event) {
        callback(event.data);
    };
};

// NEW v1.1.0: Metadata governance SSE subscription
app.ports.metadataGovernanceUpdates.send = function(callback) {
    const eventSource = new EventSource('/api/metadata/stream');
    eventSource.addEventListener('metadata', function(event) {
        callback(event.data);
    });
};
```

---

## Database Dependencies (v1.1.0)

The Master Control Dashboard queries the following tables:

**Core Tables (existing):**
- `projects` - Project metadata
- `profiles` - User profiles
- `dashboards` - Dashboard configurations
- `modules` - Module configurations
- `sync_buffer` - The Bin pending changes
- `propagation_log` - Change propagation history
- `version_history` - Version snapshots

**Metadata Governance Tables (NEW v1.1.0):**
- `metadata_standards` - Active validation rules
- `metadata_violations` - Compliance issues
- `clock_sync_status` - NTP/PTP monitoring

**AI Cortex Tables (NEW v1.1.0):**
- `token_usage` - Token consumption tracking
- `api_configurations` - Provider configurations

---

## Cross-References

- **ARTIFACT_03_THE_BIN_SPECIFICATION.md** - The Bin validation pipeline
- **ARTIFACT_05_DATA_MODEL.md** v1.1.0 - Database schema definitions
- **ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md** - Full metadata governance specification
- **AI_CORTEX_MANAGEMENT_DASHBOARD.md** - AI provider management details

---

## Changelog

### v1.1.0 (2026-01-01T00:00:00Z)
- Added Metadata Governance Panel with compliance monitoring
- Added AI Cortex Panel with token usage tracking
- Updated UI layout to 3-column panel design
- Added SSE subscription for metadata updates
- Added keyboard shortcut Ctrl+M for metadata panel toggle
- Added data fetching for `metadata_violations` and `clock_sync_status` tables
- Updated The Bin items to show metadata compliance status
- All timestamps now use ISO 8601 with Z suffix (Zulu-First compliance)

### v1.0.0 (2026-01-01)
- Initial specification
- Dashboard selection with drag-and-drop
- Impact assessment workflow
- Accept/reject propagation
- Version management
- The Bin integration
- Template selection

---

**End of MASTER_CONTROL_DASHBOARD.md v1.1.0**
