# DevGuide Cockpit: Migration System Specification

**Document Type:** Technical Implementation (Tier 5)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** THE_BIN_SPECIFICATION.md, DATA_MODEL.md, CROSS_CUTTING_CONCERNS.md

---

## Executive Summary

This artifact specifies the **version upgrade and migration system** for DevGuide Cockpit. When dashboard templates or system components are updated with breaking changes, users are guided through migration via an interactive wizard.

**Key Features:**
- **Breaking Change Detection**: Elm compiler detects incompatible changes
- **Migration Wizard**: Step-by-step UI guiding users through upgrades
- **Side-by-Side Diff**: Visual comparison of old vs new configs
- **Rollback Support**: 30-day rollback window to previous version
- **Automatic Migration**: AI-assisted config transformation when possible

---

## Migration Scenarios

### Scenario 1: Dashboard Template Update (Breaking Change)

**Example:**
```
Vibe Coding Dashboard v1.0 → v2.0

Breaking Change:
  v1.0: code_review_tool = "continue.dev"
  v2.0: code_review_tool = { provider: "local", model: "llama-3" }
```

**User Impact:** Existing Vibe Coding projects cannot load v2.0 without migration.

### Scenario 2: System-Wide API Change

**Example:**
```
The Bin API v1 → v2

Breaking Change:
  v1: POST /api/bin/commit { changes: [...] }
  v2: POST /api/bin/commit { batch: { items: [...], metadata: {...} } }
```

**User Impact:** All dashboards making Bin API calls must update.

### Scenario 3: Guardrail Rule Change

**Example:**
```
Security Guardrail v1 → v2

Breaking Change:
  v1: min_password_length = 8
  v2: password_policy = { min_length: 12, require_special: true }
```

**User Impact:** Existing security policies must be migrated.

---

## Breaking Change Detection

### Elm Compiler Detection

**Mechanism:** Elm's type system detects breaking changes at compile time.

```elm
-- Dashboard Template v1.0
type alias CodeReviewConfig =
    { tool : String  -- "continue.dev" or "coderabbit"
    }

-- Dashboard Template v2.0
type alias CodeReviewConfig =
    { provider : String  -- "local" or "remote"
    , model : String     -- "llama-3" or "gpt-4"
    }

-- Elm compiler error when loading v1.0 config:
-- "Type mismatch: expecting CodeReviewConfig with provider and model"
```

**Detection Flow:**
```
1. User updates template to v2.0
   ↓
2. Elm compiler attempts to compile dashboard
   ↓
3. Type mismatch detected
   ↓
4. System checks if migration path exists
   ↓
5. If yes: Launch Migration Wizard
   If no: Show "manual migration required" error
```

### Automated Detection Algorithm

```rust
// migration/detector.rs
pub fn detect_breaking_changes(
    old_schema: &DashboardSchema,
    new_schema: &DashboardSchema
) -> Vec<BreakingChange> {
    let mut changes = Vec::new();
    
    // Check for removed fields
    for field in &old_schema.fields {
        if !new_schema.fields.contains(field) {
            changes.push(BreakingChange::FieldRemoved {
                field_name: field.name.clone(),
                old_type: field.field_type.clone(),
            });
        }
    }
    
    // Check for changed field types
    for old_field in &old_schema.fields {
        if let Some(new_field) = new_schema.fields.iter().find(|f| f.name == old_field.name) {
            if old_field.field_type != new_field.field_type {
                changes.push(BreakingChange::FieldTypeChanged {
                    field_name: old_field.name.clone(),
                    old_type: old_field.field_type.clone(),
                    new_type: new_field.field_type.clone(),
                });
            }
        }
    }
    
    // Check for added required fields (breaking if no default)
    for new_field in &new_schema.fields {
        if !new_field.is_optional && 
           !old_schema.fields.iter().any(|f| f.name == new_field.name) {
            changes.push(BreakingChange::RequiredFieldAdded {
                field_name: new_field.name.clone(),
                new_type: new_field.field_type.clone(),
            });
        }
    }
    
    changes
}
```

---

## Migration Wizard UI/UX

### Step 1: Detection & Notification

