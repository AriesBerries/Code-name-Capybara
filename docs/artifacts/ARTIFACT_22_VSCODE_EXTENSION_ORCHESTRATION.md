# VS Code Extension Orchestration System

**Document Type:** Core Architecture  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** ARTIFACT_06_MASTER_CONTROL_DASHBOARD.md, ARTIFACT_21_AI_CORTEX_MANAGEMENT_DASHBOARD.md, TiDB  
**Priority:** Tier 1 - Platform Differentiator

---

## Executive Summary

The **VS Code Extension Orchestration System** elevates VS Code extensions to **first-class customizable components** within DevGuide Cockpit. Instead of treating extensions as black boxes, users can:

1. **Intercept and modify** inputs/outputs to any extension
2. **Wrap extensions** in custom dashboards with specialized workflows
3. **Share dashboard templates** via community repository (like Claude MCP servers)
4. **Use different dashboards** for the same extension across different projects
5. **Create just-in-time** information delivery pipelines to extensions

**Philosophy:** Extensions are **data transformation pipelines** that can be observed, controlled, and customized through **dashboard-driven workflows**.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  DevGuide Cockpit (Elm Frontend)                            │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Extension Dashboard Selector                         │   │
│  │  • Choose from community dashboard templates          │   │
│  │  • Preview before installing                          │   │
│  │  • Per-project customization                          │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Active Extension Dashboard (e.g., "Copilot Pro")     │   │
│  │                                                         │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │   │
│  │  │ Input    │  │ Transform│  │ Output   │            │   │
│  │  │ Controls │→ │ Pipeline │→ │ Preview  │            │   │
│  │  └──────────┘  └──────────┘  └──────────┘            │   │
│  │                                                         │   │
│  │  [Start Extension] [Stop] [Logs] [Analytics]           │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP/WebSocket
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Extension Proxy Layer (Go Service)                         │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Command Interceptor                                  │   │
│  │  • Capture all vscode.commands.execute() calls        │   │
│  │  • Modify arguments before execution                  │   │
│  │  • Transform results after execution                  │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Event Interceptor                                    │   │
│  │  • Listen to workspace events                         │   │
│  │  • Filter/transform event data                        │   │
│  │  • Inject additional context                          │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  I/O Transformer                                      │   │
│  │  • Modify input before sending to extension           │   │
│  │  • Transform output before returning to user          │   │
│  │  • Apply user-defined transformation rules            │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  VS Code Extension Host (Native)                            │
│  • Extension runs normally                                   │
│  • Believes it's talking directly to VS Code                │
│  • All I/O intercepted transparently                         │
└─────────────────────────────────────────────────────────────┘
```

---

## Component 1: Extension Proxy Layer

**Purpose:** Transparent interception of all extension I/O

### Implementation (Go Service)

```go
// extension_proxy.go
package extensionproxy

import (
    "context"
    "encoding/json"
    "sync"
)

// ExtensionProxy intercepts all extension communication
type ExtensionProxy struct {
    extensionID      string
    config           *ProxyConfig
    commandHooks     map[string]CommandHook
    eventHooks       map[string]EventHook
    transformPipeline *TransformPipeline
    analytics        *AnalyticsCollector
    mu               sync.RWMutex
}

type ProxyConfig struct {
    ExtensionID        string
    EnableCommandLog   bool
    EnableEventLog     bool
    InputTransforms    []TransformRule
    OutputTransforms   []TransformRule
    JITDataSources     []DataSource
    MaxLogSize         int64
}

type CommandHook struct {
    CommandID string
    PreHook   func(ctx context.Context, args []interface{}) ([]interface{}, error)
    PostHook  func(ctx context.Context, result interface{}) (interface{}, error)
}

type EventHook struct {
    EventName string
    Filter    func(ctx context.Context, event interface{}) bool
    Transform func(ctx context.Context, event interface{}) (interface{}, error)
}

type TransformRule struct {
    Name      string
    Type      TransformType  // input, output, bidirectional
    Pattern   string         // regex or JSONPath
    Transform string         // JavaScript or Go function
    Enabled   bool
}

type DataSource struct {
    Name     string
    Type     SourceType  // database, api, file, ai
    Query    string
    Cache    time.Duration
    Priority int
}

