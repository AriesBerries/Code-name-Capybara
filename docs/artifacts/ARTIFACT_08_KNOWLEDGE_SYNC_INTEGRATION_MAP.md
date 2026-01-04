# DevGuide Cockpit: Knowledge Synchronization Integration Map

**Document Type:** Foundation  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** TERMINOLOGY_GLOSSARY.md, THE_BIN_SPECIFICATION.md, MASTER_CONTROL_DASHBOARD.md

---

## Executive Summary

This artifact demonstrates how the **6-layer Knowledge Synchronization Engine** from the original conceptual framework maps to DevGuide Cockpit's implementation. It proves that **The Bin IS the Merge Gate**, showing how the system achieves "structural impossibility of desynchronization" through compile-time guarantees, runtime enforcement, and transactional commits.

**Core Philosophy:**
> "Truth has a single origin point, and everything else (code, docs, tests, specs) are projections of that truth."

---

## Original 6-Layer Concept

### Layer Definitions

| Layer | What It Represents | Purpose |
|-------|-------------------|---------|
| **1. REALITY** | What actually happens when code runs | Runtime execution logs, traces, telemetry |
| **2. SPECIFICATION** | What we promised the system would do | Config files, API contracts, schemas |
| **3. DOCUMENTATION** | What we told humans the system does | README, API docs, architecture diagrams |
| **4. DECISIONS** | Why we built the system this way | ADRs, decision logs, rationale |
| **5. VALIDATION** | Continuous checks that all layers align | Dependency graphs, AI validation, guardrails |
| **6. MERGE GATE** | Final checkpoint before changes go live | Atomic commit boundary, validation aggregator |

**Guarantee:** Changes propagate automatically across all layers. Impossible to have "stale docs" or "outdated tests" because all layers update atomically through the Merge Gate.

---

## DevGuide Cockpit Implementation Mapping

```
ORIGINAL 6-LAYER CONCEPT          DEVGUIDE COCKPIT IMPLEMENTATION
═══════════════════════════       ═══════════════════════════════

Layer 1: REALITY                  Runtime Execution Logs
(what actually runs)              ├─ Elm runtime (browser console)
                                  ├─ Go API server logs (JSON structured)
                                  ├─ Rust WASM execution traces
                                  └─ TiDB query performance metrics

Layer 2: SPECIFICATION            Dashboard Configuration Files
(what we promised)                ├─ Project profiles (.elm config)
                                  ├─ Workflow module specs (JSON)
                                  ├─ Guardrail rule definitions (YAML)
                                  ├─ Template manifests
                                  └─ API contracts (OpenAPI specs)

Layer 3: DOCUMENTATION            Generated Documentation
(what we told humans)             ├─ Docusaurus static site
                                  ├─ Jinja2-generated markdown
                                  ├─ In-dashboard help text
                                  ├─ API reference (auto-generated)
                                  └─ Architecture diagrams (PlantUML)

Layer 4: DECISIONS                Audit Log + Version History
(why we built this way)           ├─ Git commit messages
                                  ├─ project_change_log table (TiDB)
                                  ├─ Decision records in dashboards
                                  ├─ Guardrail override rationale
                                  └─ ADRs (Architecture Decision Records)

Layer 5: VALIDATION               Propagation Engine + AI Validation
(continuous alignment)            ├─ Dependency graph checks (Rust)
                                  ├─ AI-powered guardrail validation
                                  ├─ Elm compiler type checks
                                  ├─ Real-time SSE status updates
                                  └─ Cross-dashboard impact analysis

Layer 6: MERGE GATE               THE BIN (Synchronization Layer)
(final checkpoint)                ├─ SyncBuffer<T> (Rust data structure)
                                  ├─ Validation checkpoint UI (always visible)
                                  ├─ Atomic commit to TiDB + Git
                                  ├─ Propagation orchestration
                                  └─ Rollback capability (30-day window)
```

---

## Critical Insight: The Bin IS the Merge Gate

### Three Manifestations of The Bin

