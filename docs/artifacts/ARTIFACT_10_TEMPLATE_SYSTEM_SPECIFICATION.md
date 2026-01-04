# DevGuide Cockpit: Template System Specification

**Document Type:** Mechanics (Tier 3)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** ARTIFACT_02_TERMINOLOGY_GLOSSARY.md, ARTIFACT_11_DASHBOARD_TEMPLATE_CATALOG.md, ARTIFACT_12_COMMUNITY_GOVERNANCE_SPECIFICATION.md

---

## Executive Summary

This artifact specifies the **template definition and management system** for DevGuide Cockpit. It defines:
- Template definition format (YAML schema + Elm modules)
- Installation and versioning process
- AI-assisted customization wizard
- Community template validation and certification
- Template upgrade and migration paths

**Key Principle:** Templates are immutable once installed. Customization happens through configuration overlays, not template modification.

---

## Template Definition Format

### Directory Structure

```
templates/
└── vibe-coding-v1/
    ├── template.yaml          # Metadata + configuration
    ├── README.md              # Template documentation
    ├── dashboards/
    │   ├── inception/
    │   │   ├── Dashboard.elm  # Elm implementation
    │   │   ├── config.yaml    # Default config
    │   │   └── CAPABILITIES.yaml  # Security manifest
    │   ├── architecture/
    │   ├── techstack/
    │   ├── security/
    │   ├── quality/
    │   └── deployment/
    ├── modules/
    │   ├── VisionStatement.elm
    │   ├── UserStories.elm
    │   └── SuccessMetrics.elm
    ├── guardrails/
    │   ├── baseline.yaml      # System guardrails
    │   └── recommended.yaml   # Template-specific guardrails
    └── ai_prompts/
        ├── inception.yaml     # AI prompts per dashboard
        └── validation.yaml    # AI validation prompts
```

### template.yaml Schema

```yaml
# templates/vibe-coding-v1/template.yaml
id: "vibe-coding-v1"
name: "Vibe Coding Template"
version: "1.0.0"
author:
  name: "DevGuide Team"
  email: "templates@devguide.io"
license: "MIT"
category: "web-development"
certification_tier: "tier1_certified"  # or tier2_community, tier3_experimental

description: |
  Emulates the Lovable/Cursor experience for non-technical "vibe coders" 
  building web applications with AI assistance. Minimal decisions, maximum 
  guidance, fast iteration.

tags:
  - "web-development"
  - "ai-assisted"
  - "low-code"
  - "beginner-friendly"

target_audience:
  skill_level: "beginner"
  use_cases:
    - "Building web applications"
    - "Rapid prototyping"
    - "Learning software development"

dependencies:
  min_devguide_version: "0.1.0"
  required_capabilities:
    - "RequestAICompletion"
    - "RenderCanvas"
    - "GitIntegration"

# Dashboard configuration
dashboards:
  - id: "inception-001"
    name: "Project Inception"
    description: "Define project goals and success criteria"
    sequence: 1
    required: true
    elm_module: "Dashboard.Inception"
    default_config:
      max_features: 5
      require_success_metrics: true
    ai_prompts:
      - name: "vision_generator"
        prompt_file: "ai_prompts/inception.yaml"
        max_tokens: 2000
    
  - id: "architecture-001"
    name: "Architecture Canvas"
    description: "Visual system design"
    sequence: 2
    required: true
    elm_module: "Dashboard.Architecture"
    capabilities:
      - "RenderCanvas"
      - "ExecuteJavaScript"  # For D3.js
    default_config:
      max_services: 3
      enforce_microservices: false

  - id: "techstack-001"
    name: "Tech Stack Matrix"
    description: "Technology selection with AI recommendations"
    sequence: 3
    required: true
    elm_module: "Dashboard.TechStack"
    
  - id: "security-001"
    name: "Security Checklist"
    sequence: 4
    required: false
    elm_module: "Dashboard.Security"
    
  - id: "quality-001"
    name: "Quality Gates"
    sequence: 5
    required: false
    elm_module: "Dashboard.Quality"
    
  - id: "deployment-001"
    name: "Deployment Orchestrator"
    sequence: 6
    required: true
    elm_module: "Dashboard.Deployment"

# Guardrail configuration
guardrails:
  baseline:
    - id: "max_service_count"
      type: "constraint"
      value: 3
      enforcement: "hard"  # Cannot override
      rationale: "Vibe Coding focuses on simplicity"
    
    - id: "api_versioning_required"
      type: "validation"
      value: true
      enforcement: "soft"  # Can override with rationale
    
  recommended:
    - id: "test_coverage_min"
      type: "metric"
      value: 80
      enforcement: "soft"

# Propagation rules
propagation_rules:
  default_mode: "gated"  # gated, auto, hybrid
  auto_sync_enabled: false
  impact_threshold: "medium"  # low, medium, high
  require_review_for:
    - "database_change"
    - "language_change"
    - "architecture_change"

# AI configuration
ai_settings:
  provider: "claude"  # claude, openai, local
  monthly_token_budget: 1000000
  enable_guardrail_validation: true
  enable_code_suggestions: true
  enable_documentation_generation: true

# Version history
changelog:
  - version: "1.0.0"
    date: "2026-01-01"
    changes:
      - "Initial release"
      - "6 core dashboards"
      - "AI-assisted configuration"
```

