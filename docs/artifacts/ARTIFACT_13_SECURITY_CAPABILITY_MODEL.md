# DevGuide Cockpit: Security Capability Model

**Document Type:** Governance & Security (Tier 4)  
**Status:** Active  
**Version:** 1.1.0  
**Date:** 2026-01-02T00:00:00Z  
**Dependencies:** ARTIFACT_03_THE_BIN_SPECIFICATION.md, ARTIFACT_10_TEMPLATE_SYSTEM_SPECIFICATION.md, ARTIFACT_05_DATA_MODEL.md, ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md

---

## Changelog

### v1.1.0 (2026-01-02T00:00:00Z)
- Added Category 6: Temporal Security Capabilities (CAP-SEC-36 through CAP-SEC-40)
- Added Category 7: Metadata Governance Capabilities
- Updated capability enumeration to 35+ capabilities across 7 categories
- Added clock_sync_status and metadata_standards table integrations
- Added auditd monitoring for timestomping detection

### v1.0.0 (2026-01-01T00:00:00Z)
- Initial release with 30+ capabilities across 5 categories

---

## Executive Summary

This artifact specifies the **complete capability and permission system** for DevGuide Cockpit. It defines:
- Complete capability enumeration (35+ capabilities across 7 categories)
- Elm capability declaration patterns
- Three-layer enforcement (compile-time, runtime, transaction)
- JavaScript/Port sandboxing with manifests
- Web component permission system
- Audit trail for capability usage
- **Temporal security for clock synchronization and timestamp integrity** (NEW in v1.1.0)
- **Metadata governance enforcement capabilities** (NEW in v1.1.0)

**Security Philosophy:** Fine-grained capabilities > coarse-grained roles

---

## Complete Capability Enumeration

### Category 1: Data Access Capabilities

```elm
type DataCapability
    = ReadViolations          -- Access bin_violations table
    | ReadDecisions           -- Access project_decisions table
    | ReadProjectConfig       -- Access project.yaml
    | ReadAuditLog            -- Access project_change_log
    | ReadChatHistory         -- Access AI conversation history
    | ReadGuardrails          -- Access guardrail_rules table
    | ReadUserData            -- Access user preferences
    | ReadAllProjects         -- Admin: access all projects
    | ReadSensitiveData       -- PII, financial data
```

**Enforcement:** TiDB row-level security + Go API middleware

---

### Category 2: Modification Capabilities

```elm
type ModificationCapability
    = WriteViolations         -- Add items to The Bin
    | PropagateChanges        -- Trigger cross-dashboard updates
    | ModifyGuardrails        -- Override guardrail settings
    | DeleteProjects          -- Permanently delete projects
    | UnlockProject           -- Remove dashboard lock
    | ForceCommit             -- Bypass validation (admin only)
    | EditArchetype           -- Modify template
    | RollbackVersion         -- Restore old version
```

**Enforcement:** The Bin validation + transaction-level checks

---

### Category 3: Rendering Capabilities

```elm
type RenderingCapability
    = RenderCanvas            -- Display diagrams (D3.js, WebGL)
    | ExecuteJavaScript       -- Use Elm Ports
    | AccessDOM               -- Direct DOM manipulation
    | LoadExternalAssets      -- Fetch from CDNs
    | UseWebComponents        -- Load web components
    | RenderPDF               -- Generate PDF exports
    | StreamSSE               -- Server-Sent Events subscription
```

**Enforcement:** Elm runtime + browser sandbox + CSP

---

### Category 4: AI Capabilities

```elm
type AICapability
    = RequestAICompletion     -- Send prompts to AI Layer
    | ValidateWithAI          -- AI-powered guardrails
    | AccessAIContext         -- Read conversation history
    | ModifyAIBudget          -- Change token limits
    | ConfigureAIProvider     -- Switch AI provider
```

**Enforcement:** Go API token budget checks

---

### Category 5: External Integration Capabilities

```elm
type IntegrationCapability
    = GitCommit               -- Commit to repository
    | GitPush                 -- Push to remote
    | NetworkRequest          -- Make HTTP calls
    | FileSystemRead          -- Read local files
    | FileSystemWrite         -- Write local files
    | ExecuteShellCommand     -- Run OS commands
    | DatabaseQuery           -- Direct SQL queries
```

**Enforcement:** OS-level sandboxing + capability manifest

---

### Category 6: Temporal Security Capabilities (NEW in v1.1.0)

```elm
type TemporalSecurityCapability
    = CAP_SEC_36_ClockDriftDetection        -- Monitor NTP/PTP synchronization
    | CAP_SEC_37_TimestompingDetection      -- Detect utimensat syscall abuse
    | CAP_SEC_38_TimezoneInjectionPrevention -- Enforce Zulu-only timestamps
    | CAP_SEC_39_MetadataStandardsImmutability -- Protect metadata standards
    | CAP_SEC_40_ViolationSuppressionAudit  -- Audit all suppression actions
```