// Intercept command execution
func (ep *ExtensionProxy) InterceptCommand(ctx context.Context, commandID string, args []interface{}) (interface{}, error) {
    ep.mu.RLock()
    hook, exists := ep.commandHooks[commandID]
    ep.mu.RUnlock()
    
    // Log command if enabled
    if ep.config.EnableCommandLog {
        ep.analytics.LogCommand(CommandLog{
            ExtensionID: ep.extensionID,
            CommandID:   commandID,
            Args:        args,
            Timestamp:   time.Now(),
        })
    }
    
    // Apply pre-hook transformation
    if exists && hook.PreHook != nil {
        modifiedArgs, err := hook.PreHook(ctx, args)
        if err != nil {
            return nil, err
        }
        args = modifiedArgs
    }
    
    // Apply input transforms from user config
    args, err := ep.transformPipeline.TransformInput(args)
    if err != nil {
        return nil, err
    }
    
    // Inject JIT data if configured
    args = ep.injectJITData(ctx, commandID, args)
    
    // Execute actual command in VS Code
    result, err := ep.executeNativeCommand(ctx, commandID, args)
    if err != nil {
        return nil, err
    }
    
    // Apply post-hook transformation
    if exists && hook.PostHook != nil {
        result, err = hook.PostHook(ctx, result)
        if err != nil {
            return nil, err
        }
    }
    
    // Apply output transforms from user config
    result, err = ep.transformPipeline.TransformOutput(result)
    if err != nil {
        return nil, err
    }
    
    // Log result
    ep.analytics.LogCommandResult(commandID, result)
    
    return result, nil
}

// Inject just-in-time data from configured sources
func (ep *ExtensionProxy) injectJITData(ctx context.Context, commandID string, args []interface{}) []interface{} {
    for _, source := range ep.config.JITDataSources {
        data, err := ep.fetchDataSource(ctx, source)
        if err != nil {
            log.Warnf("Failed to fetch JIT data from %s: %v", source.Name, err)
            continue
        }
        
        // Append to args or merge based on source config
        args = append(args, data)
    }
    return args
}

// Fetch data from configured source
func (ep *ExtensionProxy) fetchDataSource(ctx context.Context, source DataSource) (interface{}, error) {
    // Check cache first
    if cached := ep.getCachedData(source.Name); cached != nil {
        return cached, nil
    }
    
    var data interface{}
    var err error
    
    switch source.Type {
    case SourceTypeDatabase:
        data, err = ep.queryDatabase(ctx, source.Query)
    case SourceTypeAPI:
        data, err = ep.callAPI(ctx, source.Query)
    case SourceTypeFile:
        data, err = ep.readFile(source.Query)
    case SourceTypeAI:
        data, err = ep.askAI(ctx, source.Query)
    }
    
    if err != nil {
        return nil, err
    }
    
    // Cache if configured
    if source.Cache > 0 {
        ep.cacheData(source.Name, data, source.Cache)
    }
    
    return data, nil
}

// Transform pipeline
type TransformPipeline struct {
    inputTransforms  []TransformRule
    outputTransforms []TransformRule
    jsRuntime        *goja.Runtime  // JavaScript engine for custom transforms
}

func (tp *TransformPipeline) TransformInput(data interface{}) (interface{}, error) {
    for _, rule := range tp.inputTransforms {
        if !rule.Enabled {
            continue
        }
        
        // Apply transformation
        data = tp.applyTransform(rule, data)
    }
    return data, nil
}

func (tp *TransformPipeline) applyTransform(rule TransformRule, data interface{}) interface{} {
    // Execute JavaScript transform
    if strings.HasPrefix(rule.Transform, "function") {
        result, err := tp.jsRuntime.RunString(rule.Transform + "; transform(data);")
        if err != nil {
            log.Errorf("Transform %s failed: %v", rule.Name, err)
            return data
        }
        return result.Export()
    }
    
    // Apply regex transformation
    if rule.Pattern != "" {
        // ... regex logic
    }
    
    return data
}
```

---

## Component 2: Extension Dashboard Builder

**Purpose:** Visual interface for creating custom dashboards around extensions

### Dashboard Template Structure

```yaml
# copilot-pro-dashboard.yaml
metadata:
  name: "Copilot Pro Dashboard"
  version: "1.2.0"
  author: "john.doe@example.com"
  description: "Enhanced GitHub Copilot dashboard with context injection"
  extension_id: "GitHub.copilot"
  compatible_versions: ["1.x", "2.x"]
  tags: ["ai", "copilot", "productivity"]
  stars: 4523
  downloads: 15234