---

## Installation Process

### 1. Template Discovery

**Master Control Dashboard:**
```elm
-- Template marketplace view
type alias TemplateMetadata =
    { id : String
    , name : String
    , version : String
    , author : Author
    , description : String
    , downloads : Int
    , rating : Float
    , certificationTier : CertificationTier
    , lastUpdated : Time.Posix
    }

viewTemplateMarketplace : Model -> Html Msg
viewTemplateMarketplace model =
    div [ class "template-marketplace" ]
        [ div [ class "filters" ]
            [ select [ onInput FilterByCategory ]
                [ option [] [ text "All Categories" ]
                , option [] [ text "Web Development" ]
                , option [] [ text "Data Science" ]
                , option [] [ text "Note Taking" ]
                ]
            , input 
                [ type_ "search"
                , placeholder "Search templates..."
                , onInput SearchTemplates
                ] []
            ]
        , div [ class "template-grid" ]
            (List.map viewTemplateCard model.availableTemplates)
        ]

viewTemplateCard : TemplateMetadata -> Html Msg
viewTemplateCard template =
    div [ class "template-card", onClick (SelectTemplate template.id) ]
        [ div [ class "header" ]
            [ h3 [] [ text template.name ]
            , span [ class ("tier-" ++ certificationTierToString template.certificationTier) ]
                [ text (certificationTierToString template.certificationTier) ]
            ]
        , p [ class "description" ] [ text template.description ]
        , div [ class "metadata" ]
            [ span [ class "downloads" ]
                [ icon "download"
                , text (formatNumber template.downloads ++ " downloads")
                ]
            , span [ class "rating" ]
                [ icon "star"
                , text (String.fromFloat template.rating ++ "/5")
                ]
            ]
        , div [ class "author" ]
            [ text ("by " ++ template.author.name) ]
        , button [ onClick (InstallTemplate template.id) ]
            [ text "Install Template" ]
        ]
```

### 2. Installation Flow

```
User clicks "Install Template"
    ↓
[1] Download template files from registry
    ↓
[2] Verify template signature (prevent tampering)
    ↓
[3] Run security scan (detect malicious code)
    ↓
[4] Show capability permissions request
    ↓
[5] User approves/denies capabilities
    ↓
[6] AI-assisted configuration wizard
    ↓
[7] Generate project from template
    ↓
[8] Create TiDB entries (project, dashboards, modules)
    ↓
[9] Initialize Git repository
    ↓
[10] Open first dashboard (Project Inception)
```

**Go API:**
```go
// api/templates/installer.go
type TemplateInstaller struct {
    registry   *TemplateRegistry
    validator  *SecurityValidator
    aiLayer    *AILayer
    db         *sql.DB
}

func (ti *TemplateInstaller) Install(ctx context.Context, templateID, projectName string, userID string) (*Project, error) {
    // [1] Download template
    template, err := ti.registry.Download(ctx, templateID)
    if err != nil {
        return nil, fmt.Errorf("download failed: %w", err)
    }
    
    // [2] Verify signature
    if err := ti.validator.VerifySignature(template); err != nil {
        return nil, fmt.Errorf("signature verification failed: %w", err)
    }
    
    // [3] Security scan
    scanResult, err := ti.validator.ScanForMaliciousCode(template)
    if err != nil || !scanResult.Safe {
        return nil, fmt.Errorf("security scan failed: %v", scanResult.Issues)
    }
    
    // [4] Check capabilities
    requiredCaps := template.GetRequiredCapabilities()
    if err := ti.checkUserPermissions(userID, requiredCaps); err != nil {
        return nil, err
    }
    
    // [5] User approval (interactive prompt via SSE)
    approved, err := ti.requestUserApproval(ctx, userID, requiredCaps)
    if err != nil || !approved {
        return nil, fmt.Errorf("user denied capability permissions")
    }
    
    // [6] AI configuration wizard
    config, err := ti.runConfigurationWizard(ctx, template, projectName, userID)
    if err != nil {
        return nil, err
    }
    
    // [7-9] Generate project
    project, err := ti.generateProject(ctx, template, config, userID)
    if err != nil {
        return nil, err
    }
    
    return project, nil
}
```