**Enforcement:** Rust WASM kernel + auditd + TiDB triggers

---

### Category 7: Metadata Governance Capabilities (NEW in v1.1.0)

```elm
type MetadataCapability
    = ReadMetadataStandards    -- Access metadata_standards table
    | WriteMetadataStandards   -- Create/modify standards (admin)
    | ReadMetadataViolations   -- View compliance issues
    | SuppressViolation        -- Mark violation as suppressed
    | ResolveViolation         -- Mark violation as resolved
    | AccessClockSyncStatus    -- Monitor clock_sync_status table
    | AccessFilesystemTimestamps -- Read filesystem_timestamps
```

**Enforcement:** metadata_standards.is_active flag + audit_logs

---

## Temporal Security Capabilities (Detailed)

### CAP-SEC-36: Clock Drift Detection

**Purpose:** Ensure all system components maintain synchronized clocks to prevent timestamp manipulation attacks and ensure audit trail integrity.

**Implementation:**

```rust
// security/src/clock_drift_detector.rs

use chrono::{DateTime, Utc};

pub struct ClockDriftDetector {
    db: TiDBClient,
    alert_threshold_ms: i64,
    quarantine_threshold_ms: i64,
}

impl ClockDriftDetector {
    pub fn new(db: TiDBClient) -> Self {
        Self {
            db,
            alert_threshold_ms: 10,      // Alert if drift > 10ms
            quarantine_threshold_ms: 100, // Quarantine if drift > 100ms
        }
    }

    pub async fn check_component(&self, component_name: &str) -> DriftStatus {
        let status = self.db.query_one::<ClockSyncStatus>(
            "SELECT * FROM clock_sync_status WHERE component_name = ?",
            &[component_name]
        ).await?;
        
        let drift_ms = status.offset_microseconds / 1000;
        
        if drift_ms.abs() > self.quarantine_threshold_ms {
            DriftStatus::Quarantine {
                component: component_name.to_string(),
                drift_ms,
                last_sync: status.last_sync_at,
            }
        } else if drift_ms.abs() > self.alert_threshold_ms {
            DriftStatus::Alert {
                component: component_name.to_string(),
                drift_ms,
                last_sync: status.last_sync_at,
            }
        } else {
            DriftStatus::Synchronized {
                component: component_name.to_string(),
                drift_ms,
            }
        }
    }

    pub async fn quarantine_component(&self, component_name: &str) -> Result<(), SecurityError> {
        // Update status to quarantined
        self.db.execute(
            "UPDATE clock_sync_status SET status = 'error' WHERE component_name = ?",
            &[component_name]
        ).await?;
        
        // Log security event
        self.db.execute(
            "INSERT INTO audit_logs (timestamp, user_id, action, resource_type, resource_id, status, metadata)
             VALUES (CURRENT_TIMESTAMP(6), 'SYSTEM', 'CLOCK_QUARANTINE', 'component', ?, 'success', ?)",
            &[component_name, json!({"reason": "drift_exceeded_threshold"})]
        ).await?;
        
        Ok(())
    }
}

#[derive(Debug)]
pub enum DriftStatus {
    Synchronized { component: String, drift_ms: i64 },
    Alert { component: String, drift_ms: i64, last_sync: DateTime<Utc> },
    Quarantine { component: String, drift_ms: i64, last_sync: DateTime<Utc> },
}
```

**Database Integration:**

```sql
-- Query: Check all components for clock drift
SELECT 
    component_name,
    protocol,
    stratum,
    offset_microseconds / 1000 AS drift_ms,
    last_sync_at,
    CASE 
        WHEN ABS(offset_microseconds) > 100000 THEN 'QUARANTINE'
        WHEN ABS(offset_microseconds) > 10000 THEN 'ALERT'
        ELSE 'OK'
    END AS drift_status
FROM clock_sync_status
WHERE status IN ('synchronized', 'drifting')
ORDER BY ABS(offset_microseconds) DESC;
```

**Thresholds:**

| Threshold | Value | Action |
|-----------|-------|--------|
| Normal | < 10ms | No action |
| Alert | 10ms - 100ms | Warning logged, notification sent |
| Quarantine | > 100ms | Component isolated, manual intervention required |

---

### CAP-SEC-37: Timestomping Detection

**Purpose:** Detect and prevent malicious modification of filesystem timestamps (atime, mtime, ctime) that could be used to hide unauthorized file changes.

**Implementation:**