**UI:**
```
┌─────────────────────────────────────────────────────┐
│ 🔔 Migration Required                               │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Vibe Coding Dashboard v1.0 → v2.0                  │
│                                                     │
│ This update includes breaking changes that require │
│ your attention.                                     │
│                                                     │
│ Breaking Changes Detected:                          │
│ • code_review_tool format changed                   │
│ • New required field: ai_model                      │
│                                                     │
│ [View Changes]  [Start Migration Wizard]            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Step 2: Side-by-Side Diff

**UI:**
```
┌─────────────────────────────────────────────────────┐
│ Migration: Code Review Configuration                │
├──────────────────────┬──────────────────────────────┤
│ Current (v1.0)       │ New (v2.0)                   │
├──────────────────────┼──────────────────────────────┤
│ code_review_tool:    │ code_review_tool:            │
│   "continue.dev"     │   provider: "local"          │
│                      │   model: "llama-3"           │
│                      │                              │
│                      │ NEW FIELD:                   │
│                      │ ai_model:                    │
│                      │   "claude-sonnet-4"          │
├──────────────────────┴──────────────────────────────┤
│                                                     │
│ Recommended Mapping:                                │
│ • provider = "local" (based on your setup)          │
│ • model = "llama-3" (default for local provider)    │
│ • ai_model = "claude-sonnet-4" (system default)     │
│                                                     │
│ [< Back]  [Customize Mapping]  [Apply & Continue >] │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Step 3: AI-Assisted Transformation

**UI:**
```
┌─────────────────────────────────────────────────────┐
│ 🤖 AI-Assisted Migration                            │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Claude is analyzing your current configuration...   │
│                                                     │
│ [████████████████████────] 80%                      │
│                                                     │
│ Suggested Migration:                                │
│                                                     │
│ code_review_tool:                                   │
│   provider: "local"       ← Detected: Continue.dev  │
│   model: "llama-3"        ← Recommended for local   │
│                                                     │
│ ai_model: "claude-sonnet-4"  ← Your default model   │
│                                                     │
│ Confidence: 95% ✅                                   │
│                                                     │
│ [Edit Manually]  [Accept Suggestion]                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Step 4: Manual Adjustment (Optional)

**UI:**
```
┌─────────────────────────────────────────────────────┐
│ Customize Migration                                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│ provider: [local ▼]                                 │
│           ├─ local                                  │
│           ├─ remote                                 │
│           └─ hybrid                                 │
│                                                     │
│ model: [llama-3 ▼]                                  │
│        ├─ llama-3                                   │
│        ├─ llama-3-70b                               │
│        ├─ codellama-34b                             │
│        └─ custom...                                 │
│                                                     │
│ ai_model: [claude-sonnet-4 ▼]                       │
│                                                     │
│ [< Back]  [Reset to Suggested]  [Apply Changes >]   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Step 5: Preview & Test

**UI:**
```
┌─────────────────────────────────────────────────────┐
│ Preview Migration                                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│ New Configuration:                                  │
│                                                     │
│ code_review_tool:                                   │
│   provider: "local"                                 │
│   model: "llama-3"                                  │
│                                                     │
│ ai_model: "claude-sonnet-4"                         │
│                                                     │
│ ✅ Elm Compiler: No errors                          │
│ ✅ Guardrail Check: Passed                          │
│ ✅ The Bin Validation: Passed                       │
│                                                     │
│ [< Back]  [Test Dashboard]  [Commit Migration]      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Step 6: Rollback Option

**UI:**
```
┌─────────────────────────────────────────────────────┐
│ Migration Committed ✅                               │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Your dashboard has been successfully migrated to    │
│ v2.0.                                               │
│                                                     │
│ Rollback Window: 30 days                            │
│                                                     │
│ If you encounter issues, you can roll back to v1.0  │
│ until February 1, 2026.                             │
│                                                     │
│ [Close]  [🔙 Rollback to v1.0]                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Migration Transformation Engine

### AI-Assisted Transformation

```rust
// migration/ai_transformer.rs
use crate::ai::AIProvider;

pub async fn transform_config_with_ai(
    old_config: &serde_json::Value,
    breaking_changes: &[BreakingChange],
    ai_provider: &dyn AIProvider
) -> Result<serde_json::Value, MigrationError> {
    let prompt = format!(r#"
        You are assisting with a configuration migration.
        
        Old Configuration:
        {}
        
        Breaking Changes:
        {}
        
        Transform the old configuration to the new format. Return only valid JSON.
    "#, 
        serde_json::to_string_pretty(old_config)?,
        format_breaking_changes(breaking_changes)
    );
    
    let response = ai_provider.complete(&prompt, 2000).await?;
    
    // Parse AI response as JSON
    let new_config: serde_json::Value = serde_json::from_str(&response.text)?;
    
    Ok(new_config)
}
```

### Rule-Based Transformation