**1. Data Structure (Rust Implementation)**
```rust
// bin/sync_buffer.rs
pub struct SyncBuffer<T> {
    items: Vec<BinItem<T>>,
    validation_status: ValidationStatus,
    checkpoint_hash: String,  // SHA-256 of current state
    rollback_snapshots: VecDeque<Snapshot>,
}

pub struct BinItem<T> {
    id: Uuid,
    payload: T,
    source_dashboard: DashboardId,
    affected_dashboards: Vec<DashboardId>,
    violations: Vec<Violation>,
    status: ItemStatus,  // Pending, Approved, Rejected
    layer_validations: LayerValidationResults,
}

pub struct LayerValidationResults {
    reality_check: bool,      // Layer 1: Runtime logs show no errors
    spec_compliance: bool,    // Layer 2: Matches config schemas
    docs_generated: bool,     // Layer 3: Docs build successful
    rationale_provided: bool, // Layer 4: Decision recorded
    all_checks_pass: bool,    // Layer 5: All validations green
}
```

**2. Conceptual Checkpoint (Validation Moment)**

The Bin aggregates validation from all 6 layers before allowing commit:

```
User completes work in Dashboard A
    ↓
Dashboard reports change to The Bin
    ↓
The Bin runs Layer 1-5 validation:
    ├─ Layer 1: Check runtime logs for errors
    ├─ Layer 2: Validate config file syntax
    ├─ Layer 3: Verify docs can be generated
    ├─ Layer 4: Require rationale for critical changes
    └─ Layer 5: Run dependency checks + AI validation
    ↓
If ANY layer fails:
    ├─ Item marked "Blocked" in Bin
    ├─ User sees violations with fix suggestions
    └─ Cannot proceed until resolved
    ↓
If ALL layers pass:
    ├─ Item marked "Ready to Commit"
    ├─ User reviews impact assessment
    └─ User approves → Layer 6 executes
    ↓
Layer 6 (The Bin as Merge Gate):
    ├─ Atomic commit to TiDB (all tables)
    ├─ Git commit (code + config + docs together)
    ├─ Propagate to downstream dashboards
    └─ Refresh all dashboard UIs via SSE
```

**3. Visible UI Element**

```
┌─────────────────────────────────────────────────────────┐
│  The Bin - Synchronization Queue          [Collapse]    │
├─────────────────────────────────────────────────────────┤
│  🔴 CRITICAL (2)                                        │
│  ├─ Architecture change affects 5 dashboards           │
│  │   Layer Status:                                      │
│  │   [✓] Reality [✓] Spec [✓] Docs [!] Decisions [✓] Validation │
│  │   ❌ Missing: Rationale for microservices split     │
│  │   [Add Rationale] [Review Impact] [Reject]          │
│  │                                                      │
│  └─ Tech stack change breaks deployment config         │
│      Layer Status:                                      │
│      [✓] Reality [!] Spec [✓] Docs [✓] Decisions [✓] Validation │
│      ❌ Violation: Docker config missing for new runtime│
│      [Fix Now] [Defer] [Override]                      │
│                                                          │
│  🟡 WARNING (5)                                         │
│  └─ [Expand to view warnings]                          │
│                                                          │
│  🟢 APPROVED (12)                                       │
│  └─ [View commit history]                              │
│                                                          │
│  Last Validation: 30 seconds ago                        │
│  Next Checkpoint: Architecture Lock (after 2 approvals) │
└─────────────────────────────────────────────────────────┘
```

---

## Layer-by-Layer Integration Details

### Layer 1: REALITY → Runtime Execution Logs

**Implementation:**

```elm
-- Dashboard/Architecture.elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        APICallFailed error ->
            let
                _ = Debug.log "Architecture Dashboard API Error" error
            in
            ( { model | error = Just error }
            , reportToBin 
                { dashboard_id = "architecture-001"
                , violation_type = "runtime_error"
                , layer = Layer1Reality
                , details = error
                }
            )
```

**Go API Logger:**
```go
// api/logging/structured_logger.go
func LogAPICall(ctx context.Context, method, endpoint string, duration time.Duration, err error) {
    entry := logrus.WithFields(logrus.Fields{
        "method":   method,
        "endpoint": endpoint,
        "duration_ms": duration.Milliseconds(),
        "user_id":  ctx.Value("user_id"),
    })
    
    if err != nil {
        entry.Error(err.Error())
        // Send to The Bin
        binService.ReportViolation(ctx, Violation{
            Layer: Layer1Reality,
            Type: "api_error",
            Message: err.Error(),
        })
    } else {
        entry.Info("API call successful")
    }
}
```