```rust
// security/src/timestomping_detector.rs

use std::process::Command;

pub struct TimestompingDetector {
    db: TiDBClient,
    auditd_enabled: bool,
}

impl TimestompingDetector {
    /// Check if a file's timestamps have been manipulated
    pub async fn detect_timestomping(&self, file_path: &str) -> TimestompingResult {
        // Get recorded timestamps from database
        let recorded = self.db.query_one::<FilesystemTimestamp>(
            "SELECT * FROM filesystem_timestamps 
             WHERE file_path = ? 
             ORDER BY captured_at DESC LIMIT 1",
            &[file_path]
        ).await?;
        
        // Get current filesystem timestamps
        let current = self.get_current_timestamps(file_path)?;
        
        // Detection rule: ctime should never go backwards
        if current.ctime < recorded.ctime {
            return TimestompingResult::Detected {
                file_path: file_path.to_string(),
                reason: "ctime regression detected".to_string(),
                recorded_ctime: recorded.ctime,
                current_ctime: current.ctime,
            };
        }
        
        // Detection rule: mtime changed but ctime unchanged (impossible normally)
        if current.mtime != recorded.mtime && current.ctime == recorded.ctime {
            return TimestompingResult::Detected {
                file_path: file_path.to_string(),
                reason: "mtime changed without ctime update".to_string(),
                recorded_ctime: recorded.ctime,
                current_ctime: current.ctime,
            };
        }
        
        TimestompingResult::Clean
    }
    
    /// Parse auditd logs for utimensat syscalls
    pub async fn check_auditd_alerts(&self) -> Vec<AuditdAlert> {
        // Search audit logs for timestamp manipulation syscalls
        let output = Command::new("ausearch")
            .args(&["-sc", "utimensat", "-ts", "recent"])
            .output()
            .expect("Failed to execute ausearch");
        
        self.parse_auditd_output(&output.stdout)
    }
}

#[derive(Debug)]
pub enum TimestompingResult {
    Clean,
    Detected {
        file_path: String,
        reason: String,
        recorded_ctime: DateTime<Utc>,
        current_ctime: DateTime<Utc>,
    },
}
```

**auditd Configuration:**

```bash
# /etc/audit/rules.d/devguide-timestomping.rules

# Monitor utimensat syscall on project directories
-a always,exit -F arch=b64 -S utimensat -F dir=/opt/devguide -F key=timestomping
-a always,exit -F arch=b64 -S utimes -F dir=/opt/devguide -F key=timestomping
-a always,exit -F arch=b64 -S futimesat -F dir=/opt/devguide -F key=timestomping

# Alert on any timestamp modification attempts
-w /opt/devguide -p wa -k devguide_write
```

**Database Schema Integration:**

```sql
-- Query: Find files with suspicious timestamp patterns
SELECT 
    fs.file_path,
    fs.mtime AS recorded_mtime,
    fs.ctime AS recorded_ctime,
    fs.captured_at,
    fs.git_commit_sha
FROM filesystem_timestamps fs
WHERE EXISTS (
    SELECT 1 FROM filesystem_timestamps fs2
    WHERE fs2.file_path = fs.file_path
      AND fs2.captured_at > fs.captured_at
      AND fs2.ctime < fs.ctime  -- ctime regression = tampering
);
```

---

### CAP-SEC-38: Timezone Injection Prevention

**Purpose:** Prevent timezone manipulation attacks by enforcing strict Zulu-only timestamp parsing across all system inputs.

**Implementation:**

```rust
// security/src/timezone_injection_prevention.rs

use chrono::{DateTime, Utc, FixedOffset};
use regex::Regex;

pub struct TimezoneInjectionPrevention;

impl TimezoneInjectionPrevention {
    /// Validate that a timestamp is in strict Zulu format
    pub fn validate_zulu_only(timestamp: &str) -> Result<DateTime<Utc>, SecurityError> {
        // Strict regex: ONLY accept Z suffix, reject all offset formats
        let zulu_regex = Regex::new(
            r"^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?Z$"
        ).unwrap();
        
        if !zulu_regex.is_match(timestamp) {
            return Err(SecurityError::TimezoneInjection {
                value: timestamp.to_string(),
                reason: "Non-Zulu timestamp rejected".to_string(),
                suggestion: Self::suggest_zulu_conversion(timestamp),
            });
        }
        
        // Parse and validate
        DateTime::parse_from_rfc3339(timestamp)
            .map(|dt| dt.with_timezone(&Utc))
            .map_err(|e| SecurityError::InvalidTimestamp {
                value: timestamp.to_string(),
                parse_error: e.to_string(),
            })
    }
    
    /// Detect timezone injection attempts
    pub fn detect_injection_attempt(input: &str) -> Option<InjectionAttempt> {
        // Pattern 1: Offset injection (+XX:XX or -XX:XX)
        let offset_regex = Regex::new(r"[+-]\d{2}:\d{2}$").unwrap();
        if offset_regex.is_match(input) {
            return Some(InjectionAttempt::OffsetInjection {
                value: input.to_string(),
            });
        }
        
        // Pattern 2: Named timezone injection (e.g., "EST", "PST")
        let tz_names = ["EST", "PST", "CST", "MST", "EDT", "PDT", "GMT", "UTC"];
        for tz in &tz_names {
            if input.contains(tz) && !input.ends_with('Z') {
                return Some(InjectionAttempt::NamedTimezoneInjection {
                    value: input.to_string(),
                    detected_tz: tz.to_string(),
                });
            }
        }
        
        None
    }
    
    /// Convert any valid timestamp to Zulu format
    pub fn normalize_to_zulu(timestamp: &str) -> Result<String, SecurityError> {
        // Try parsing with offset
        if let Ok(dt) = DateTime::parse_from_rfc3339(timestamp) {
            return Ok(dt.with_timezone(&Utc).format("%Y-%m-%dT%H:%M:%S%.6fZ").to_string());
        }
        
        Err(SecurityError::CannotNormalize {
            value: timestamp.to_string(),
        })
    }
    
    fn suggest_zulu_conversion(timestamp: &str) -> String {
        match Self::normalize_to_zulu(timestamp) {
            Ok(zulu) => format!("Convert to: {}", zulu),
            Err(_) => "Unable to suggest conversion".to_string(),
        }
    }
}

#[derive(Debug)]
pub enum InjectionAttempt {
    OffsetInjection { value: String },
    NamedTimezoneInjection { value: String, detected_tz: String },
}
```