### 3. AI-Assisted Configuration Wizard

**Conversational Workflow:**
```
AI: "Let's set up your Vibe Coding project! What's the project name?"
User: "TravelBuddy"

AI: "Great! What type of application are you building?"
User: "A travel planning app"

AI: "Perfect. I'll suggest some features:
     1. User authentication
     2. Trip itinerary builder
     3. Budget tracker
     4. Destination recommendations
     
     Do these sound right, or would you like to customize?"
User: "Add photo sharing"

AI: "Added! Now, which databases would you like to use?
     I recommend: TiDB (for trip data) + S3 (for photos)
     Sound good?"
User: "Yes"

AI: "Configuring guardrails... I'm setting:
     - Max 3 microservices (trip, user, photo)
     - Test coverage minimum 80%
     - API versioning required
     
     Ready to create your project?"
User: "Yes!"

AI: [Creates project, initializes dashboards]
    "Your project is ready! Opening Project Inception dashboard..."
```

**Implementation:**
```go
// api/templates/config_wizard.go
func (ti *TemplateInstaller) runConfigurationWizard(ctx context.Context, template *Template, projectName string, userID string) (*ProjectConfig, error) {
    conversation := ti.aiLayer.NewConversation(ctx, userID)
    
    // Step 1: Project basics
    projectType, err := conversation.Ask(
        "What type of application are you building?",
        []string{"web app", "mobile app", "data science", "other"},
    )
    
    // Step 2: Feature selection
    features, err := conversation.AskMultiSelect(
        fmt.Sprintf("I recommend these features for a %s. Select all that apply:", projectType),
        template.GetRecommendedFeatures(projectType),
    )
    
    // Step 3: Tech stack
    techStack, err := conversation.AskWithRecommendations(
        "Which technologies would you like to use?",
        template.GetRecommendedTechStack(projectType),
    )
    
    // Step 4: Guardrails
    guardrails, err := conversation.ConfirmGuardrails(
        template.GetDefaultGuardrails(),
    )
    
    // Generate final config
    return &ProjectConfig{
        Name:       projectName,
        Type:       projectType,
        Features:   features,
        TechStack:  techStack,
        Guardrails: guardrails,
    }, nil
}
```

---

## Versioning & Upgrades

### Semantic Versioning

**Version Format:** `MAJOR.MINOR.PATCH`

- **MAJOR:** Breaking changes (incompatible dashboard APIs)
- **MINOR:** New features (backward-compatible)
- **PATCH:** Bug fixes (no API changes)

**Example:**
```
vibe-coding-v1.0.0 → vibe-coding-v1.1.0  (new dashboard added)
vibe-coding-v1.1.0 → vibe-coding-v2.0.0  (dashboard API changed)
```

### Template Update Notification

```elm
-- Master Control Dashboard shows update notification
viewUpdateAvailable : Template -> Html Msg
viewUpdateAvailable template =
    div [ class "update-notification" ]
        [ icon "info"
        , text ("Update available: " ++ template.name ++ " v" ++ template.latestVersion)
        , button [ onClick (ShowUpdateDetails template.id) ]
            [ text "View Changes" ]
        , button [ onClick (UpgradeTemplate template.id) ]
            [ text "Upgrade Now" ]
        ]
```

### Upgrade Process

**Non-Breaking Changes (Patch/Minor):**
```
User clicks "Upgrade Now"
    ↓
System checks compatibility
    ↓
[Automated] Download new template version
    ↓
[Automated] Run migration script (if any)
    ↓
[Automated] Update TiDB entries
    ↓
[Automated] Reload dashboards
    ↓
User sees "Upgrade complete" notification
```