**Dashboard Integration:**
- Master Control Dashboard shows aggregate runtime health
- Individual dashboards display real-time logs for their modules
- Errors automatically create items in The Bin for review

---

### Layer 2: SPECIFICATION → Dashboard Configuration Files

**Implementation:**

```yaml
# .devguide/project.yaml
name: "E-commerce Platform"
version: "1.2.0"
archetype: "vibe-coding-v1"

specification:
  api_versioning_required: true
  max_service_count: 3
  required_languages: ["elm", "go", "rust"]
  
  services:
    - name: "product-catalog"
      port: 8080
      dependencies: ["postgres"]
    - name: "checkout"
      port: 8081
      dependencies: ["product-catalog", "payment-gateway"]
```

**Validation (Rust WASM):**
```rust
// validation/spec_validator.rs
pub fn validate_spec_compliance(project: &Project) -> Result<(), SpecViolation> {
    // Check service count
    if project.services.len() > project.spec.max_service_count {
        return Err(SpecViolation::TooManyServices {
            current: project.services.len(),
            max: project.spec.max_service_count,
        });
    }
    
    // Validate service dependencies
    for service in &project.services {
        for dep in &service.dependencies {
            if !project.service_exists(dep) {
                return Err(SpecViolation::MissingDependency {
                    service: service.name.clone(),
                    missing_dep: dep.clone(),
                });
            }
        }
    }
    
    Ok(())
}
```

**Dashboard Integration:**
- Tech Stack Dashboard edits `project.yaml`
- Change queued in The Bin
- Layer 2 validation runs automatically
- Violations shown before commit

---

### Layer 3: DOCUMENTATION → Generated Documentation

**Implementation:**

```python
# scripts/generate_docs.py
import jinja2
from pathlib import Path

def generate_documentation(project_yaml_path):
    """Generate docs from project specification."""
    with open(project_yaml_path) as f:
        project = yaml.safe_load(f)
    
    env = jinja2.Environment(loader=jinja2.FileSystemLoader('templates/'))
    
    # Generate README
    readme_template = env.get_template('README.md.jinja2')
    readme_content = readme_template.render(project=project)
    Path('README.md').write_text(readme_content)
    
    # Generate API docs
    api_template = env.get_template('API_REFERENCE.md.jinja2')
    api_content = api_template.render(services=project['services'])
    Path('docs/API_REFERENCE.md').write_text(api_content)
    
    # Generate architecture diagram
    generate_plantuml_diagram(project)
    
    return True  # Success
```

**The Bin Integration:**
```rust
// bin/layer3_validator.rs
pub fn validate_docs_generation(project_id: &str) -> Result<(), Layer3Violation> {
    let script_path = Path::new("scripts/generate_docs.py");
    
    let output = Command::new("python3")
        .arg(script_path)
        .arg(project_id)
        .output()?;
    
    if !output.status.success() {
        return Err(Layer3Violation::DocGenerationFailed {
            stderr: String::from_utf8_lossy(&output.stderr).to_string(),
        });
    }
    
    // Verify files exist
    let expected_files = vec!["README.md", "docs/API_REFERENCE.md"];
    for file in expected_files {
        if !Path::new(file).exists() {
            return Err(Layer3Violation::MissingDocFile {
                path: file.to_string(),
            });
        }
    }
    
    Ok(())
}
```

---

### Layer 4: DECISIONS → Audit Log + Version History

**Implementation:**

```sql
-- TiDB Schema
CREATE TABLE project_decisions (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    project_id VARCHAR(255) NOT NULL,
    decision_type ENUM('architecture', 'tech_stack', 'security', 'deployment'),
    title VARCHAR(500) NOT NULL,
    rationale TEXT NOT NULL,
    alternatives_considered JSON,
    decided_by VARCHAR(255) NOT NULL,
    decided_at DATETIME NOT NULL,
    superseded_by BIGINT,  -- References another decision
    
    INDEX idx_project (project_id),
    INDEX idx_type (decision_type),
    INDEX idx_timestamp (decided_at)
);
```