**Enforcement Points:**

| Layer | Check | Action on Failure |
|-------|-------|-------------------|
| API Gateway | All request timestamps | 400 Bad Request |
| The Bin | Submission validation | Block with SEC-038 violation |
| Database Trigger | BEFORE INSERT/UPDATE | Reject transaction |
| AI Layer | Response normalization | Auto-convert to Zulu |

**Database Trigger:**

```sql
-- Trigger: Enforce Zulu-only timestamps on insert
DELIMITER //
CREATE TRIGGER enforce_zulu_timestamp
BEFORE INSERT ON audit_logs
FOR EACH ROW
BEGIN
    -- Verify timestamp ends with microseconds and Z
    IF NEW.timestamp NOT REGEXP '^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}\\.[0-9]{6}Z$' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'CAP-SEC-38: Non-Zulu timestamp rejected';
    END IF;
END //
DELIMITER ;
```

---

### CAP-SEC-39: Metadata Standards Immutability

**Purpose:** Protect the integrity of metadata standards by enforcing append-only semantics and preventing unauthorized modifications.

**Implementation:**

```rust
// security/src/metadata_standards_immutability.rs

pub struct MetadataStandardsGuard {
    db: TiDBClient,
}

impl MetadataStandardsGuard {
    /// Enforce append-only semantics for metadata_standards table
    pub async fn validate_standard_change(
        &self,
        operation: StandardOperation,
        user_id: &str,
    ) -> Result<(), SecurityError> {
        match operation {
            StandardOperation::Create { standard } => {
                // CREATE allowed - log and proceed
                self.log_standard_change("CREATE", &standard.id, user_id).await?;
                Ok(())
            }
            StandardOperation::Update { standard_id, changes } => {
                // UPDATE restricted - only is_active can be changed
                if changes.keys().any(|k| k != "is_active") {
                    return Err(SecurityError::ImmutabilityViolation {
                        resource: "metadata_standards".to_string(),
                        attempted_change: format!("{:?}", changes.keys()),
                        reason: "Only is_active field can be modified".to_string(),
                    });
                }
                self.log_standard_change("DEACTIVATE", &standard_id, user_id).await?;
                Ok(())
            }
            StandardOperation::Delete { standard_id } => {
                // DELETE prohibited - use soft delete (is_active = false)
                Err(SecurityError::ImmutabilityViolation {
                    resource: "metadata_standards".to_string(),
                    attempted_change: format!("DELETE {}", standard_id),
                    reason: "Hard delete prohibited. Use is_active = false".to_string(),
                })
            }
        }
    }
    
    /// Log all standard changes for audit
    async fn log_standard_change(
        &self,
        action: &str,
        standard_id: &str,
        user_id: &str,
    ) -> Result<(), SecurityError> {
        self.db.execute(
            "INSERT INTO audit_logs 
             (timestamp, user_id, action, resource_type, resource_id, status, metadata)
             VALUES (CURRENT_TIMESTAMP(6), ?, ?, 'metadata_standard', ?, 'success', ?)",
            &[user_id, action, standard_id, json!({"cap": "CAP-SEC-39"})]
        ).await?;
        Ok(())
    }
}

#[derive(Debug)]
pub enum StandardOperation {
    Create { standard: MetadataStandard },
    Update { standard_id: String, changes: HashMap<String, String> },
    Delete { standard_id: String },
}
```

**Database Constraints:**