layout:
  type: "three-column"
  sections:
    - id: "input-control"
      title: "Input Controls"
      position: "left"
      width: "25%"
      
    - id: "extension-panel"
      title: "Copilot Suggestions"
      position: "center"
      width: "50%"
      
    - id: "analytics"
      title: "Usage Analytics"
      position: "right"
      width: "25%"

components:
  - id: "context-injector"
    type: "input-transformer"
    section: "input-control"
    config:
      transforms:
        - name: "Add project context"
          enabled: true
          source: "database"
          query: "SELECT context FROM project_context WHERE project_id = ?"
          inject_at: "prompt_prefix"
          
        - name: "Include recent files"
          enabled: true
          source: "filesystem"
          pattern: "*.{ts,js,go}"
          limit: 5
          inject_at: "context"

  - id: "suggestion-filter"
    type: "output-transformer"
    section: "extension-panel"
    config:
      filters:
        - name: "Remove banned patterns"
          pattern: "(TODO|FIXME|console\\.log)"
          action: "reject"
          
        - name: "Enforce style guide"
          source: "ai"
          prompt: "Does this code follow our style guide?"
          threshold: 0.8

  - id: "usage-tracker"
    type: "analytics"
    section: "analytics"
    config:
      metrics:
        - name: "Acceptance rate"
          formula: "accepted / total"
          
        - name: "Time saved"
          formula: "suggestions_used * 30s"
          
        - name: "Token usage"
          source: "copilot_api"

proxy_config:
  commands:
    - id: "copilot.suggest"
      pre_hook:
        type: "javascript"
        code: |
          function(args) {
            // Inject project-specific context
            const context = getProjectContext();
            args[0].context = context;
            return args;
          }
      
      post_hook:
        type: "javascript"
        code: |
          function(result) {
            // Filter out low-quality suggestions
            return result.filter(s => s.confidence > 0.7);
          }

  events:
    - id: "copilot.suggestionShown"
      transform:
        type: "log"
        destination: "analytics"

jit_data_sources:
  - name: "project_context"
    type: "database"
    query: "SELECT * FROM project_metadata WHERE id = ?"
    cache: "5m"
    
  - name: "coding_standards"
    type: "file"
    path: ".devguide/coding-standards.md"
    cache: "1h"
    
  - name: "recent_bugs"
    type: "ai"
    prompt: "List recent bugs in this project"
    cache: "15m"

shortcuts:
  - key: "Ctrl+Shift+C"
    action: "inject_full_context"
    
  - key: "Ctrl+Shift+A"
    action: "accept_with_modifications"

customization:
  allow_user_js: true
  allow_custom_transforms: true
  allow_api_calls: true
```

### Dashboard UI (Elm)

```elm
-- ExtensionDashboard.elm
type alias Model =
    { extensionId : String
    , dashboardConfig : DashboardConfig
    , activeTransforms : List TransformRule
    , commandLogs : List CommandLog
    , analytics : AnalyticsData
    , extensionStatus : ExtensionStatus
    }

type alias DashboardConfig =
    { name : String
    , layout : LayoutConfig
    , components : List Component
    , proxyConfig : ProxyConfig
    }

type Msg
    = LoadDashboard String
    | UpdateTransform String Bool
    | ExecuteCommand String (List Json.Value)
    | ReceiveCommandResult (Result Http.Error Json.Value)
    | ToggleExtension
    | ViewLogs
    | ExportAnalytics

view : Model -> Html Msg
view model =
    div [ class "extension-dashboard" ]
        [ header model
        , layoutView model.dashboardConfig.layout model
        , footer model
        ]

layoutView : LayoutConfig -> Model -> Html Msg
layoutView layout model =
    case layout.type_ of
        "three-column" ->
            div [ class "three-column-layout" ]
                [ leftPanel layout.sections model
                , centerPanel layout.sections model
                , rightPanel layout.sections model
                ]
        
        "single-panel" ->
            div [ class "single-panel-layout" ]
                [ mainPanel layout.sections model ]
        
        _ ->
            text "Unknown layout"