**Dashboard Decision Capture:**
```elm
-- When user makes architectural change
type Msg
    = ArchitectureChanged ArchitectureChange
    | RationaleProvided String
    | AlternativesListed (List Alternative)

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        ArchitectureChanged change ->
            -- Require rationale before allowing commit
            if String.isEmpty model.rationale then
                ( { model | showRationaleModal = True }
                , Cmd.none
                )
            else
                ( model
                , reportToBin
                    { change = change
                    , rationale = model.rationale
                    , alternatives = model.alternatives
                    }
                )
```

**Validation:**
```rust
// bin/layer4_validator.rs
pub fn validate_decision_recorded(item: &BinItem) -> Result<(), Layer4Violation> {
    if item.is_critical_change() && item.rationale.is_none() {
        return Err(Layer4Violation::RationaleMissing {
            change_type: item.change_type.clone(),
            message: "Critical changes require documented rationale".to_string(),
        });
    }
    
    if item.rationale.as_ref().map(|r| r.len()) < Some(100) {
        return Err(Layer4Violation::RationaleInsufficient {
            current_length: item.rationale.as_ref().map(|r| r.len()).unwrap_or(0),
            minimum_required: 100,
        });
    }
    
    Ok(())
}
```

---

### Layer 5: VALIDATION → Propagation Engine + AI

**Implementation:**

**Dependency Graph Checks (Rust WASM):**
```rust
// propagation/dependency_graph.rs
pub fn analyze_impact(change: &Change, graph: &DependencyGraph) -> ImpactAnalysis {
    let affected_nodes = graph.downstream_dependencies(&change.source);
    
    ImpactAnalysis {
        affected_dashboards: affected_nodes.dashboards,
        affected_modules: affected_nodes.modules,
        estimated_token_cost: estimate_ai_cost(&affected_nodes),
        estimated_time_minutes: estimate_propagation_time(&affected_nodes),
        criticality: calculate_criticality(&affected_nodes),
    }
}
```

**AI-Powered Validation (Go API):**
```go
// api/validation/ai_validator.go
func ValidateWithAI(ctx context.Context, item BinItem) (*AIValidationResult, error) {
    prompt := fmt.Sprintf(`
Validate this change against project guardrails:

Change Type: %s
Dashboard: %s
Details: %s

Project Guardrails:
%s

Does this change violate any guardrails? Provide specific violations if any.
`, item.ChangeType, item.SourceDashboard, item.Details, item.ProjectGuardrails)

    response, err := aiLayer.Complete(ctx, AIRequest{
        Prompt: prompt,
        MaxTokens: 1000,
        Temperature: 0.1,  // Low temp for consistent validation
    })
    
    if err != nil {
        return nil, err
    }
    
    return parseAIValidationResponse(response)
}
```

**Cross-Dashboard Validation:**
```
Architecture Dashboard defines microservices
    ↓
Security Dashboard validates:
    - Auth strategy per service?
    - Network policies defined?
    - Secrets management configured?
    ↓
Deployment Dashboard validates:
    - Docker configs present?
    - CI/CD pipeline updated?
    - Health check endpoints defined?
    ↓
All results flow to The Bin
```

---

### Layer 6: MERGE GATE → THE BIN

**Atomic Commit Implementation:**

