# Propagation Engine Specification

**Document Type:** Core Mechanics  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** THE_BIN_SPECIFICATION.md, DATA_MODEL_SPECIFICATION.md

---

## Overview

The **Propagation Engine** ensures changes to one artifact automatically update all dependent artifacts while maintaining consistency and preventing desynchronization.

**Core Guarantee:** When user accepts a change in The Bin, ALL affected artifacts update atomically.

---

## Propagation Modes

### Mode 1: Auto-Propagate (Opt-In)

**Behavior:** Changes flow immediately without user review  
**Use Case:** Low-risk changes (formatting, typos)  
**Configuration:** Per-project setting

**Risk:** Silent accumulation of token costs

### Mode 2: Gated Propagation (DEFAULT)

**Behavior:** Changes flagged for review before propagation  
**Use Case:** General development workflow  
**Configuration:** System default

**Benefit:** User controls when/how changes propagate

### Mode 3: Hybrid (Impact-Based)

**Behavior:** Auto for low-impact, gated for high-impact  
**Thresholds:**
- Low impact: <5K tokens, 1 dashboard
- Medium impact: 5K-10K tokens, 2-3 dashboards
- High impact: >10K tokens, 4+ dashboards

**Configuration:** Per-dashboard granularity

---

## Dependency Graph

**Graph Structure:**

```rust
pub struct DependencyGraph {
    nodes: HashMap<NodeId, Node>,
    edges: HashMap<NodeId, Vec<Edge>>,
}

pub struct Node {
    id: NodeId,
    node_type: NodeType,  // Dashboard, Module, File
    metadata: NodeMetadata,
}

pub struct Edge {
    from: NodeId,
    to: NodeId,
    edge_type: EdgeType,  // DataFlow, ControlFlow, Capability
    weight: f32,           // Propagation cost estimate
}

pub enum NodeType {
    Dashboard(DashboardId),
    Module(ModuleId),
    File(PathBuf),
    GitCommit(String),
}
```

**Example Graph:**

```
API Spec (File)
  ├─> Backend Implementation (Module)
  │   ├─> Unit Tests (Module)
  │   └─> API Docs (File)
  └─> Frontend Client (Module)
      ├─> UI Components (Module)
      └─> Integration Tests (Module)
```

**Query:** "If I change API Spec, what's affected?"

```rust
pub fn find_dependents(graph: &DependencyGraph, node_id: &NodeId) -> Vec<NodeId> {
    let mut visited = HashSet::new();
    let mut stack = vec![node_id.clone()];
    let mut result = Vec::new();
    
    while let Some(current) = stack.pop() {
        if visited.contains(&current) {
            continue;
        }
        visited.insert(current.clone());
        
        if let Some(edges) = graph.edges.get(&current) {
            for edge in edges {
                stack.push(edge.to.clone());
                result.push(edge.to.clone());
            }
        }
    }
    
    result
}
```

---

## Propagation Plan

**Plan Structure:**

```rust
pub struct PropagationPlan {
    source_change: ChangeId,
    steps: Vec<PropagationStep>,
    estimated_duration: Duration,
    estimated_tokens: u32,
    estimated_file_writes: u32,
}

pub struct PropagationStep {
    order: usize,
    target: NodeId,
    action: PropagationAction,
    dependencies: Vec<usize>,  // Step indices that must complete first
}

pub enum PropagationAction {
    RegenerateWithAI { prompt: String, budget: TokenBudget },
    UpdateFile { path: PathBuf, patch: String },
    ExecuteTest { test_path: PathBuf },
    CommitToGit { message: String },
}
```

**Example Plan:**

```yaml
propagation_plan:
  source: "change_abc123"
  steps:
    - order: 1
      target: "backend_implementation"
      action: "RegenerateWithAI"
      prompt: "Update Go handlers to match new API spec"
      budget: 3000
      dependencies: []
    
    - order: 2
      target: "unit_tests"
      action: "RegenerateWithAI"
      prompt: "Update tests for modified handlers"
      budget: 2000
      dependencies: [1]
    
    - order: 3
      target: "api_docs"
      action: "UpdateFile"
      path: "docs/api.md"
      patch: "+++ Add new endpoint section"
      dependencies: [1]
    
    - order: 4
      target: "git_commit"
      action: "CommitToGit"
      message: "Update API spec and implementations"
      dependencies: [1, 2, 3]
```

---

## Execution Engine

**Parallel Execution:**

```rust
pub async fn execute_plan(plan: PropagationPlan) -> Result<PropagationResult, PropagationError> {
    let mut completed_steps: HashSet<usize> = HashSet::new();
    let mut results: HashMap<usize, StepResult> = HashMap::new();
    
    while completed_steps.len() < plan.steps.len() {
        // Find steps ready to execute (dependencies met)
        let ready_steps: Vec<&PropagationStep> = plan.steps.iter()
            .filter(|step| {
                !completed_steps.contains(&step.order) &&
                step.dependencies.iter().all(|dep| completed_steps.contains(dep))
            })
            .collect();
        
        // Execute ready steps in parallel
        let futures: Vec<_> = ready_steps.iter()
            .map(|step| execute_step(step))
            .collect();
        
        let step_results = futures::future::join_all(futures).await;
        
        for (step, result) in ready_steps.iter().zip(step_results) {
            match result {
                Ok(res) => {
                    completed_steps.insert(step.order);
                    results.insert(step.order, res);
                },
                Err(e) => {
                    return Err(PropagationError::StepFailed {
                        step: step.order,
                        error: e,
                    });
                }
            }
        }
    }
    
    Ok(PropagationResult { results })
}
```

**Step Execution:**