leftPanel : List Section -> Model -> Html Msg
leftPanel sections model =
    let
        leftSection = sections
            |> List.filter (\s -> s.position == "left")
            |> List.head
    in
    case leftSection of
        Just section ->
            div [ class "left-panel" ]
                [ h3 [] [ text section.title ]
                , renderComponents section.id model.dashboardConfig.components model
                ]
        Nothing ->
            text ""

renderComponents : String -> List Component -> Model -> Html Msg
renderComponents sectionId components model =
    components
        |> List.filter (\c -> c.section == sectionId)
        |> List.map (renderComponent model)
        |> div [ class "components-container" ]

renderComponent : Model -> Component -> Html Msg
renderComponent model component =
    case component.type_ of
        "input-transformer" ->
            inputTransformerView component model
        
        "output-transformer" ->
            outputTransformerView component model
        
        "analytics" ->
            analyticsView component model
        
        "command-palette" ->
            commandPaletteView component model
        
        _ ->
            text ("Unknown component: " ++ component.type_)

inputTransformerView : Component -> Model -> Html Msg
inputTransformerView component model =
    div [ class "input-transformer" ]
        [ h4 [] [ text "Input Transformations" ]
        , transformRulesList component.config.transforms
        , addTransformButton
        ]

transformRulesList : List TransformRule -> Html Msg
transformRulesList rules =
    ul [ class "transform-rules" ]
        (List.map transformRuleItem rules)

transformRuleItem : TransformRule -> Html Msg
transformRuleItem rule =
    li [ class "transform-rule" ]
        [ div [ class "rule-header" ]
            [ span [ class "rule-name" ] [ text rule.name ]
            , toggle rule.name rule.enabled
            ]
        , div [ class "rule-details" ]
            [ p [] [ text ("Source: " ++ rule.source) ]
            , p [] [ text ("Query: " ++ rule.query) ]
            , button [ onClick (EditTransform rule.name) ] [ text "Edit" ]
            ]
        ]

analyticsView : Component -> Model -> Html Msg
analyticsView component model =
    div [ class "analytics-panel" ]
        [ h4 [] [ text "Extension Analytics" ]
        , metricsGrid model.analytics
        , chartView model.analytics
        , exportButton
        ]

metricsGrid : AnalyticsData -> Html Msg
metricsGrid data =
    div [ class "metrics-grid" ]
        [ metricCard "Commands Executed" (String.fromInt data.commandsExecuted)
        , metricCard "Avg Response Time" (String.fromFloat data.avgResponseTime ++ "ms")
        , metricCard "Success Rate" (String.fromFloat (data.successRate * 100) ++ "%")
        , metricCard "Tokens Used" (String.fromInt data.tokensUsed)
        ]
```

---

## Component 3: Community Dashboard Repository

**Purpose:** Share and discover extension dashboards (like Claude MCP servers)

### Repository Structure

```
community-dashboards/
├── github-copilot/
│   ├── copilot-basic/
│   │   ├── dashboard.yaml
│   │   ├── README.md
│   │   ├── screenshots/
│   │   └── transforms/
│   │       └── context-injector.js
│   ├── copilot-pro/
│   │   └── ...
│   └── copilot-minimal/
│       └── ...
├── prettier/
│   ├── prettier-team-style/
│   └── prettier-auto-fix/
├── eslint/
│   └── eslint-enforcer/
└── registry.json
```

### Registry Format

```json
{
  "dashboards": [
    {
      "id": "copilot-pro",
      "name": "Copilot Pro Dashboard",
      "extension_id": "GitHub.copilot",
      "version": "1.2.0",
      "author": {
        "name": "John Doe",
        "email": "john@example.com",
        "github": "johndoe"
      },
      "description": "Enhanced GitHub Copilot with context injection and analytics",
      "repository": "https://github.com/devguide-community/copilot-pro-dashboard",
      "download_url": "https://cdn.devguide.io/dashboards/copilot-pro-1.2.0.tar.gz",
      "stars": 4523,
      "downloads": 15234,
      "last_updated": "2026-01-01T00:00:00Z",
      "compatible_versions": ["1.x", "2.x"],
      "tags": ["ai", "copilot", "productivity"],
      "screenshots": [
        "https://cdn.devguide.io/dashboards/copilot-pro/screenshot-1.png"
      ],
      "verified": true,
      "featured": true
    }
  ]
}
```

### Dashboard Discovery UI

```elm
-- DashboardMarketplace.elm
type alias Model =
    { searchQuery : String
    , selectedExtension : Maybe String
    , dashboards : List DashboardListing
    , filters : Filters
    , sortBy : SortOption
    }