```go
// api/bin/commit_executor.go
func (be *BinExecutor) ExecuteCommit(ctx context.Context, binItemID string) error {
    // Start distributed transaction
    tx, err := be.db.BeginTx(ctx, &sql.TxOptions{Isolation: sql.LevelSerializable})
    if err != nil {
        return fmt.Errorf("failed to start transaction: %w", err)
    }
    defer tx.Rollback()  // Rollback if not committed
    
    // Step 1: Load bin item
    item, err := be.loadBinItem(ctx, tx, binItemID)
    if err != nil {
        return err
    }
    
    // Step 2: Verify all 6 layers passed
    if !item.AllLayersPass() {
        return fmt.Errorf("cannot commit: validation failures in layers %v", item.FailedLayers())
    }
    
    // Step 3: Update TiDB (all affected tables)
    for _, tableName := range item.AffectedTables {
        if err := be.updateTable(ctx, tx, tableName, item.Changes[tableName]); err != nil {
            return fmt.Errorf("failed to update %s: %w", tableName, err)
        }
    }
    
    // Step 4: Git commit (code + config + docs together)
    gitCommit := GitCommit{
        Message: item.CommitMessage,
        Files: []string{
            "project.yaml",
            "README.md",
            "docs/API_REFERENCE.md",
            "services/*/config.yaml",
        },
        Author: item.UserID,
    }
    
    if err := be.gitService.Commit(ctx, gitCommit); err != nil {
        return fmt.Errorf("git commit failed: %w", err)
    }
    
    // Step 5: Propagate to downstream dashboards
    for _, dashboardID := range item.AffectedDashboards {
        if err := be.propagateChange(ctx, tx, dashboardID, item); err != nil {
            return fmt.Errorf("propagation failed: %w", err)
        }
    }
    
    // Step 6: Create rollback snapshot
    snapshot := be.createSnapshot(ctx, item.ProjectID)
    if err := be.saveSnapshot(ctx, tx, snapshot); err != nil {
        return fmt.Errorf("snapshot creation failed: %w", err)
    }
    
    // Step 7: Commit transaction (atomic across TiDB + Git)
    if err := tx.Commit(); err != nil {
        // Rollback git commit if DB commit fails
        be.gitService.Revert(ctx, gitCommit.SHA)
        return fmt.Errorf("transaction commit failed: %w", err)
    }
    
    // Step 8: Notify all dashboards via SSE
    be.sseService.BroadcastUpdate(ctx, SSEUpdate{
        ProjectID: item.ProjectID,
        Type: "propagation_complete",
        AffectedDashboards: item.AffectedDashboards,
    })
    
    return nil
}
```

---

## Guaranteed Synchronization Properties

### Property 1: Single Atomic Commit

**Guarantee:** All layers update together or not at all.

**Implementation:**
- TiDB distributed transaction spans all affected tables
- Git commit includes code, config, and generated docs
- If any step fails, entire transaction rolls back
- No partial updates possible

**Example Failure Scenario:**
```
User approves change
    → TiDB updates succeed
    → Git commit fails (network issue)
    → TiDB transaction automatically rolls back
    → User sees "Commit failed, please retry"
    → Project state unchanged
```

### Property 2: No Stale Documentation

**Guarantee:** Documentation always reflects current specification.

**Implementation:**
```
Specification Change (Layer 2)
    ↓
Triggers docs generation (Layer 3)
    ↓
If docs generation fails:
    → Layer 3 validation fails
    → The Bin blocks commit
    → User must fix docs template
    ↓
Only commits if docs successfully generated
```

### Property 3: Traceable Decisions

**Guarantee:** Every critical change has recorded rationale.

**Implementation:**
```rust
// bin/critical_change_detector.rs
pub fn is_critical_change(change: &Change) -> bool {
    match change.change_type {
        ChangeType::ArchitectureModification => true,
        ChangeType::SecurityPolicyChange => true,
        ChangeType::DatabaseMigration => true,
        ChangeType::APIBreakingChange => true,
        _ => false,
    }
}

// Require rationale before commit
if is_critical_change(&item.change) && item.rationale.is_none() {
    return Err("Critical changes require documented rationale");
}
```

### Property 4: Validated Propagation

**Guarantee:** Changes only propagate if all affected dashboards validate successfully.

**Implementation:**
```
Architecture change affects Security + Deployment dashboards
    ↓
Propagation Engine checks:
    ├─ Security Dashboard: Auth config updated? ✓
    ├─ Security Dashboard: Encryption enabled? ✗ (FAIL)
    └─ Deployment Dashboard: (not checked, Security failed)
    ↓
The Bin shows:
    "❌ Propagation blocked: Security validation failed
     Missing: TLS encryption configuration"
    ↓
User fixes Security config
    ↓
Re-validates:
    ├─ Security Dashboard: All checks pass ✓
    └─ Deployment Dashboard: CI/CD updated ✓
    ↓
Propagation proceeds
```

---

## Real-World Workflow Example

### Scenario: User changes database from PostgreSQL to TiDB

**Step-by-Step Through 6 Layers:**