```rust
async fn execute_step(step: &PropagationStep) -> Result<StepResult, StepError> {
    let start = Instant::now();
    
    let outcome = match &step.action {
        PropagationAction::RegenerateWithAI { prompt, budget } => {
            let ai_response = ai_provider.reason(prompt.clone(), *budget).await?;
            StepOutcome::AIGenerated { content: ai_response.content }
        },
        
        PropagationAction::UpdateFile { path, patch } => {
            fs::write(path, patch)?;
            StepOutcome::FileUpdated { path: path.clone() }
        },
        
        PropagationAction::ExecuteTest { test_path } => {
            let test_result = run_tests(test_path).await?;
            StepOutcome::TestExecuted { passed: test_result.passed }
        },
        
        PropagationAction::CommitToGit { message } => {
            let commit_sha = git_commit(message)?;
            StepOutcome::Committed { sha: commit_sha }
        },
    };
    
    Ok(StepResult {
        step_order: step.order,
        duration: start.elapsed(),
        outcome,
    })
}
```

---

## Conflict Resolution

**Concurrent Changes:**

```
Scenario: User A changes API spec, User B changes backend implementation
Result: Conflict detected when both try to propagate

Resolution Strategy:
1. Last-write-wins (MVP - single user, no concurrency)
2. Three-way merge (Post-MVP)
3. User-assisted resolution (manual merge)
```

**Lock Mechanism:**

```rust
pub struct PropagationLock {
    project_id: String,
    locked_at: SystemTime,
    locked_by: UserId,
    timeout: Duration,
}

pub async fn acquire_lock(project_id: &str, user_id: &str) -> Result<PropagationLock, LockError> {
    let existing_lock = db.get_lock(project_id).await?;
    
    match existing_lock {
        Some(lock) if lock.locked_at.elapsed()? < lock.timeout => {
            Err(LockError::AlreadyLocked {
                by: lock.locked_by,
                until: lock.locked_at + lock.timeout,
            })
        },
        _ => {
            let new_lock = PropagationLock {
                project_id: project_id.to_string(),
                locked_at: SystemTime::now(),
                locked_by: user_id.to_string(),
                timeout: Duration::from_secs(300),  // 5 minutes
            };
            db.create_lock(&new_lock).await?;
            Ok(new_lock)
        }
    }
}
```

---

## Progress Reporting

**SSE Updates:**

```go
type PropagationProgress struct {
    ChangeID        string  `json:"change_id"`
    TotalSteps      int     `json:"total_steps"`
    CompletedSteps  int     `json:"completed_steps"`
    CurrentStep     string  `json:"current_step"`
    Progress        float64 `json:"progress"` // 0.0 - 1.0
}

func (s *Service) ReportProgress(ctx context.Context, progress PropagationProgress) {
    data, _ := json.Marshal(progress)
    s.sseHub.Broadcast(ctx, "propagation_progress", data)
}
```

**Elm UI:**

```elm
type alias PropagationProgress =
    { changeId : String
    , totalSteps : Int
    , completedSteps : Int
    , currentStep : String
    , progress : Float
    }

viewPropagationProgress : Maybe PropagationProgress -> Html Msg
viewPropagationProgress maybeProgress =
    case maybeProgress of
        Just progress ->
            div [ class "propagation-progress" ]
                [ h4 [] [ text "Propagating Changes..." ]
                , progressBar progress.progress
                , p [] [ text ("Step " ++ String.fromInt progress.completedSteps 
                              ++ " of " ++ String.fromInt progress.totalSteps
                              ++ ": " ++ progress.currentStep) ]
                ]
        
        Nothing ->
            text ""

progressBar : Float -> Html msg
progressBar value =
    div [ class "progress-bar" ]
        [ div 
            [ class "progress-fill"
            , style "width" (String.fromFloat (value * 100) ++ "%")
            ] 
            []
        ]
```

---

## Rollback on Failure

**Strategy:** If any step fails, rollback completed steps

```rust
pub async fn rollback_propagation(
    completed_steps: &[StepResult]
) -> Result<(), RollbackError> {
    for step_result in completed_steps.iter().rev() {
        match &step_result.outcome {
            StepOutcome::FileUpdated { path } => {
                git_restore(path)?;
            },
            StepOutcome::Committed { sha } => {
                git_revert(sha)?;
            },
            StepOutcome::AIGenerated { .. } => {
                // No rollback needed (not persisted yet)
            },
            StepOutcome::TestExecuted { .. } => {
                // No rollback needed (read-only operation)
            },
        }
    }
    Ok(())
}
```

---

## Cost Estimation

**Token Cost:**

```rust
pub fn estimate_tokens(action: &PropagationAction) -> u32 {
    match action {
        PropagationAction::RegenerateWithAI { budget, .. } => budget.limit,
        PropagationAction::UpdateFile { patch, .. } => {
            // Estimate based on patch size
            (patch.len() / 4) as u32  // Rough token estimate
        },
        _ => 0,
    }
}
```

**Time Estimation:**

```rust
pub fn estimate_duration(step: &PropagationStep) -> Duration {
    match &step.action {
        PropagationAction::RegenerateWithAI { .. } => Duration::from_secs(3),
        PropagationAction::UpdateFile { .. } => Duration::from_millis(100),
        PropagationAction::ExecuteTest { .. } => Duration::from_secs(5),
        PropagationAction::CommitToGit { .. } => Duration::from_millis(500),
    }
}
```

---

## Performance Metrics

| Metric | Target | Max | Monitoring |
|--------|--------|-----|------------|
| Plan Generation | < 100ms | 300ms | Prometheus histogram |
| Step Execution (avg) | < 2s | 5s | Prometheus histogram |
| Full Propagation | < 10s | 30s | Prometheus histogram |
| Rollback | < 5s | 10s | Prometheus histogram |

---

**End of PROPAGATION_ENGINE.md**