type Msg
    = SearchDashboards String
    | FilterByExtension String
    | FilterByTag String
    | SortBy SortOption
    | PreviewDashboard DashboardListing
    | InstallDashboard DashboardListing
    | RateDashboard DashboardListing Int

view : Model -> Html Msg
view model =
    div [ class "dashboard-marketplace" ]
        [ searchBar model.searchQuery
        , filtersPanel model.filters
        , dashboardGrid model.dashboards
        ]

dashboardGrid : List DashboardListing -> Html Msg
dashboardGrid dashboards =
    div [ class "dashboard-grid" ]
        (List.map dashboardCard dashboards)

dashboardCard : DashboardListing -> Html Msg
dashboardCard dashboard =
    div [ class "dashboard-card" ]
        [ img [ src (Maybe.withDefault "" (List.head dashboard.screenshots)) ] []
        , h3 [] [ text dashboard.name ]
        , p [ class "author" ] [ text ("by " ++ dashboard.author.name) ]
        , p [ class "description" ] [ text dashboard.description ]
        , div [ class "stats" ]
            [ stat "★" (String.fromInt dashboard.stars)
            , stat "↓" (String.fromInt dashboard.downloads)
            ]
        , div [ class "actions" ]
            [ button [ onClick (PreviewDashboard dashboard) ] [ text "Preview" ]
            , button [ onClick (InstallDashboard dashboard) ] [ text "Install" ]
            ]
        ]
```

---

## Component 4: Per-Project Customization

**Purpose:** Use different dashboard configurations for different projects

### Data Model

```sql
-- TiDB Schema
CREATE TABLE extension_dashboards (
    id UUID PRIMARY KEY,
    extension_id VARCHAR(255) NOT NULL,
    dashboard_id VARCHAR(255) NOT NULL,  -- From community repo
    name VARCHAR(255) NOT NULL,
    config JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_extension (extension_id)
);

CREATE TABLE project_extension_mappings (
    id UUID PRIMARY KEY,
    project_id UUID NOT NULL,
    extension_id VARCHAR(255) NOT NULL,
    dashboard_id UUID NOT NULL,  -- References extension_dashboards.id
    enabled BOOLEAN DEFAULT TRUE,
    custom_config JSONB,  -- Project-specific overrides
    created_at TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (dashboard_id) REFERENCES extension_dashboards(id),
    UNIQUE(project_id, extension_id)  -- One dashboard per extension per project
);

CREATE TABLE dashboard_versions (
    id UUID PRIMARY KEY,
    dashboard_id UUID NOT NULL,
    version VARCHAR(50) NOT NULL,
    config JSONB NOT NULL,
    changelog TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (dashboard_id) REFERENCES extension_dashboards(id)
);
```

### Project-Specific Dashboard Switcher

```elm
-- ProjectExtensionConfig.elm
type alias Model =
    { projectId : String
    , installedExtensions : List Extension
    , availableDashboards : Dict String (List DashboardListing)
    , activeMappings : List ProjectExtensionMapping
    }

type Msg
    = LoadProjectExtensions
    | SwitchDashboard String String  -- extensionId, dashboardId
    | CustomizeConfig String DashboardConfig
    | ToggleExtension String Bool

view : Model -> Html Msg
view model =
    div [ class "project-extension-config" ]
        [ h2 [] [ text ("Extension Dashboards for " ++ model.projectId) ]
        , extensionList model
        ]

extensionList : Model -> Html Msg
extensionList model =
    div [ class "extension-list" ]
        (List.map (extensionItem model) model.installedExtensions)

extensionItem : Model -> Extension -> Html Msg
extensionItem model ext =
    let
        activeMapping = getActiveMapping ext.id model.activeMappings
        availableDashboards = Dict.get ext.id model.availableDashboards
            |> Maybe.withDefault []
    in
    div [ class "extension-item" ]
        [ div [ class "extension-header" ]
            [ img [ src ext.icon ] []
            , h3 [] [ text ext.name ]
            , toggle ext.id (isEnabled activeMapping)
            ]
        , div [ class "dashboard-selector" ]
            [ label [] [ text "Dashboard:" ]
            , select [ onInput (SwitchDashboard ext.id) ]
                (dashboardOptions activeMapping availableDashboards)
            , button [ onClick (CustomizeConfig ext.id) ] [ text "Customize" ]
            ]
        ]