```sql
-- Prevent DELETE on metadata_standards (use soft delete)
CREATE TRIGGER prevent_metadata_standards_delete
BEFORE DELETE ON metadata_standards
FOR EACH ROW
BEGIN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'CAP-SEC-39: Hard delete prohibited. Set is_active = FALSE instead.';
END;

-- Prevent modification of core fields (only is_active allowed)
CREATE TRIGGER prevent_metadata_standards_update
BEFORE UPDATE ON metadata_standards
FOR EACH ROW
BEGIN
    IF OLD.standard_name != NEW.standard_name 
       OR OLD.category != NEW.category
       OR OLD.pattern_regex != NEW.pattern_regex
       OR OLD.validation_message != NEW.validation_message
       OR OLD.enforcement_level != NEW.enforcement_level
    THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'CAP-SEC-39: Only is_active field can be modified. Create a new standard version.';
    END IF;
END;
```

**Version Control Pattern:**

```sql
-- To "update" a standard: deactivate old, create new version
-- Step 1: Deactivate existing
UPDATE metadata_standards 
SET is_active = FALSE, updated_at = CURRENT_TIMESTAMP(6)
WHERE standard_name = 'ts-001-zulu-required' AND is_active = TRUE;

-- Step 2: Create new version
INSERT INTO metadata_standards (
    id, standard_name, category, pattern_regex, 
    validation_message, enforcement_level, is_active
) VALUES (
    UUID(), 'ts-001-zulu-required-v2', 'temporal',
    '^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?Z$',
    'Timestamp must use ISO 8601 with Z suffix (v2: added microseconds)',
    'error', TRUE
);
```

---

### CAP-SEC-40: Violation Suppression Audit

**Purpose:** Ensure complete audit trail for all violation suppressions, preventing hidden compliance issues and maintaining accountability.

**Implementation:**

```rust
// security/src/violation_suppression_audit.rs

pub struct ViolationSuppressionAudit {
    db: TiDBClient,
}

impl ViolationSuppressionAudit {
    /// Suppress a violation with full audit trail
    pub async fn suppress_violation(
        &self,
        violation_id: &str,
        user_id: &str,
        justification: &str,
        expiration: Option<DateTime<Utc>>,
    ) -> Result<SuppressionRecord, SecurityError> {
        // Validate justification length
        if justification.len() < 50 {
            return Err(SecurityError::InsufficientJustification {
                minimum_chars: 50,
                provided_chars: justification.len(),
            });
        }
        
        // Create suppression record
        let suppression = SuppressionRecord {
            id: Uuid::new_v4().to_string(),
            violation_id: violation_id.to_string(),
            suppressed_by: user_id.to_string(),
            suppressed_at: Utc::now(),
            justification: justification.to_string(),
            expires_at: expiration,
            reviewed_by: None,
            review_status: ReviewStatus::Pending,
        };
        
        // Update violation status
        self.db.execute(
            "UPDATE metadata_violations 
             SET status = 'suppressed', resolved_at = CURRENT_TIMESTAMP(6), resolved_by = ?
             WHERE id = ?",
            &[user_id, violation_id]
        ).await?;
        
        // Log suppression in audit trail
        self.db.execute(
            "INSERT INTO audit_logs 
             (timestamp, user_id, action, resource_type, resource_id, status, metadata)
             VALUES (CURRENT_TIMESTAMP(6), ?, 'VIOLATION_SUPPRESSED', 'metadata_violation', ?, 'success', ?)",
            &[user_id, violation_id, json!({
                "justification": justification,
                "expires_at": expiration,
                "cap": "CAP-SEC-40"
            })]
        ).await?;
        
        // Notify security team for review
        self.notify_security_review(&suppression).await?;
        
        Ok(suppression)
    }
    
    /// Get suppression audit report
    pub async fn get_suppression_report(
        &self,
        start_date: DateTime<Utc>,
        end_date: DateTime<Utc>,
    ) -> Vec<SuppressionReportEntry> {
        self.db.query(
            "SELECT 
                mv.id AS violation_id,
                mv.standard_id,
                ms.standard_name,
                mv.resource_type,
                mv.resource_id,
                mv.violation_value,
                mv.resolved_by AS suppressed_by,
                mv.resolved_at AS suppressed_at,
                al.metadata->>'$.justification' AS justification
             FROM metadata_violations mv
             JOIN metadata_standards ms ON mv.standard_id = ms.id
             JOIN audit_logs al ON al.resource_id = mv.id 
                  AND al.action = 'VIOLATION_SUPPRESSED'
             WHERE mv.status = 'suppressed'
               AND mv.resolved_at BETWEEN ? AND ?
             ORDER BY mv.resolved_at DESC",
            &[start_date, end_date]
        ).await.unwrap_or_default()
    }
    
    /// Check for expired suppressions
    pub async fn check_expired_suppressions(&self) -> Vec<ExpiredSuppression> {
        self.db.query(
            "SELECT id, resolved_by, resolved_at
             FROM metadata_violations
             WHERE status = 'suppressed'
               AND JSON_EXTRACT(metadata, '$.expires_at') < CURRENT_TIMESTAMP(6)",
            &[]
        ).await.unwrap_or_default()
    }
    
    async fn notify_security_review(&self, suppression: &SuppressionRecord) -> Result<(), SecurityError> {
        // Send SSE notification to security dashboard
        // Implementation depends on notification system
        Ok(())
    }
}

#[derive(Debug)]
pub struct SuppressionRecord {
    pub id: String,
    pub violation_id: String,
    pub suppressed_by: String,
    pub suppressed_at: DateTime<Utc>,
    pub justification: String,
    pub expires_at: Option<DateTime<Utc>>,
    pub reviewed_by: Option<String>,
    pub review_status: ReviewStatus,
}

#[derive(Debug)]
pub enum ReviewStatus {
    Pending,
    Approved,
    Rejected,
}
```