```rust
// migration/rule_transformer.rs
pub fn transform_config_with_rules(
    old_config: &serde_json::Value,
    transformation_rules: &[TransformationRule]
) -> Result<serde_json::Value, MigrationError> {
    let mut new_config = old_config.clone();
    
    for rule in transformation_rules {
        match rule {
            TransformationRule::RenameField { old_name, new_name } => {
                if let Some(value) = new_config.get(old_name) {
                    new_config[new_name] = value.clone();
                    new_config.as_object_mut().unwrap().remove(old_name);
                }
            }
            
            TransformationRule::SplitField { old_name, new_fields } => {
                if let Some(value) = new_config.get(old_name) {
                    for (new_field, extractor) in new_fields {
                        new_config[new_field] = extractor(value)?;
                    }
                    new_config.as_object_mut().unwrap().remove(old_name);
                }
            }
            
            TransformationRule::AddDefaultField { name, default_value } => {
                if !new_config.get(name).is_some() {
                    new_config[name] = default_value.clone();
                }
            }
        }
    }
    
    Ok(new_config)
}
```

---

## Rollback Mechanism

### Version Snapshot Storage

**TiDB Schema:**
```sql
CREATE TABLE version_snapshots (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    dashboard_id VARCHAR(36) NOT NULL,
    version VARCHAR(20) NOT NULL,  -- "v1.0", "v2.0", etc.
    snapshot_data JSON NOT NULL,   -- Full dashboard config
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,           -- 30 days from created_at
    
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project_dashboard (project_id, dashboard_id),
    INDEX idx_expires (expires_at)
);
```

### Snapshot Creation

```rust
// migration/snapshot.rs
pub async fn create_snapshot(
    db: &TiDBConnection,
    project_id: &str,
    dashboard_id: &str,
    version: &str,
    config: &serde_json::Value
) -> Result<String, SnapshotError> {
    let snapshot_id = uuid::Uuid::new_v4().to_string();
    let expires_at = chrono::Utc::now() + chrono::Duration::days(30);
    
    db.execute(
        r#"
        INSERT INTO version_snapshots 
        (id, project_id, dashboard_id, version, snapshot_data, created_at, expires_at)
        VALUES (?, ?, ?, ?, ?, NOW(), ?)
        "#,
        &[
            &snapshot_id,
            project_id,
            dashboard_id,
            version,
            &serde_json::to_string(config)?,
            &expires_at
        ]
    ).await?;
    
    Ok(snapshot_id)
}
```

### Rollback Execution

```rust
// migration/rollback.rs
pub async fn rollback_to_snapshot(
    db: &TiDBConnection,
    project_id: &str,
    dashboard_id: &str,
    snapshot_id: &str
) -> Result<(), RollbackError> {
    // Retrieve snapshot
    let snapshot = db.query_one(
        "SELECT snapshot_data FROM version_snapshots WHERE id = ?",
        &[snapshot_id]
    ).await?;
    
    let old_config: serde_json::Value = serde_json::from_str(&snapshot.snapshot_data)?;
    
    // Create snapshot of current state before rollback
    let current_config = get_current_dashboard_config(db, project_id, dashboard_id).await?;
    create_snapshot(db, project_id, dashboard_id, "pre-rollback", &current_config).await?;
    
    // Apply rollback
    db.execute(
        r#"
        UPDATE project_dashboards 
        SET config_overrides = ?
        WHERE project_id = ? AND id = ?
        "#,
        &[
            &serde_json::to_string(&old_config)?,
            project_id,
            dashboard_id
        ]
    ).await?;
    
    // Notify user
    send_rollback_notification(project_id, dashboard_id).await?;
    
    Ok(())
}
```

---

## Elm Compiler Integration

### Compile-Time Validation

```elm
-- migration/compiler_integration.elm

-- Define migration interface
type alias MigrationPath =
    { fromVersion : String
    , toVersion : String
    , transform : Config -> Result Error Config
    }

-- Example migration path
vibeCodeV1ToV2 : MigrationPath
vibeCodeV1ToV2 =
    { fromVersion = "1.0"
    , toVersion = "2.0"
    , transform = \oldConfig ->
        case oldConfig.codeReviewTool of
            "continue.dev" ->
                Ok { oldConfig
                   | codeReviewTool = 
                       { provider = "local"
                       , model = "llama-3"
                       }
                   , aiModel = "claude-sonnet-4"
                   }
            
            _ ->
                Err "Unknown code review tool"
    }

-- Compiler will enforce type safety
applyMigration : MigrationPath -> Config -> Result Error Config
applyMigration path oldConfig =
    path.transform oldConfig
    |> Result.andThen validateConfig
```

### Compiler Error Messages

```
Elm Compiler Error:

  I found a type mismatch:

  78│     config.codeReviewTool
          ^^^^^^^^^^^^^^^^^^^^^^
  The type annotation says this is:

      String

  But I am inferring that it is:

      { provider : String, model : String }

  Hint: It looks like you're using an old dashboard configuration.
        Run the Migration Wizard to update to v2.0.
```