dashboardOptions : Maybe ProjectExtensionMapping -> List DashboardListing -> List (Html Msg)
dashboardOptions activeMapping dashboards =
    let
        activeDashboardId = activeMapping
            |> Maybe.map .dashboardId
            |> Maybe.withDefault ""
    in
    [ option [ value "native" ] [ text "Native Panel (No Dashboard)" ] ] ++
    List.map (dashboardOption activeDashboardId) dashboards

dashboardOption : String -> DashboardListing -> Html Msg
dashboardOption activeDashboardId dashboard =
    option
        [ value dashboard.id
        , selected (dashboard.id == activeDashboardId)
        ]
        [ text dashboard.name ]
```

---

## Component 5: Just-in-Time Data Delivery

**Purpose:** Automatically provide extensions with project-specific context

### JIT Data Provider

```go
// jit_data_provider.go
package jit

type JITDataProvider struct {
    db         *tidb.Client
    fileSystem *FileSystem
    aiClient   *AIClient
    cache      *redis.Client
}

type DataRequest struct {
    ExtensionID string
    CommandID   string
    ProjectID   string
    Context     map[string]interface{}
}

type DataResponse struct {
    Sources map[string]interface{}
    Latency time.Duration
}

// Gather all relevant context for extension command
func (jdp *JITDataProvider) GatherContext(ctx context.Context, req DataRequest) (*DataResponse, error) {
    start := time.Now()
    
    sources := make(map[string]interface{})
    
    // Parallel data fetching
    var wg sync.WaitGroup
    var mu sync.Mutex
    
    // 1. Project metadata from database
    wg.Add(1)
    go func() {
        defer wg.Done()
        if metadata, err := jdp.db.GetProjectMetadata(req.ProjectID); err == nil {
            mu.Lock()
            sources["project_metadata"] = metadata
            mu.Unlock()
        }
    }()
    
    // 2. Coding standards from filesystem
    wg.Add(1)
    go func() {
        defer wg.Done()
        if standards, err := jdp.fileSystem.ReadFile(".devguide/coding-standards.md"); err == nil {
            mu.Lock()
            sources["coding_standards"] = standards
            mu.Unlock()
        }
    }()
    
    // 3. Recent git commits
    wg.Add(1)
    go func() {
        defer wg.Done()
        if commits, err := jdp.fileSystem.GitLog(10); err == nil {
            mu.Lock()
            sources["recent_commits"] = commits
            mu.Unlock()
        }
    }()
    
    // 4. Active issues/bugs
    wg.Add(1)
    go func() {
        defer wg.Done()
        if issues, err := jdp.db.GetActiveIssues(req.ProjectID); err == nil {
            mu.Lock()
            sources["active_issues"] = issues
            mu.Unlock()
        }
    }()
    
    // 5. AI-generated project summary (cached)
    wg.Add(1)
    go func() {
        defer wg.Done()
        cacheKey := fmt.Sprintf("project_summary:%s", req.ProjectID)
        if cached := jdp.cache.Get(ctx, cacheKey).Val(); cached != "" {
            mu.Lock()
            sources["project_summary"] = cached
            mu.Unlock()
        } else {
            if summary, err := jdp.aiClient.SummarizeProject(req.ProjectID); err == nil {
                jdp.cache.Set(ctx, cacheKey, summary, 1*time.Hour)
                mu.Lock()
                sources["project_summary"] = summary
                mu.Unlock()
            }
        }
    }()
    
    wg.Wait()
    
    return &DataResponse{
        Sources: sources,
        Latency: time.Since(start),
    }, nil
}
```

---

## Component 6: Dashboard Creation Wizard

**Purpose:** Guide users through creating custom dashboards

### Wizard Flow

```
Step 1: Select Extension
  → Choose from installed extensions
  → View extension capabilities

Step 2: Choose Base Template
  → Start from scratch
  → Clone existing dashboard
  → Import from community

Step 3: Configure Layout
  → Single panel (native)
  → Two columns
  → Three columns
  → Custom grid

Step 4: Add Components
  → Input transformers
  → Output filters
  → Analytics widgets
  → Custom panels