**Breaking Changes (Major):**
```
User clicks "Upgrade Now"
    ↓
System detects breaking changes
    ↓
Show interactive migration wizard:
    "⚠️ This upgrade contains breaking changes:
     
     1. Dashboard API changed:
        - Old: `getViolations()`
        - New: `fetchViolations(context)`
     
     2. Config format changed:
        - Old: guardrails.yaml
        - New: guardrails/*.yaml (split files)
     
     3. Required new capability:
        - ExecuteJavaScript (for new visualization)"
    ↓
User reviews changes
    ↓
Show side-by-side diff:
    [Old Config]  |  [New Config]
    max_services: 3  |  services:
                     |    max_count: 3
                     |    enforce_limit: true
    ↓
User confirms upgrade
    ↓
[Automated] Create rollback snapshot
    ↓
[Semi-Automated] Apply breaking changes with user confirmation
    ↓
User tests new version
    ↓
Either:
    [✓] User confirms "Looks good" → Delete rollback snapshot after 30 days
    [✗] User clicks "Rollback" → Restore from snapshot
```

**Rollback Mechanism:**
```go
// api/templates/upgrader.go
func (tu *TemplateUpgrader) RollbackUpgrade(ctx context.Context, projectID string) error {
    // Load rollback snapshot
    snapshot, err := tu.loadRollbackSnapshot(ctx, projectID)
    if err != nil {
        return err
    }
    
    // Verify snapshot age (max 30 days)
    if time.Since(snapshot.CreatedAt) > 30*24*time.Hour {
        return fmt.Errorf("rollback snapshot expired")
    }
    
    // Start transaction
    tx, err := tu.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    // Restore template version
    _, err = tx.ExecContext(ctx, `
        UPDATE projects 
        SET template_id = ?, template_version = ?
        WHERE id = ?
    `, snapshot.TemplateID, snapshot.TemplateVersion, projectID)
    
    // Restore dashboards
    for _, dashboard := range snapshot.Dashboards {
        err = tu.restoreDashboard(ctx, tx, dashboard)
        if err != nil {
            return err
        }
    }
    
    // Commit
    return tx.Commit()
}
```

---

## Customization Model

### Configuration Overlay System

**Principle:** Never modify template source, only layer configuration on top.

```yaml
# .devguide/project.yaml (user's project config)
template:
  id: "vibe-coding-v1"
  version: "1.0.0"

# Overlay configuration (overrides template defaults)
config_overrides:
  dashboards:
    inception-001:
      max_features: 10  # Override template's max_features: 5
      custom_fields:
        - name: "target_market"
          type: "text"
          required: true
    
    security-001:
      enabled: true  # Template had this as optional
      strict_mode: true  # Custom setting not in template

  guardrails:
    max_service_count:
      value: 5  # Override template's 3
      rationale: "Complex e-commerce requires more services"

  ai_settings:
    monthly_token_budget: 2000000  # Override template's 1M
```

**Elm: Load Merged Configuration**
```elm
-- Dashboard/Inception.elm
type alias Config =
    { maxFeatures : Int
    , requireSuccessMetrics : Bool
    , customFields : List CustomField
    }

loadConfig : ProjectId -> Task Error Config
loadConfig projectId =
    Http.task
        { method = "GET"
        , url = "/api/projects/" ++ projectId ++ "/config/inception-001"
        , headers = []
        , body = Http.emptyBody
        , resolver = Http.stringResolver (handleResponse decodeConfig)
        , timeout = Nothing
        }

-- API returns merged config (template defaults + user overrides)
```

**Go API: Merge Configuration**
```go
// api/config/merger.go
func (cm *ConfigMerger) GetDashboardConfig(ctx context.Context, projectID, dashboardID string) (*DashboardConfig, error) {
    // Load template defaults
    template, err := cm.loadProjectTemplate(ctx, projectID)
    if err != nil {
        return nil, err
    }
    
    templateConfig := template.GetDashboardConfig(dashboardID)
    
    // Load user overrides
    userOverrides, err := cm.loadUserOverrides(ctx, projectID, dashboardID)
    if err != nil {
        // No overrides found, use template defaults
        return templateConfig, nil
    }
    
    // Merge (user overrides win)
    return cm.merge(templateConfig, userOverrides), nil
}
```

---

## Community Template Validation

### Submission Process

```
Developer creates custom template
    ↓
Runs local validation:
    $ devguide-cli template validate ./my-template
    ↓
Submits to community registry:
    $ devguide-cli template publish ./my-template
    ↓
Automated checks:
    ✓ template.yaml syntax valid
    ✓ All Elm modules compile
    ✓ No malicious code patterns
    ✓ Capability declarations present
    ✓ Security scan passed
    ↓
Manual review queue (Tier 1 certification)
    OR
Auto-approved (Tier 2 community, use at own risk)
```