**Suppression Workflow:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VIOLATION SUPPRESSION WORKFLOW                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. User requests suppression with justification (>50 chars)      │
│      ↓                                                              │
│   2. System validates justification completeness                   │
│      ↓                                                              │
│   3. Violation status → 'suppressed'                               │
│      ↓                                                              │
│   4. Audit log entry created with full context                     │
│      ↓                                                              │
│   5. Security team notified for review                             │
│      ↓                                                              │
│   6. Review: APPROVE or REJECT (requires different user)           │
│      ↓                                                              │
│   7. If REJECT: Violation reopened, user notified                  │
│   7. If APPROVE: Suppression finalized                             │
│      ↓                                                              │
│   8. Expiration check runs daily (cron)                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Audit Query:**

```sql
-- Suppression audit report for compliance
SELECT 
    DATE(al.timestamp) AS suppression_date,
    COUNT(*) AS total_suppressions,
    COUNT(DISTINCT al.user_id) AS unique_suppressors,
    ms.category AS standard_category,
    ms.enforcement_level
FROM audit_logs al
JOIN metadata_violations mv ON al.resource_id = mv.id
JOIN metadata_standards ms ON mv.standard_id = ms.id
WHERE al.action = 'VIOLATION_SUPPRESSED'
  AND al.timestamp >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(al.timestamp), ms.category, ms.enforcement_level
ORDER BY suppression_date DESC;
```

---

## Dashboard Capability Declaration

### Elm Module Pattern

```elm
-- Dashboard/Security.elm
module Dashboard.Security exposing (..)

-- REQUIRED: Capability declaration
capabilities : List Capability
capabilities =
    [ ReadProjectConfig       -- Need to validate tech stack
    , ReadGuardrails          -- Access guardrail rules
    , WriteViolations         -- Report security issues
    , RequestAICompletion     -- AI vulnerability scanning
    , ValidateWithAI          -- AI-powered validation
    , RenderCanvas            -- Threat model diagram
    -- NEW: Temporal Security Capabilities (v1.1.0)
    , CAP_SEC_36_ClockDriftDetection
    , CAP_SEC_37_TimestompingDetection
    , CAP_SEC_38_TimezoneInjectionPrevention
    , AccessClockSyncStatus   -- Monitor clock drift
    ]

-- REQUIRED: Metadata
metadata : DashboardMetadata
metadata =
    { name = "Security Dashboard"
    , version = "1.2.0"
    , author = "DevGuide Team"
    , category = "security"
    , certificationTier = Tier1Certified
    }

-- Dashboard implementation
init : Flags -> ( Model, Cmd Msg )
init flags =
    if not (hasAllCapabilities capabilities) then
        ( { model | error = Just "Missing required capabilities" }
        , Cmd.none
        )
    else
        ( initializeSecurityChecks flags
        , fetchSecurityConfig
        )
```

### Capability Manifest (YAML)

```yaml
# .devguide/dashboards/security/CAPABILITIES.yaml
dashboard:
  id: "security-001"
  name: "Security Dashboard"
  version: "1.2.0"

capabilities:
  data_access:
    - ReadProjectConfig
    - ReadGuardrails
    - AccessClockSyncStatus       # NEW v1.1.0
    - AccessFilesystemTimestamps  # NEW v1.1.0
  
  modification:
    - WriteViolations
  
  ai:
    - RequestAICompletion
    - ValidateWithAI
  
  rendering:
    - RenderCanvas
  
  temporal_security:              # NEW v1.1.0
    - CAP_SEC_36_ClockDriftDetection
    - CAP_SEC_37_TimestompingDetection
    - CAP_SEC_38_TimezoneInjectionPrevention
    - CAP_SEC_39_MetadataStandardsImmutability
    - CAP_SEC_40_ViolationSuppressionAudit

rationale:
  ReadProjectConfig: "Validate tech stack security settings"
  WriteViolations: "Report security issues to The Bin"
  ValidateWithAI: "AI-powered vulnerability scanning"
  RenderCanvas: "Display threat model diagrams"
  CAP_SEC_36_ClockDriftDetection: "Monitor NTP/PTP sync across components"
  CAP_SEC_37_TimestompingDetection: "Detect filesystem timestamp manipulation"
  CAP_SEC_38_TimezoneInjectionPrevention: "Enforce Zulu-only timestamps"
  CAP_SEC_39_MetadataStandardsImmutability: "Protect governance standards"
  CAP_SEC_40_ViolationSuppressionAudit: "Audit all suppression actions"

alternatives_if_denied:
  ValidateWithAI: "Manual security checklist (slower)"
  RenderCanvas: "Text-based threat model (no viz)"
  CAP_SEC_36_ClockDriftDetection: "Manual NTP status check (unreliable)"
```