Step 5: Configure Transforms
  → JavaScript editor
  → Visual rule builder
  → AI-assisted generation

Step 6: Test & Preview
  → Live preview with real extension
  → Test transform rules
  → View analytics

Step 7: Publish (Optional)
  → Save locally
  → Share with team
  → Publish to community
```

### Wizard UI (Elm)

```elm
-- DashboardCreationWizard.elm
type alias Model =
    { currentStep : WizardStep
    , selectedExtension : Maybe Extension
    , baseTemplate : Maybe DashboardTemplate
    , layout : LayoutConfig
    , components : List Component
    , transforms : List TransformRule
    , previewData : Maybe PreviewData
    }

type WizardStep
    = SelectExtension
    | ChooseTemplate
    | ConfigureLayout
    | AddComponents
    | ConfigureTransforms
    | TestPreview
    | Publish

type Msg
    = NextStep
    | PrevStep
    | SelectExtensionMsg Extension
    | ChooseTemplateMsg DashboardTemplate
    | UpdateLayout LayoutConfig
    | AddComponent Component
    | RemoveComponent String
    | UpdateTransform TransformRule
    | RunPreview
    | PublishDashboard PublishConfig

view : Model -> Html Msg
view model =
    div [ class "dashboard-wizard" ]
        [ wizardHeader model.currentStep
        , wizardContent model
        , wizardFooter model
        ]

wizardContent : Model -> Html Msg
wizardContent model =
    case model.currentStep of
        SelectExtension ->
            selectExtensionView model
        
        ChooseTemplate ->
            chooseTemplateView model
        
        ConfigureLayout ->
            configureLayoutView model
        
        AddComponents ->
            addComponentsView model
        
        ConfigureTransforms ->
            configureTransformsView model
        
        TestPreview ->
            testPreviewView model
        
        Publish ->
            publishView model

configureTransformsView : Model -> Html Msg
configureTransformsView model =
    div [ class "transform-config" ]
        [ h3 [] [ text "Configure Transformations" ]
        , transformEditor model.transforms
        , aiAssistButton
        ]

transformEditor : List TransformRule -> Html Msg
transformEditor rules =
    div [ class "transform-editor" ]
        [ div [ class "editor-tabs" ]
            [ button [] [ text "Visual Builder" ]
            , button [] [ text "JavaScript Editor" ]
            , button [] [ text "AI Generate" ]
            ]
        , editorContent rules
        ]
```

---

## Integration with Existing Architecture

### With Master Control Dashboard

```elm
-- Add "Extension Dashboards" card to Master Control
type Msg
    = ...
    | ManageExtensionDashboards
    | ViewExtensionAnalytics String

view : Model -> Html Msg
view model =
    div [ class "master-control" ]
        [ ...
        , extensionDashboardsCard model.extensionMappings
        , ...
        ]

extensionDashboardsCard : List ProjectExtensionMapping -> Html Msg
extensionDashboardsCard mappings =
    div [ class "card extension-dashboards" ]
        [ h3 [] [ text "Active Extension Dashboards" ]
        , ul []
            (List.map extensionDashboardItem mappings)
        , button [ onClick ManageExtensionDashboards ] [ text "Manage" ]
        ]
```

### With AI Cortex Management

```
Extension commands can be logged and analyzed alongside AI API calls:

- Token usage by extension
- Command latency tracking
- Success/failure rates
- Cost attribution (if extension uses paid APIs)
```

---

## Security & Sandboxing

### Transform Execution Sandbox

```go
// sandbox.go
package sandbox

import (
    "github.com/dop251/goja"
    "time"
)

type Sandbox struct {
    runtime       *goja.Runtime
    maxExecutionTime time.Duration
    allowedAPIs   map[string]bool
}

func NewSandbox() *Sandbox {
    runtime := goja.New()
    
    // Disable dangerous APIs
    runtime.Set("require", nil)
    runtime.Set("process", nil)
    runtime.Set("child_process", nil)
    
    // Provide safe APIs
    runtime.Set("console", map[string]interface{}{
        "log": func(msg string) {
            log.Info("[Transform]", msg)
        },
    })
    
    return &Sandbox{
        runtime:         runtime,
        maxExecutionTime: 1 * time.Second,
        allowedAPIs: map[string]bool{
            "JSON":   true,
            "String": true,
            "Array":  true,
            "Object": true,
        },
    }
}