---

## Testing Strategy

### Migration Test Suite

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_detect_breaking_changes() {
        let old_schema = DashboardSchema {
            fields: vec![
                Field { name: "code_review_tool", field_type: "String", is_optional: false },
            ]
        };
        
        let new_schema = DashboardSchema {
            fields: vec![
                Field { name: "code_review_tool", field_type: "Object", is_optional: false },
                Field { name: "ai_model", field_type: "String", is_optional: false },
            ]
        };
        
        let changes = detect_breaking_changes(&old_schema, &new_schema);
        
        assert_eq!(changes.len(), 2);
        assert!(matches!(changes[0], BreakingChange::FieldTypeChanged { .. }));
        assert!(matches!(changes[1], BreakingChange::RequiredFieldAdded { .. }));
    }
    
    #[test]
    fn test_ai_assisted_transformation() {
        // Mock AI provider
        let mock_ai = MockAIProvider::new();
        mock_ai.set_response(r#"{
            "code_review_tool": {
                "provider": "local",
                "model": "llama-3"
            },
            "ai_model": "claude-sonnet-4"
        }"#);
        
        let old_config = json!({
            "code_review_tool": "continue.dev"
        });
        
        let new_config = transform_config_with_ai(
            &old_config,
            &[],
            &mock_ai
        ).await.unwrap();
        
        assert_eq!(new_config["code_review_tool"]["provider"], "local");
    }
    
    #[test]
    fn test_rollback_creates_snapshot() {
        let db = setup_test_db();
        
        let current_config = json!({ "version": "2.0" });
        create_snapshot(&db, "proj-1", "dash-1", "v2.0", &current_config).await.unwrap();
        
        // Rollback to v1.0
        rollback_to_snapshot(&db, "proj-1", "dash-1", "snap-v1").await.unwrap();
        
        // Verify pre-rollback snapshot created
        let snapshots = db.query(
            "SELECT * FROM version_snapshots WHERE version = 'pre-rollback'"
        ).await.unwrap();
        
        assert_eq!(snapshots.len(), 1);
    }
}
```

### E2E Migration Test

```typescript
// test/migration-e2e.spec.ts
test('user completes migration wizard', async ({ page }) => {
    // Navigate to project with outdated dashboard
    await page.goto('/projects/my-project');
    
    // Migration notification should appear
    await expect(page.locator('[data-testid="migration-notification"]')).toBeVisible();
    
    // Start migration wizard
    await page.click('[data-testid="start-migration"]');
    
    // View changes
    await expect(page.locator('[data-testid="diff-viewer"]')).toBeVisible();
    
    // AI suggests transformation
    await expect(page.locator('[data-testid="ai-suggestion"]')).toContainText('provider: "local"');
    
    // Accept suggestion
    await page.click('[data-testid="accept-suggestion"]');
    
    // Preview migration
    await expect(page.locator('[data-testid="elm-compiler-status"]')).toContainText('No errors');
    
    // Commit migration
    await page.click('[data-testid="commit-migration"]');
    
    // Verify success
    await expect(page.locator('[data-testid="migration-success"]')).toBeVisible();
    await expect(page.locator('[data-testid="rollback-option"]')).toContainText('30 days');
});
```

---

## Error Handling

### Migration Failures

```rust
#[derive(Debug)]
pub enum MigrationError {
    // User errors
    InvalidConfig(String),
    RequiredFieldMissing(String),
    
    // System errors
    CompilerError(String),
    DatabaseError(String),
    AIProviderUnavailable,
    
    // Validation errors
    GuardrailViolation(String),
    BinRejection(String),
}

impl Display for MigrationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            MigrationError::InvalidConfig(msg) => 
                write!(f, "Invalid configuration: {}. Please review your changes.", msg),
            
            MigrationError::AIProviderUnavailable => 
                write!(f, "AI assistance unavailable. You can still complete migration manually."),
            
            MigrationError::GuardrailViolation(msg) => 
                write!(f, "Migration violates guardrail: {}. Please adjust configuration.", msg),
            
            _ => write!(f, "{:?}", self)
        }
    }
}
```

---

## Open Questions

1. **Migration Complexity**: Should complex migrations (> 5 breaking changes) require manual review?
2. **AI Confidence Threshold**: What confidence level (%) should trigger manual review?
3. **Rollback Expiration**: 30 days sufficient, or should high-impact projects get longer?
4. **Batch Migrations**: Support migrating all dashboards in a project at once?

---

**End of ARTIFACT_18_MIGRATION_SYSTEM_SPECIFICATION.md**