**Certification Tiers:**

| Tier | Requirements | Review Process | Trust Level |
|------|-------------|----------------|-------------|
| **Tier 1 Certified** | Full code review, security audit, documentation | Manual by DevGuide team | High |
| **Tier 2 Community** | Automated checks only | Automated | Medium |
| **Tier 3 Experimental** | Basic syntax validation | None | Low |

### Security Validation

```go
// api/templates/security_validator.go
func (sv *SecurityValidator) ScanForMaliciousCode(template *Template) (*ScanResult, error) {
    result := &ScanResult{Safe: true, Issues: []SecurityIssue{}}
    
    // Check 1: No network calls without NetworkAccess capability
    if sv.detectNetworkCalls(template.ElmCode) && !template.HasCapability(NetworkAccess) {
        result.Safe = false
        result.Issues = append(result.Issues, SecurityIssue{
            Severity: "high",
            Message: "Network calls detected without NetworkAccess capability",
        })
    }
    
    // Check 2: No file system access without FileSystemAccess capability
    if sv.detectFileSystemCalls(template.ElmCode) && !template.HasCapability(FileSystemAccess) {
        result.Safe = false
        result.Issues = append(result.Issues, SecurityIssue{
            Severity: "high",
            Message: "File system access without permission",
        })
    }
    
    // Check 3: No eval() or similar dangerous patterns
    if sv.detectDangerousPatterns(template.JavaScriptPorts) {
        result.Safe = false
        result.Issues = append(result.Issues, SecurityIssue{
            Severity: "critical",
            Message: "Dangerous JavaScript patterns detected (eval, Function constructor)",
        })
    }
    
    // Check 4: Validate external dependencies
    for _, dep := range template.ExternalDependencies {
        if !sv.isWhitelistedDependency(dep) {
            result.Safe = false
            result.Issues = append(result.Issues, SecurityIssue{
                Severity: "medium",
                Message: fmt.Sprintf("Non-whitelisted dependency: %s", dep),
            })
        }
    }
    
    return result, nil
}
```

---

## Database Schema

### templates Table

```sql
CREATE TABLE templates (
    id VARCHAR(255) PRIMARY KEY,
    name VARCHAR(500) NOT NULL,
    version VARCHAR(50) NOT NULL,
    author_id VARCHAR(255) NOT NULL,
    category VARCHAR(100),
    certification_tier ENUM('tier1_certified', 'tier2_community', 'tier3_experimental'),
    description TEXT,
    manifest_json JSON NOT NULL,  -- Full template.yaml
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    downloads INT DEFAULT 0,
    rating DECIMAL(3,2) DEFAULT 0.00,
    
    INDEX idx_category (category),
    INDEX idx_tier (certification_tier),
    INDEX idx_downloads (downloads),
    UNIQUE KEY uk_id_version (id, version)
);

CREATE TABLE template_installations (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    template_id VARCHAR(255) NOT NULL,
    template_version VARCHAR(50) NOT NULL,
    project_id VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    installed_at DATETIME NOT NULL,
    config_overrides JSON,  -- User's customizations
    
    INDEX idx_project (project_id),
    INDEX idx_template (template_id),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE
);
```

---

## Testing Strategy

### Template Validation Tests

```elm
-- test/TemplateValidatorTest.elm
testTemplateStructure : Test
testTemplateStructure =
    describe "Template structure validation"
        [ test "valid template passes validation" <|
            \() ->
                let
                    template = loadTemplate "vibe-coding-v1"
                in
                Expect.ok (validateTemplate template)
        
        , test "missing required dashboard fails" <|
            \() ->
                let
                    template = loadTemplate "invalid-missing-dashboard"
                in
                Expect.err (validateTemplate template)
        
        , test "invalid capability declaration fails" <|
            \() ->
                let
                    template = loadTemplate "invalid-capability"
                in
                Expect.err (validateTemplate template)
        ]
```

---

## Open Questions

1. Should template authors be able to monetize templates? (Suggest: Post-MVP)
2. Maximum template size limit? (Suggest: 50 MB)
3. Support for template dependencies (template A extends template B)? (Suggest: Post-MVP)
4. Allow users to fork certified templates? (Suggest: Yes, but loses certification)

---

**End of ARTIFACT_10_TEMPLATE_SYSTEM_SPECIFICATION.md**