---

## Three-Layer Enforcement

```
┌──────────────────────────────────────────────────────────────────────┐
│ Layer 1: Elm Compile-Time Checks                                     │
├──────────────────────────────────────────────────────────────────────┤
│ • Dashboard declares capabilities in Elm module                      │
│ • Compiler ensures capabilities list present                         │
│ • Type system prevents calling without cap                           │
└──────────────────────────────────┬───────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────────────────────────┐
│ Layer 2: Runtime Capability Registry (Go API)                        │
├──────────────────────────────────────────────────────────────────────┤
│ • Dashboard registers capabilities on load                           │
│ • Registry stored in Redis (fast lookup)                             │
│ • Every API call checks registry                                     │
│ • Middleware: RequireCapability(ReadViolations)                      │
│ • NEW: Temporal capabilities checked via clock_sync_status           │
└──────────────────────────────────┬───────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────────────────────────┐
│ Layer 3: The Bin Checkpoint (Transaction)                            │
├──────────────────────────────────────────────────────────────────────┤
│ • Before commit, verify dashboard had capability                     │
│ • Audit log records which capabilities used                          │
│ • Rollback transaction if capability missing                         │
│ • Prevent privilege escalation attacks                               │
│ • NEW: CAP-SEC-38 enforced at database trigger level                 │
└──────────────────────────────────────────────────────────────────────┘
```

---

## JavaScript/Port Sandboxing

### Port Manifest

```yaml
# .devguide/dashboards/architecture/PORTS.yaml
ports:
  - name: "renderComponentDiagram"
    direction: "outgoing"  # Elm → JavaScript
    capabilities_required:
      - RenderCanvas
      - ExecuteJavaScript
    security_review: "approved"
    reviewer: "security-team"
    approved_at: "2025-12-15T10:00:00Z"
    
    allowed_libraries:
      - name: "d3"
        version: "7.8.5"
        source: "https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js"
        integrity: "sha384-..."  # SRI hash
    
    sandboxing:
      iframe: true  # Run in isolated iframe
      csp: "script-src 'self' https://cdnjs.cloudflare.com"
      max_execution_time_ms: 5000
      memory_limit_mb: 50

  - name: "receiveCanvasEvents"
    direction: "incoming"  # JavaScript → Elm
    capabilities_required:
      - ExecuteJavaScript
    event_types: ["click", "drag", "zoom"]
    rate_limit: 100  # Max 100 events/second
```

### Port Registration

```elm
port module Dashboard.Architecture exposing (..)

port renderComponentDiagram : DiagramData -> Cmd msg
port receiveCanvasEvents : (CanvasEvent -> msg) -> Sub msg

init : Flags -> ( Model, Cmd Msg )
init flags =
    if not (hasCapability ExecuteJavaScript) then
        ( { model | error = Just "ExecuteJavaScript capability missing" }
        , Cmd.none
        )
    else
        ( model
        , registerPort "renderComponentDiagram" capabilities
        )
```

**Go API Verification:**
```go
func RegisterPort(dashboardID, portName string, caps []Capability) error {
    manifest, err := loadPortManifest(dashboardID, portName)
    if err != nil {
        return fmt.Errorf("port manifest not found: %w", err)
    }
    
    // Verify capabilities match manifest
    for _, requiredCap := range manifest.CapabilitiesRequired {
        if !contains(caps, requiredCap) {
            return fmt.Errorf("missing capability: %s", requiredCap)
        }
    }
    
    // Register in runtime registry
    return portRegistry.Register(dashboardID, portName, manifest)
}
```

---

## Audit Trail

### Capability Usage Logging

```sql
CREATE TABLE capability_usage_log (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    dashboard_id VARCHAR(255) NOT NULL,
    capability VARCHAR(100) NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    resource_accessed VARCHAR(500),
    success BOOLEAN NOT NULL,
    denial_reason VARCHAR(500),
    
    INDEX idx_dashboard (dashboard_id),
    INDEX idx_capability (capability),
    INDEX idx_timestamp (timestamp)
) ENGINE=InnoDB;
```