```
1. User updates Tech Stack Dashboard
   - Selects "TiDB" in database dropdown
   - Change queued in The Bin (status: Pending)

2. Layer 1 (REALITY) Validation
   ✓ No runtime errors (change not yet applied)
   
3. Layer 2 (SPECIFICATION) Validation
   ✓ project.yaml syntax valid
   ✓ TiDB connection string format correct
   
4. Layer 3 (DOCUMENTATION) Validation
   ✓ README updated with TiDB setup instructions
   ✓ Architecture diagram regenerated
   
5. Layer 4 (DECISIONS) Validation
   ❌ FAIL: Rationale missing
   
   The Bin shows:
   "Why switching from PostgreSQL to TiDB?
    Please document rationale and alternatives."
   
   User provides:
   "Rationale: Need HTAP for real-time analytics.
    Alternatives considered: PostgreSQL + ClickHouse (rejected: complexity)."
   
   ✓ Rationale recorded
   
6. Layer 5 (VALIDATION) Validation
   - Dependency Graph: Affects Deployment Dashboard (Docker config)
   - AI Validation: "TiDB requires port 4000, ensure firewall rules updated"
   
   ✓ Deployment Dashboard validates:
     - Docker Compose updated with TiDB service
     - Firewall rules added
   
7. Layer 6 (MERGE GATE) - The Bin executes atomic commit:
   
   BEGIN TRANSACTION;
   
   -- Update project spec
   UPDATE projects SET database_type = 'tidb' WHERE id = 'proj-123';
   
   -- Update deployment config
   UPDATE deployment_configs 
   SET config_yaml = 'tidb-config' 
   WHERE project_id = 'proj-123';
   
   -- Git commit
   git add project.yaml README.md docker-compose.yml
   git commit -m "chore: migrate database from PostgreSQL to TiDB
   
   Rationale: HTAP capability for real-time analytics
   Alternatives: PostgreSQL + ClickHouse (too complex)
   
   Affects: Deployment configuration, CI/CD pipeline"
   
   -- Create rollback snapshot
   INSERT INTO version_snapshots (project_id, snapshot_data)
   VALUES ('proj-123', '<serialized_state>');
   
   COMMIT;  -- Atomic across all changes
   
   -- Notify dashboards
   SSE → All dashboards refresh UI
```

**Result:**
- Database change propagated to all affected areas
- Documentation updated automatically
- Decision recorded for future reference
- Rollback available for 30 days
- Zero possibility of desynchronization

---

## Architecture Diagrams

### Information Flow Through 6 Layers

```
┌─────────────────────────────────────────────────────────┐
│                    User Action                          │
│         (Edit dashboard, make decision)                 │
└──────────────────────┬──────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────┐
│  Layer 1: REALITY Validation                             │
│  • Check runtime logs for errors                         │
│  • Verify system health metrics                          │
└──────────────────────┬───────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────┐
│  Layer 2: SPECIFICATION Validation                       │
│  • Validate config file syntax                           │
│  • Check schema compliance                               │
└──────────────────────┬───────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────┐
│  Layer 3: DOCUMENTATION Validation                       │
│  • Attempt to generate docs                              │
│  • Verify all required files present                     │
└──────────────────────┬───────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────┐
│  Layer 4: DECISIONS Validation                           │
│  • Require rationale for critical changes                │
│  • Check alternatives_considered populated               │
└──────────────────────┬───────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────┐
│  Layer 5: VALIDATION (Propagation + AI)                  │
│  • Dependency graph checks (Rust WASM)                   │
│  • AI-powered guardrail validation                       │
│  • Cross-dashboard impact analysis                       │
└──────────────────────┬───────────────────────────────────┘
                       ↓
                ALL LAYERS PASS?
                ┌────────┴────────┐
               YES              NO
                │                │
                ↓                ↓
      ┌──────────────────┐  ┌─────────────────┐
      │ Mark as "Ready"  │  │  Block with     │
      │ Show impact      │  │  Violations     │
      │ Await user       │  │  Show fixes     │
      │ approval         │  │  Cannot commit  │
      └────────┬─────────┘  └─────────────────┘
               ↓
┌──────────────────────────────────────────────────────────┐
│  Layer 6: MERGE GATE (The Bin)                           │
│  • Atomic commit to TiDB (all tables)                    │
│  • Git commit (code + config + docs together)            │
│  • Propagate to downstream dashboards                    │
│  • Create rollback snapshot                              │
│  • Broadcast SSE update                                  │
└──────────────────────────────────────────────────────────┘
               ↓
┌──────────────────────────────────────────────────────────┐
│           All Dashboards Synchronized                    │
│  (Structurally impossible to be out of sync)             │
└──────────────────────────────────────────────────────────┘
```