func (s *Sandbox) Execute(code string, data interface{}) (interface{}, error) {
    s.runtime.Set("data", data)
    
    // Execute with timeout
    done := make(chan interface{})
    var result interface{}
    var err error
    
    go func() {
        result, err = s.runtime.RunString(code)
        done <- result
    }()
    
    select {
    case <-done:
        if err != nil {
            return nil, err
        }
        return result.Export(), nil
    case <-time.After(s.maxExecutionTime):
        return nil, fmt.Errorf("transform execution timeout")
    }
}
```

---

## Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| Command interception overhead | <10ms | Transparent to user |
| Transform execution | <100ms | Real-time feel |
| JIT data fetch | <500ms | Acceptable for context injection |
| Dashboard switch | <1s | Smooth UX |
| Community dashboard search | <2s | Responsive discovery |

---

## Migration Path

### Phase 1: Core Proxy (Week 1-2)
- ✅ Command interceptor
- ✅ Event interceptor
- ✅ Basic transform pipeline
- ✅ TiDB integration

### Phase 2: Dashboard Builder (Week 3-4)
- ✅ Visual dashboard designer
- ✅ Component library
- ✅ Live preview
- ✅ Per-project configs

### Phase 3: Community Repository (Week 5-6)
- ✅ Dashboard registry
- ✅ Publish/install flow
- ✅ Rating & reviews
- ✅ Version management

### Phase 4: Advanced Features (Week 7-8)
- ✅ JIT data injection
- ✅ AI-powered transform generation
- ✅ Analytics dashboard
- ✅ Multi-extension orchestration

---

## Example Use Cases

### Use Case 1: Enhanced GitHub Copilot

```yaml
# Inject project-specific context into every Copilot request
jit_data_sources:
  - name: "coding_standards"
    type: "file"
    path: ".devguide/coding-standards.md"
  
  - name: "recent_bugs"
    type: "database"
    query: "SELECT * FROM bugs WHERE status='open' LIMIT 10"

proxy_config:
  commands:
    - id: "copilot.suggest"
      pre_hook: |
        function(args) {
          args[0].context = {
            standards: getCodingStandards(),
            recent_bugs: getRecentBugs(),
            team_conventions: getTeamConventions()
          };
          return args;
        }
```

### Use Case 2: Prettier with Team Styles

```yaml
# Auto-apply team formatting rules before Prettier runs
proxy_config:
  commands:
    - id: "prettier.format"
      pre_hook: |
        function(args) {
          // Apply team-specific transformations
          let code = args[0];
          code = applyTeamQuoteStyle(code);
          code = applyTeamIndentation(code);
          return [code];
        }
      
      post_hook: |
        function(result) {
          // Log formatting changes for analytics
          logFormattingStats(result);
          return result;
        }
```

### Use Case 3: ESLint with Auto-Fix

```yaml
# Automatically fix certain errors, flag others for review
proxy_config:
  commands:
    - id: "eslint.executeAutofix"
      post_hook: |
        function(result) {
          // Separate auto-fixable vs manual review
          const autoFixed = result.filter(e => e.fixable);
          const manualReview = result.filter(e => !e.fixable);
          
          // Send manual review to dashboard
          notifyManualReview(manualReview);
          
          return autoFixed;
        }
```

---

## Open Questions

1. Should we support WebAssembly transforms in addition to JavaScript?
2. What's the maximum transform execution time before timeout?
3. Should community dashboards be sandboxed more strictly than private dashboards?
4. How to handle breaking changes in extension APIs across versions?
5. Should we support dashboard "forks" (derivative works)?

---

## References

**VS Code Extension API:**
- [Extension API Reference](https://code.visualstudio.com/api/references/vscode-api)
- [Extension Capabilities](https://code.visualstudio.com/api/extension-capabilities/overview)

**Similar Patterns:**
- [vscode-proxy](https://github.com/mkloubert/vscode-proxy) - TCP proxy with trace support
- [Intercept Wave](https://github.com/zhongmiao-org/intercept-wave-vscode) - HTTP proxy/mock

**Transform Engines:**
- [Goja](https://github.com/dop251/goja) - JavaScript engine in Go
- [JSONPath](https://goessner.net/articles/JsonPath/) - JSON query language

---

**End of ARTIFACT_22_VSCODE_EXTENSION_ORCHESTRATION.md**