**Log Every Capability Usage:**
```go
func (cm *CapabilityManager) RecordUsage(ctx context.Context, dashboardID string, capability Capability, success bool, denialReason string) error {
    _, err := cm.db.ExecContext(ctx, `
        INSERT INTO capability_usage_log 
        (timestamp, dashboard_id, capability, user_id, success, denial_reason)
        VALUES (CURRENT_TIMESTAMP(6), ?, ?, ?, ?, ?)
    `, dashboardID, capability.String(), getUserID(ctx), success, denialReason)
    
    return err
}
```

---

## Web Component Permission System

### Component Manifest

```yaml
# .devguide/components/chart-builder/MANIFEST.yaml
component:
  name: "chart-builder"
  version: "2.1.0"
  author: "community-contributor"
  
capabilities_required:
  - RenderCanvas
  - LoadExternalAssets  # For Chart.js CDN
  
external_dependencies:
  - name: "chart.js"
    version: "4.4.0"
    source: "https://cdn.jsdelivr.net/npm/chart.js@4.4.0"
    integrity: "sha384-..."
  
sandboxing:
  shadow_dom: true  # Isolate styles
  csp: "script-src 'self' https://cdn.jsdelivr.net"
```

---

## Temporal Security Summary Table

| Capability | ID | Enforcement Layer | Database Table | Trigger |
|------------|-----|-------------------|----------------|---------|
| Clock Drift Detection | CAP-SEC-36 | Runtime (Go) | clock_sync_status | Polling (30s) |
| Timestomping Detection | CAP-SEC-37 | OS (auditd) | filesystem_timestamps | Event-driven |
| Timezone Injection Prevention | CAP-SEC-38 | All layers | N/A (validation) | Every request |
| Metadata Standards Immutability | CAP-SEC-39 | Database (trigger) | metadata_standards | INSERT/UPDATE/DELETE |
| Violation Suppression Audit | CAP-SEC-40 | API (Go) | audit_logs | Suppression action |

---

## Testing & Validation

### Capability Enforcement Tests

```elm
-- test/CapabilityEnforcementTest.elm
testMissingCapability : Test
testMissingCapability =
    test "Dashboard without capability cannot access resource" <|
        \() ->
            let
                dashboard =
                    { capabilities = [ ReadProjectConfig ]
                    , attemptedAction = WriteViolations
                    }
                
                result = executeAction dashboard
            in
            Expect.err result
```

### Temporal Security Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_zulu_only_validation() {
        // Valid Zulu timestamps
        assert!(TimezoneInjectionPrevention::validate_zulu_only("2026-01-01T14:30:45Z").is_ok());
        assert!(TimezoneInjectionPrevention::validate_zulu_only("2026-01-01T14:30:45.123456Z").is_ok());
        
        // Invalid: offset format
        assert!(TimezoneInjectionPrevention::validate_zulu_only("2026-01-01T14:30:45+05:30").is_err());
        assert!(TimezoneInjectionPrevention::validate_zulu_only("2026-01-01T14:30:45-08:00").is_err());
        
        // Invalid: no timezone
        assert!(TimezoneInjectionPrevention::validate_zulu_only("2026-01-01T14:30:45").is_err());
    }

    #[test]
    fn test_clock_drift_thresholds() {
        let detector = ClockDriftDetector::new(mock_db());
        
        // Normal drift
        assert!(matches!(detector.evaluate_drift(5), DriftStatus::Synchronized { .. }));
        
        // Alert threshold
        assert!(matches!(detector.evaluate_drift(50), DriftStatus::Alert { .. }));
        
        // Quarantine threshold
        assert!(matches!(detector.evaluate_drift(150), DriftStatus::Quarantine { .. }));
    }
}
```

---

## Open Questions

1. Should capability grants expire? (Suggest: No, but audit regularly)
2. Maximum capabilities per dashboard? (Suggest: 10)
3. Should users be able to revoke capabilities post-installation? (Suggest: Yes, with warning)
4. **NEW:** Clock drift quarantine auto-recovery threshold? (Suggest: 3 consecutive checks within threshold)
5. **NEW:** Suppression justification minimum length? (Current: 50 chars - is this sufficient?)
6. **NEW:** Should timestomping detection trigger automatic incident response? (Suggest: Yes, for production)

---

## Cross-References

| Artifact | Relationship |
|----------|--------------|
| ARTIFACT_03_THE_BIN_SPECIFICATION.md | Integrates MetadataValidator for CAP-SEC-38 enforcement |
| ARTIFACT_05_DATA_MODEL.md | Defines clock_sync_status, metadata_standards, audit_logs tables |
| ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md | Provides UI for temporal security monitoring |
| ARTIFACT_06_MASTER_CONTROL_DASHBOARD.md | Displays security capability status widgets |

---

**End of ARTIFACT_13_SECURITY_CAPABILITY_MODEL.md v1.1.0**