---

## Testing & Verification

### Integration Test: 6-Layer Propagation

```rust
#[tokio::test]
async fn test_six_layer_propagation() {
    let mut project = create_test_project().await;
    
    // User changes architecture
    let change = ArchitectureChange {
        change_type: ChangeType::AddMicroservice,
        details: "Add payment-service",
    };
    
    // Submit to The Bin
    let bin_item_id = bin.submit_change(change).await.unwrap();
    
    // Layer 1: Runtime check (passes - no code running yet)
    assert!(bin.validate_layer1(bin_item_id).await.is_ok());
    
    // Layer 2: Spec check (fails - no config)
    assert!(bin.validate_layer2(bin_item_id).await.is_err());
    
    // User adds service config
    project.add_service_config("payment-service").await;
    
    // Layer 2: Now passes
    assert!(bin.validate_layer2(bin_item_id).await.is_ok());
    
    // Layer 3: Docs generation (fails - template missing)
    assert!(bin.validate_layer3(bin_item_id).await.is_err());
    
    // User adds docs template
    project.add_docs_template("payment-service").await;
    
    // Layer 3: Now passes
    assert!(bin.validate_layer3(bin_item_id).await.is_ok());
    
    // Layer 4: Rationale (fails - not provided)
    assert!(bin.validate_layer4(bin_item_id).await.is_err());
    
    // User provides rationale
    bin.add_rationale(bin_item_id, "Need separate payment processing for PCI compliance").await;
    
    // Layer 4: Now passes
    assert!(bin.validate_layer4(bin_item_id).await.is_ok());
    
    // Layer 5: Propagation check (identifies affected dashboards)
    let impact = bin.validate_layer5(bin_item_id).await.unwrap();
    assert_eq!(impact.affected_dashboards, vec!["security-001", "deployment-001"]);
    
    // Layer 6: Execute commit
    let result = bin.execute_commit(bin_item_id).await;
    assert!(result.is_ok());
    
    // Verify all layers synchronized
    let git_log = project.git_log().await;
    assert!(git_log.contains("payment-service"));
    
    let docs = project.read_docs("API_REFERENCE.md").await;
    assert!(docs.contains("payment-service"));
    
    let db_state = project.db_state("services").await;
    assert!(db_state.contains("payment-service"));
}
```

---

## Performance Considerations

### Validation Latency Budget

| Layer | Target Latency | Timeout |
|-------|---------------|---------|
| Layer 1 (Runtime) | < 50ms | 500ms |
| Layer 2 (Spec) | < 100ms | 1s |
| Layer 3 (Docs) | < 2s | 10s |
| Layer 4 (Decisions) | < 10ms | 100ms |
| Layer 5 (Validation) | < 3s | 30s |
| Layer 6 (Commit) | < 5s | 60s |

**Total Pipeline:** < 10 seconds for typical change

### Optimization Strategies

**Parallel Validation:**
```rust
// Run Layers 1-5 in parallel, aggregate results
let (l1, l2, l3, l4, l5) = tokio::join!(
    validate_layer1(item),
    validate_layer2(item),
    validate_layer3(item),
    validate_layer4(item),
    validate_layer5(item),
);

let all_pass = l1.is_ok() && l2.is_ok() && l3.is_ok() && l4.is_ok() && l5.is_ok();
```

**Incremental Validation:**
- Cache validation results per layer
- Only re-validate if inputs changed
- Use content-addressable storage (SHA-256 hashing)

---

## Open Questions

1. Should Layer 1 validation poll runtime logs or use event streaming?
2. Maximum retry count for Layer 3 docs generation? (Suggest: 3)
3. Should Layer 4 rationale be AI-validated for quality? (Suggest: Yes, post-MVP)
4. Can users skip Layer 5 AI validation if over token budget? (Suggest: No, use cached responses)

---

**End of KNOWLEDGE_SYNC_INTEGRATION_MAP.md**
