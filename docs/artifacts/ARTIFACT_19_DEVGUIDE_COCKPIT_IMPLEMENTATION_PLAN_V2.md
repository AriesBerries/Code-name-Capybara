# DevGuide Cockpit: Implementation Plan V2

**Document Type:** Implementation Roadmap (Tier 6)  
**Status:** Active  
**Version:** 2.1.1  
**Date:** 2026-01-02T01:00:00Z  
**Supersedes:** ARTIFACT_19 v2.0, v2.1.0  
**Dependencies:** ALL 20 artifacts (19 original + ARTIFACT_20)

---

## Changelog

### v2.1.1 (2026-01-02T01:00:00Z)
- Added pre-work requirement for SQL migration scripts
- Clarified 7th engineer onboarding timeline
- Adjusted Week 5 scope (community repository MVP reduced)
- Added Risk 6: Team Onboarding Delays
- Added this Changelog section

### v2.1.0 (2026-01-02T00:00:00Z)
- Integrated Metadata Governance Dashboard (ARTIFACT_20)
- Integrated AI Cortex Management Dashboard
- Integrated VS Code Extension Orchestration System
- Added 11 new database tables to Week 1
- Expanded team from 6 to 7 engineers
- Updated budget: $150/mo MVP, $1,300/mo Production

---

## Executive Summary

This artifact provides a **comprehensive 3-month implementation roadmap** for DevGuide Cockpit MVP, synthesizing all 20 specification artifacts into a concrete delivery plan. **Version 2.1.0 adds three major systems:**

1. **Metadata Governance Dashboard** - Temporal standardization, nomenclature validation
2. **AI Cortex Management Dashboard** - Token tracking, context monitoring, multi-provider APIs
3. **VS Code Extension Orchestration** - Proxy layer, dashboard builder, community repository

**Timeline:**
- **Month 1 (Weeks 1-4)**: Core Infrastructure - The Bin, Metadata Governance, AI Cortex
- **Month 2 (Weeks 5-8)**: Templates, Extension Orchestration, Integration
- **Month 3 (Weeks 9-12)**: Polish, Documentation, Governance Features, Beta Launch

**Team Size:** 7 engineers (2 Elm, 2 Go, 1 Rust, 2 DevOps)  
**Target:** Production-ready MVP supporting 100 beta users  
**Budget:** $150/month MVP, $1,300/month Production

---

## What's New in v2.1.0

### New Systems Added

| System | Week | Key Deliverables |
|--------|------|------------------|
| **Metadata Governance** | 1-3 | 11 DB tables, validation layer, compliance UI |
| **AI Cortex** | 4 | Token tracking, context monitoring, estimation AI |
| **Extension Orchestration** | 5 | Proxy layer, dashboard builder, community repo |

### Resource Changes

| Resource | v2.0 | v2.1.0 | Delta |
|----------|------|--------|-------|
| Engineers | 6 | 7 | +1 DevOps |
| MVP Budget | $100/mo | $150/mo | +$50 |
| Production Budget | $1,100/mo | $1,300/mo | +$200 |

### Key Dependencies Added

```
ARTIFACT_20 (Metadata Governance Dashboard)
├── → ARTIFACT_05 (4 new tables)
├── → ARTIFACT_03 (Layer 0 validation)
├── → ARTIFACT_06 (governance panel)
└── → This document (Week 1 tasks)

AI_CORTEX_MANAGEMENT_DASHBOARD
├── → ARTIFACT_05 (4 new tables)
├── → ARTIFACT_04 (timestamp normalization)
└── → ARTIFACT_06 (AI Cortex panel)

VSCODE_EXTENSION_ORCHESTRATION_SYSTEM
├── → ARTIFACT_05 (3 new tables)
└── → ARTIFACT_06 (extension panel)
```

---

## Vision & Success Criteria

### Vision (From ARTIFACT_01)

DevGuide Cockpit is an **AI-powered dashboard orchestration system** that helps developers build software projects with zero knowledge synchronization issues. Every decision, every piece of code, every document flows through **The Bin** - the central validation and merge gate that enforces governance rules and maintains project coherence.

### Success Criteria

**Month 1 (Infrastructure):**
- [ ] The Bin validates updates with **Layer 0 Metadata Governance**
- [ ] All 11 new database tables deployed and seeded
- [ ] Master Control Dashboard shows project status + governance panel
- [ ] AI Cortex provides token tracking and context monitoring
- [ ] Clock synchronization monitoring operational (< 10ms drift)
- [ ] TiDB stores all project data with HTAP queries

**Month 2 (Templates & Extensions):**
- [ ] 3 dashboard templates available (Vibe Coding, Data Science, Note Taking)
- [ ] Extension Orchestration MVP functional
- [ ] Templates integrate metadata compliance checks
- [ ] AI Cortex panels embedded in dashboards
- [ ] Community dashboard repository accessible

**Month 3 (Polish):**
- [ ] PWA works offline (service worker, IndexedDB)
- [ ] Governance documentation complete
- [ ] Migration wizard guides version upgrades
- [ ] 100 beta users actively using system
- [ ] Metadata compliance rate > 98%

---

## Month 1: Core Infrastructure (Weeks 1-4)

### Week 1: Foundation, Database, & Metadata Governance Setup

**Deliverable:** TiDB with all 11 new tables, The Bin foundation, metadata seeding

> ⚠️ **PRE-WORK REQUIRED:** SQL migration scripts for all 11 tables must be written and tested on a staging database BEFORE Week 1 begins. Allocate 2-3 days of pre-project preparation for Infrastructure Lead.

#### Day 1-2: Project Setup & Database Deployment

**Tasks:**
1. Initialize monorepo structure (from ARTIFACT_03_REPOSITORY_STRUCTURE)
   ```bash
   devguide-cockpit/
   ├── bin/              # Rust validation kernel
   ├── api/              # Go API orchestration
   ├── dashboard/        # Elm frontend
   ├── chromium-shell/   # Chromium wrapper
   ├── docs/             # Documentation
   └── .github/          # CI/CD workflows
   ```

2. Setup TiDB Cloud cluster with **all 11 new tables** (from ARTIFACT_05, ARTIFACT_20, ARTIFACT_21, ARTIFACT_22)
   ```sql
   -- ============================================
   -- METADATA GOVERNANCE TABLES (4 new)
   -- ============================================
   
   CREATE TABLE metadata_standards (
       id VARCHAR(36) PRIMARY KEY,
       standard_name VARCHAR(100) UNIQUE NOT NULL,
       category ENUM('temporal', 'nomenclature', 'versioning') NOT NULL,
       pattern_regex TEXT NOT NULL,
       validation_message TEXT,
       enforcement_level ENUM('error', 'warning', 'info') DEFAULT 'error',
       is_active BOOLEAN DEFAULT TRUE,
       created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       INDEX idx_category (category),
       INDEX idx_active (is_active)
   );

   CREATE TABLE metadata_violations (
       id VARCHAR(36) PRIMARY KEY,
       standard_id VARCHAR(36) REFERENCES metadata_standards(id),
       resource_type VARCHAR(50) NOT NULL,
       resource_id VARCHAR(255) NOT NULL,
       violation_value TEXT,
       suggested_fix TEXT,
       status ENUM('open', 'resolved', 'suppressed') DEFAULT 'open',
       detected_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       resolved_at TIMESTAMP(6),
       resolved_by VARCHAR(36),
       INDEX idx_resource (resource_type, resource_id),
       INDEX idx_status (status),
       INDEX idx_detected (detected_at)
   );

   CREATE TABLE filesystem_timestamps (
       id VARCHAR(36) PRIMARY KEY,
       file_path TEXT NOT NULL,
       atime TIMESTAMP(6) NOT NULL,  -- Access time
       mtime TIMESTAMP(6) NOT NULL,  -- Modification time
       ctime TIMESTAMP(6) NOT NULL,  -- Change time
       captured_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       git_commit_sha VARCHAR(40),
       INDEX idx_file_path (file_path(255)),
       INDEX idx_mtime (mtime)
   );

   CREATE TABLE clock_sync_status (
       id VARCHAR(36) PRIMARY KEY,
       component_name VARCHAR(100) NOT NULL,
       protocol ENUM('NTP', 'PTP', 'Browser', 'Server') NOT NULL,
       stratum INT,
       offset_microseconds BIGINT NOT NULL,
       last_sync_at TIMESTAMP(6) NOT NULL,
       status ENUM('synchronized', 'drifting', 'error') NOT NULL,
       INDEX idx_component (component_name),
       INDEX idx_status (status)
   );

   -- ============================================
   -- AI CORTEX TABLES (4 new)
   -- ============================================

   CREATE TABLE token_usage (
       id VARCHAR(36) PRIMARY KEY,
       project_id VARCHAR(36) NOT NULL,
       provider VARCHAR(50) NOT NULL,  -- 'claude', 'openai', etc.
       model VARCHAR(100) NOT NULL,
       input_tokens INT NOT NULL,
       output_tokens INT NOT NULL,
       estimated_cost_usd DECIMAL(10, 6),
       operation_type VARCHAR(50),  -- 'completion', 'embedding', etc.
       recorded_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
       INDEX idx_project_provider (project_id, provider),
       INDEX idx_recorded (recorded_at)
   );

   CREATE TABLE api_configurations (
       id VARCHAR(36) PRIMARY KEY,
       project_id VARCHAR(36),  -- NULL for global configs
       provider VARCHAR(50) NOT NULL,
       api_key_encrypted VARBINARY(512),  -- AES-256-GCM encrypted
       endpoint_url VARCHAR(500),
       model_default VARCHAR(100),
       rate_limit_rpm INT DEFAULT 60,
       is_active BOOLEAN DEFAULT TRUE,
       created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
       INDEX idx_project (project_id),
       INDEX idx_provider (provider)
   );

   CREATE TABLE tool_registry (
       id VARCHAR(36) PRIMARY KEY,
       tool_name VARCHAR(100) NOT NULL UNIQUE,
       tool_type ENUM('devguide_native', 'external', 'ai_generated') NOT NULL,
       description TEXT,
       input_schema JSON,
       output_schema JSON,
       estimated_latency_ms INT,
       is_enabled BOOLEAN DEFAULT TRUE,
       created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       INDEX idx_type (tool_type),
       INDEX idx_enabled (is_enabled)
   );

   CREATE TABLE external_tool_access (
       id VARCHAR(36) PRIMARY KEY,
       project_id VARCHAR(36),  -- NULL for universal access
       tool_id VARCHAR(36) NOT NULL,
       access_level ENUM('none', 'read', 'write', 'admin') NOT NULL,
       granted_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       granted_by VARCHAR(36),
       revoked_at TIMESTAMP(6),
       FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
       FOREIGN KEY (tool_id) REFERENCES tool_registry(id) ON DELETE CASCADE,
       INDEX idx_project_tool (project_id, tool_id)
   );

   -- ============================================
   -- EXTENSION ORCHESTRATION TABLES (3 new)
   -- ============================================

   CREATE TABLE extension_dashboards (
       id VARCHAR(36) PRIMARY KEY,
       extension_id VARCHAR(100) NOT NULL,  -- VS Code extension ID
       dashboard_name VARCHAR(255) NOT NULL,
       author VARCHAR(255),
       description TEXT,
       specification JSON NOT NULL,  -- Dashboard layout and config
       transform_rules JSON,  -- Input/output transformations
       community_rating DECIMAL(2, 1),
       download_count INT DEFAULT 0,
       is_verified BOOLEAN DEFAULT FALSE,
       created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       INDEX idx_extension (extension_id),
       INDEX idx_rating (community_rating DESC),
       INDEX idx_downloads (download_count DESC)
   );

   CREATE TABLE project_extension_mappings (
       id VARCHAR(36) PRIMARY KEY,
       project_id VARCHAR(36) NOT NULL,
       extension_id VARCHAR(100) NOT NULL,
       dashboard_id VARCHAR(36) NOT NULL,  -- Which dashboard template to use
       custom_config JSON,  -- Per-project overrides
       is_active BOOLEAN DEFAULT TRUE,
       created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
       FOREIGN KEY (dashboard_id) REFERENCES extension_dashboards(id),
       UNIQUE KEY uk_project_extension (project_id, extension_id)
   );

   CREATE TABLE dashboard_versions (
       id VARCHAR(36) PRIMARY KEY,
       dashboard_id VARCHAR(36) NOT NULL,
       version VARCHAR(20) NOT NULL,  -- SemVer
       specification JSON NOT NULL,
       changelog TEXT,
       published_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
       FOREIGN KEY (dashboard_id) REFERENCES extension_dashboards(id) ON DELETE CASCADE,
       INDEX idx_dashboard_version (dashboard_id, version)
   );
   ```

3. Configure CI/CD pipelines
   ```yaml
   # .github/workflows/main.yml
   - Elm tests (elm-test)
   - Rust tests (cargo test)
   - Go tests (go test -race ./...)
   - Lighthouse accessibility (score ≥ 90)
   - Language enforcement (no .py, .js, .ts files)
   - Metadata validation (timestamps must end in Z)
   ```

**Acceptance Criteria:**
- [ ] Monorepo builds successfully
- [ ] TiDB cluster accessible with all 11 new tables
- [ ] All CI/CD checks pass
- [ ] Schema migration scripts version-controlled

#### Day 3: Seed Default Metadata Standards

**Tasks:**
1. Insert default temporal standards (from ARTIFACT_20)
   ```sql
   -- Seed metadata_standards table
   INSERT INTO metadata_standards (id, standard_name, category, pattern_regex, validation_message, enforcement_level) VALUES
   -- Temporal Standards
   ('std-ts-001', 'ISO8601_EXTENDED_ZULU', 'temporal', 
    '^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}(\\.\\d{1,6})?Z$',
    'Timestamp must use ISO 8601 extended format with Z suffix (e.g., 2026-01-01T14:30:45Z)',
    'error'),
   
   ('std-ts-002', 'ISO8601_BASIC_ZULU', 'temporal',
    '^\\d{8}T\\d{6}Z$',
    'Compact timestamp must use ISO 8601 basic format with Z suffix (e.g., 20260101T143045Z)',
    'error'),
   
   -- Nomenclature Standards
   ('std-nm-001', 'REPOSITORY_NAMING', 'nomenclature',
    '^[a-z]+-[a-z]+-[a-z]+-[a-z]+$',
    'Repository name must follow <domain>-<technology>-<maturity>-<topology> pattern',
    'warning'),
   
   ('std-nm-002', 'BRANCH_NAMING', 'nomenclature',
    '^(feat|fix|ai|refactor|docs|chore)/[A-Z]+-\\d+-[a-z0-9-]+$',
    'Branch name must follow <type>/<TICKET>-<description> pattern',
    'warning'),
   
   ('std-nm-003', 'FILE_NAMING_LPDS', 'nomenclature',
    '^[a-z]+_[a-z]+-[a-z]+.*\\.[a-z]+$',
    'File name should follow LPDS pattern: <domain>_<type>-<variant>.<ext>',
    'info'),
   
   -- Versioning Standards
   ('std-vs-001', 'SEMVER_STRICT', 'versioning',
    '^\\d+\\.\\d+\\.\\d+$',
    'Version must follow Semantic Versioning 2.0.0 (MAJOR.MINOR.PATCH)',
    'error'),
   
   ('std-vs-002', 'SEMVER_WITH_BUILD', 'versioning',
    '^\\d+\\.\\d+\\.\\d+(\\+\\d{8}T\\d{6}Z)?$',
    'Version may include Zulu build metadata (e.g., 1.2.3+20260101T143045Z)',
    'info');
   ```

2. Insert default tool registry entries
   ```sql
   INSERT INTO tool_registry (id, tool_name, tool_type, description, estimated_latency_ms) VALUES
   ('tool-001', 'tiktoken_counter', 'devguide_native', 'Fast token counting using tiktoken', 1),
   ('tool-002', 'timestamp_normalizer', 'devguide_native', 'Convert timestamps to Zulu format', 1),
   ('tool-003', 'semver_validator', 'devguide_native', 'Validate Semantic Version strings', 1),
   ('tool-004', 'lpds_name_generator', 'devguide_native', 'Generate LPDS-compliant file names', 2),
   ('tool-005', 'context_estimator', 'devguide_native', 'Estimate tokens for context window', 5);
   ```

**Acceptance Criteria:**
- [ ] 7 metadata standards seeded
- [ ] 5 default tools registered
- [ ] Standards queryable via API

#### Day 4: Configure NTP/PTP Clock Sync Monitoring

**Tasks:**
1. Deploy chrony for NTP synchronization (from ARTIFACT_20)
   ```bash
   # Install chrony on all API servers
   sudo apt install chrony
   
   # Configure chrony.conf for AWS Time Sync
   server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4
   
   # Verify synchronization
   chronyc tracking
   ```

2. Implement clock sync monitoring service (Go)
   ```go
   // api/monitoring/clock_sync.go
   package monitoring

   import (
       "context"
       "os/exec"
       "regexp"
       "strconv"
       "time"
   )

   type ClockSyncMonitor struct {
       db        *TiDBClient
       component string
       interval  time.Duration
   }

   func (m *ClockSyncMonitor) Start(ctx context.Context) {
       ticker := time.NewTicker(m.interval)
       defer ticker.Stop()

       for {
           select {
           case <-ctx.Done():
               return
           case <-ticker.C:
               status := m.checkClockSync()
               m.recordStatus(ctx, status)
               
               if status.OffsetMicroseconds > 10000 { // > 10ms
                   m.alertClockDrift(status)
               }
           }
       }
   }

   func (m *ClockSyncMonitor) checkClockSync() ClockSyncStatus {
       out, _ := exec.Command("chronyc", "tracking").Output()
       offset := m.parseOffset(string(out))
       
       status := "synchronized"
       if offset > 10000 {
           status = "drifting"
       }
       
       return ClockSyncStatus{
           ComponentName:      m.component,
           Protocol:           "NTP",
           OffsetMicroseconds: offset,
           LastSyncAt:         time.Now().UTC(),
           Status:             status,
       }
   }
   ```

**Acceptance Criteria:**
- [ ] Chrony installed on all API servers
- [ ] Clock sync status recorded every 30 seconds
- [ ] Alert fires when drift > 10ms

#### Day 5: Set Up Filesystem Timestamp Capture

**Tasks:**
1. Implement MAC time capture service
   ```go
   // api/monitoring/filesystem_timestamps.go
   package monitoring

   import (
       "os"
       "syscall"
       "time"
   )

   type FilesystemMonitor struct {
       db       *TiDBClient
       watchDir string
   }

   func (m *FilesystemMonitor) CaptureTimestamps(filePath string) (*FilesystemTimestamp, error) {
       info, err := os.Stat(filePath)
       if err != nil {
           return nil, err
       }

       stat := info.Sys().(*syscall.Stat_t)
       
       return &FilesystemTimestamp{
           FilePath:   filePath,
           Atime:      time.Unix(stat.Atim.Sec, stat.Atim.Nsec).UTC(),
           Mtime:      info.ModTime().UTC(),
           Ctime:      time.Unix(stat.Ctim.Sec, stat.Ctim.Nsec).UTC(),
           CapturedAt: time.Now().UTC(),
       }, nil
   }

   func (m *FilesystemMonitor) DetectTimestomping(filePath string) bool {
       current, _ := m.CaptureTimestamps(filePath)
       previous := m.getLastCapture(filePath)
       
       if previous == nil {
           return false
       }
       
       // Mtime moved backward = potential timestomping
       return current.Mtime.Before(previous.Mtime)
   }
   ```

**Acceptance Criteria:**
- [ ] MAC times captured for tracked files
- [ ] Timestomping detection functional
- [ ] Integration with auditd for security events

---

### Week 2: The Bin - Metadata Validation Layer (Part 1)

**Deliverable:** MetadataValidator struct integrated into The Bin as Layer 0

#### Day 6-7: Implement MetadataValidator Struct (Rust)

**Tasks:**
1. Create metadata validation kernel (from ARTIFACT_20, ARTIFACT_03)
   ```rust
   // bin/src/metadata_validator.rs
   use wasm_bindgen::prelude::*;
   use serde::{Deserialize, Serialize};
   use regex::Regex;
   use std::collections::HashMap;

   #[derive(Serialize, Deserialize, Clone)]
   pub struct MetadataStandard {
       pub id: String,
       pub standard_name: String,
       pub category: String,
       pub pattern_regex: String,
       pub validation_message: String,
       pub enforcement_level: String,
   }

   #[derive(Serialize, Deserialize)]
   pub struct MetadataViolation {
       pub standard_id: String,
       pub resource_type: String,
       pub resource_id: String,
       pub violation_value: String,
       pub suggested_fix: String,
       pub severity: String,
   }

   #[derive(Serialize, Deserialize)]
   pub struct MetadataValidationResult {
       pub is_valid: bool,
       pub violations: Vec<MetadataViolation>,
       pub warnings: Vec<MetadataViolation>,
   }

   pub struct MetadataValidator {
       standards: HashMap<String, MetadataStandard>,
       compiled_regexes: HashMap<String, Regex>,
   }

   impl MetadataValidator {
       pub fn new(standards: Vec<MetadataStandard>) -> Self {
           let mut compiled = HashMap::new();
           let mut std_map = HashMap::new();
           
           for standard in standards {
               if let Ok(regex) = Regex::new(&standard.pattern_regex) {
                   compiled.insert(standard.id.clone(), regex);
               }
               std_map.insert(standard.id.clone(), standard);
           }
           
           Self {
               standards: std_map,
               compiled_regexes: compiled,
           }
       }

       pub fn validate(&self, item: &BinItem) -> MetadataValidationResult {
           let mut result = MetadataValidationResult {
               is_valid: true,
               violations: vec![],
               warnings: vec![],
           };

           // Extract and validate timestamps
           for (field_name, timestamp) in item.extract_timestamps() {
               self.validate_timestamp(&timestamp, &field_name, &item.id, &mut result);
           }

           // Extract and validate file names
           for file in &item.modified_files {
               self.validate_filename(&file.name, &item.id, &mut result);
           }

           // Validate version strings
           if let Some(version) = &item.version {
               self.validate_version(version, &item.id, &mut result);
           }

           result.is_valid = result.violations.is_empty();
           result
       }

       fn validate_timestamp(
           &self,
           value: &str,
           field_name: &str,
           resource_id: &str,
           result: &mut MetadataValidationResult,
       ) {
           // Check extended format
           let extended = self.compiled_regexes.get("std-ts-001");
           let basic = self.compiled_regexes.get("std-ts-002");

           let matches_extended = extended.map(|r| r.is_match(value)).unwrap_or(false);
           let matches_basic = basic.map(|r| r.is_match(value)).unwrap_or(false);

           if !matches_extended && !matches_basic {
               let standard = self.standards.get("std-ts-001").unwrap();
               result.violations.push(MetadataViolation {
                   standard_id: "std-ts-001".to_string(),
                   resource_type: "timestamp".to_string(),
                   resource_id: resource_id.to_string(),
                   violation_value: value.to_string(),
                   suggested_fix: self.suggest_timestamp_fix(value),
                   severity: standard.enforcement_level.clone(),
               });
           }
       }

       fn suggest_timestamp_fix(&self, value: &str) -> String {
           // Attempt to parse and reformat
           if let Ok(dt) = chrono::DateTime::parse_from_rfc3339(value) {
               return dt.with_timezone(&chrono::Utc)
                   .format("%Y-%m-%dT%H:%M:%SZ")
                   .to_string();
           }
           
           // Common patterns to fix
           if value.ends_with("+00:00") {
               return value.replace("+00:00", "Z");
           }
           
           format!("Convert '{}' to ISO 8601 with Z suffix", value)
       }

       fn validate_filename(
           &self,
           filename: &str,
           resource_id: &str,
           result: &mut MetadataValidationResult,
       ) {
           if let Some(regex) = self.compiled_regexes.get("std-nm-003") {
               if !regex.is_match(filename) {
                   let standard = self.standards.get("std-nm-003").unwrap();
                   result.warnings.push(MetadataViolation {
                       standard_id: "std-nm-003".to_string(),
                       resource_type: "filename".to_string(),
                       resource_id: resource_id.to_string(),
                       violation_value: filename.to_string(),
                       suggested_fix: self.suggest_filename_fix(filename),
                       severity: standard.enforcement_level.clone(),
                   });
               }
           }
       }

       fn suggest_filename_fix(&self, filename: &str) -> String {
           // Extract parts and suggest LPDS format
           let parts: Vec<&str> = filename.split('.').collect();
           let name = parts.first().unwrap_or(&filename);
           let ext = parts.get(1).unwrap_or(&"");
           
           // Convert camelCase/PascalCase to snake_case
           let snake = name.chars()
               .enumerate()
               .map(|(i, c)| {
                   if c.is_uppercase() && i > 0 {
                       format!("-{}", c.to_lowercase())
                   } else {
                       c.to_lowercase().to_string()
                   }
               })
               .collect::<String>();
           
           format!("domain_{}_proc-stage.{}", snake, ext)
       }

       fn validate_version(
           &self,
           version: &str,
           resource_id: &str,
           result: &mut MetadataValidationResult,
       ) {
           if let Some(regex) = self.compiled_regexes.get("std-vs-001") {
               if !regex.is_match(version) {
                   let standard = self.standards.get("std-vs-001").unwrap();
                   result.violations.push(MetadataViolation {
                       standard_id: "std-vs-001".to_string(),
                       resource_type: "version".to_string(),
                       resource_id: resource_id.to_string(),
                       violation_value: version.to_string(),
                       suggested_fix: "Use SemVer format: MAJOR.MINOR.PATCH (e.g., 1.2.3)".to_string(),
                       severity: standard.enforcement_level.clone(),
                   });
               }
           }
       }
   }

   // WASM entry point
   #[wasm_bindgen]
   pub fn validate_metadata(item_json: &str, standards_json: &str) -> String {
       let item: BinItem = serde_json::from_str(item_json).unwrap();
       let standards: Vec<MetadataStandard> = serde_json::from_str(standards_json).unwrap();
       
       let validator = MetadataValidator::new(standards);
       let result = validator.validate(&item);
       
       serde_json::to_string(&result).unwrap()
   }
   ```

**Acceptance Criteria:**
- [ ] MetadataValidator compiles to WASM
- [ ] Timestamp validation catches non-Zulu formats
- [ ] Auto-fix suggestions generated
- [ ] Performance < 10ms for typical items

#### Day 8-10: Integrate Layer 0 into The Bin Pipeline

**Tasks:**
1. Update Bin validation pipeline (from ARTIFACT_03)
   ```rust
   // bin/src/validation_pipeline.rs
   pub struct ValidationPipeline {
       layer0_metadata: MetadataValidator,
       layer1_runtime: RuntimeValidator,
       layer2_specification: SpecificationValidator,
       layer3_documentation: DocumentationValidator,
       layer4_decisions: DecisionValidator,
       layer5_propagation: PropagationValidator,
   }

   impl ValidationPipeline {
       pub fn validate(&self, item: &BinItem) -> PipelineResult {
           // Layer 0: Metadata Governance (FIRST - blocks all others)
           let metadata_result = self.layer0_metadata.validate(item);
           if !metadata_result.is_valid {
               return PipelineResult::Blocked {
                   layer: 0,
                   reason: "Metadata governance violations",
                   violations: metadata_result.violations,
               };
           }

           // Layer 1-5: Existing validation layers...
           let runtime_result = self.layer1_runtime.validate(item)?;
           let spec_result = self.layer2_specification.validate(item)?;
           // ... continue with other layers

           PipelineResult::Passed {
               warnings: metadata_result.warnings,
           }
       }
   }
   ```

2. Add API endpoints for violation management
   ```go
   // api/metadata/handler.go
   package metadata

   type MetadataHandler struct {
       db        *TiDBClient
       validator *wasm.MetadataValidator
   }

   // GET /api/metadata/violations
   func (h *MetadataHandler) ListViolations(w http.ResponseWriter, r *http.Request) {
       status := r.URL.Query().Get("status")  // open, resolved, suppressed
       violations, err := h.db.QueryViolations(status)
       if err != nil {
           http.Error(w, err.Error(), http.StatusInternalServerError)
           return
       }
       json.NewEncoder(w).Encode(violations)
   }

   // POST /api/metadata/violations/{id}/resolve
   func (h *MetadataHandler) ResolveViolation(w http.ResponseWriter, r *http.Request) {
       id := chi.URLParam(r, "id")
       userID := r.Context().Value("user_id").(string)
       
       err := h.db.UpdateViolationStatus(id, "resolved", userID)
       if err != nil {
           http.Error(w, err.Error(), http.StatusInternalServerError)
           return
       }
       w.WriteHeader(http.StatusOK)
   }

   // POST /api/metadata/violations/{id}/suppress
   func (h *MetadataHandler) SuppressViolation(w http.ResponseWriter, r *http.Request) {
       // Requires justification
       var req SuppressRequest
       json.NewDecoder(r.Body).Decode(&req)
       
       if req.Justification == "" {
           http.Error(w, "Justification required", http.StatusBadRequest)
           return
       }
       
       // Log suppression with justification
       h.db.SuppressViolation(chi.URLParam(r, "id"), req.Justification)
       w.WriteHeader(http.StatusOK)
   }
   ```

**Acceptance Criteria:**
- [ ] Layer 0 runs before all other validation layers
- [ ] ERROR-level violations block submissions
- [ ] WARNING-level violations allow but flag
- [ ] Violations persist to TiDB with timestamps

---

### Week 3: The Bin - Metadata Validation Layer (Part 2)

**Deliverable:** Complete metadata validation UI, auto-fix system, SSE streaming

#### Day 11-13: Build Auto-Fix Suggestion System

**Tasks:**
1. Implement batch auto-fix endpoint
   ```go
   // api/metadata/autofix.go
   type AutoFixService struct {
       validator *wasm.MetadataValidator
       db        *TiDBClient
   }

   type AutoFixPreview struct {
       ViolationID string `json:"violation_id"`
       Original    string `json:"original"`
       Suggested   string `json:"suggested"`
       CanAutoFix  bool   `json:"can_auto_fix"`
   }

   // POST /api/metadata/violations/preview-fixes
   func (s *AutoFixService) PreviewFixes(violations []string) []AutoFixPreview {
       var previews []AutoFixPreview
       
       for _, id := range violations {
           violation := s.db.GetViolation(id)
           preview := AutoFixPreview{
               ViolationID: id,
               Original:    violation.ViolationValue,
               Suggested:   violation.SuggestedFix,
               CanAutoFix:  s.canAutoFix(violation),
           }
           previews = append(previews, preview)
       }
       
       return previews
   }

   // POST /api/metadata/violations/apply-fixes
   func (s *AutoFixService) ApplyFixes(w http.ResponseWriter, r *http.Request) {
       var req ApplyFixesRequest
       json.NewDecoder(r.Body).Decode(&req)

       results := make([]FixResult, 0)
       for _, id := range req.ViolationIDs {
           result := s.applyFix(id)
           results = append(results, result)
       }

       json.NewEncoder(w).Encode(results)
   }
   ```

2. Implement Elm UI for violation management
   ```elm
   -- Metadata/ViolationList.elm
   module Metadata.ViolationList exposing (view, update, Msg)

   type alias Model =
       { violations : List Violation
       , selectedViolations : Set String
       , previewMode : Bool
       , fixPreviews : List AutoFixPreview
       }

   type Msg
       = LoadViolations (Result Http.Error (List Violation))
       | SelectViolation String Bool
       | SelectAll
       | PreviewFixes
       | PreviewLoaded (Result Http.Error (List AutoFixPreview))
       | ApplySelectedFixes
       | FixesApplied (Result Http.Error (List FixResult))

   view : Model -> Html Msg
   view model =
       div [ class "violation-list" ]
           [ viewHeader model
           , viewFilters
           , viewViolationTable model
           , viewBulkActions model
           , if model.previewMode then
               viewFixPreviews model.fixPreviews
             else
               text ""
           ]

   viewViolationTable : Model -> Html Msg
   viewViolationTable model =
       table [ class "violations" ]
           [ thead []
               [ tr []
                   [ th [] [ checkbox (SelectAll) (allSelected model) ]
                   , th [] [ text "Resource" ]
                   , th [] [ text "Standard" ]
                   , th [] [ text "Value" ]
                   , th [] [ text "Suggested Fix" ]
                   , th [] [ text "Severity" ]
                   , th [] [ text "Actions" ]
                   ]
               ]
           , tbody [] (List.map (viewViolationRow model.selectedViolations) model.violations)
           ]

   viewViolationRow : Set String -> Violation -> Html Msg
   viewViolationRow selected violation =
       tr [ class (severityClass violation.severity) ]
           [ td [] [ checkbox (SelectViolation violation.id) (Set.member violation.id selected) ]
           , td [] [ text violation.resourceId ]
           , td [] [ text violation.standardId ]
           , td [ class "violation-value" ] [ code [] [ text violation.violationValue ] ]
           , td [ class "suggested-fix" ] [ code [] [ text violation.suggestedFix ] ]
           , td [] [ severityBadge violation.severity ]
           , td []
               [ button [ onClick (ApplyFix violation.id) ] [ text "Fix" ]
               , button [ onClick (Suppress violation.id) ] [ text "Suppress" ]
               ]
           ]
   ```

**Acceptance Criteria:**
- [ ] Batch fix preview shows before/after
- [ ] Single-click apply for auto-fixable violations
- [ ] Justification required for suppression
- [ ] UI updates via SSE on changes

#### Day 14-15: Implement Violation SSE Streaming

**Tasks:**
1. Add violation events to SSE broadcaster
   ```go
   // api/sse/violation_events.go
   const (
       EventViolationDetected = "violation:detected"
       EventViolationResolved = "violation:resolved"
       EventComplianceUpdate  = "compliance:update"
   )

   func (b *SSEBroadcaster) BroadcastViolation(violation Violation) {
       b.Broadcast(EventViolationDetected, ViolationEvent{
           Violation:      violation,
           ProjectID:      violation.ProjectID,
           ComplianceRate: b.calculateComplianceRate(violation.ProjectID),
       })
   }
   ```

2. Subscribe Elm frontend to violation events
   ```elm
   -- Metadata/Subscriptions.elm
   subscriptions : Model -> Sub Msg
   subscriptions model =
       SSE.subscribe
           { url = "/api/events"
           , events =
               [ ("violation:detected", ViolationDetected)
               , ("violation:resolved", ViolationResolved)
               , ("compliance:update", ComplianceUpdated)
               ]
           }
   ```

**Acceptance Criteria:**
- [ ] New violations appear in real-time
- [ ] Resolution reflects immediately
- [ ] Compliance rate updates live

---

### Week 4: AI Cortex Infrastructure

**Deliverable:** Token tracking, context window monitoring, estimation AI (Phi-3 Mini)

#### Day 16-17: Deploy Token Usage Tracking

**Tasks:**
1. Implement token tracking middleware (from AI_CORTEX_MANAGEMENT_DASHBOARD)
   ```go
   // api/ai/token_tracker.go
   package ai

   type TokenTracker struct {
       db    *TiDBClient
       redis *redis.Client
   }

   func (t *TokenTracker) RecordUsage(ctx context.Context, usage TokenUsage) error {
       // Fast path: Update Redis counters
       pipe := t.redis.Pipeline()
       
       dailyKey := fmt.Sprintf("tokens:daily:%s:%s", usage.ProjectID, time.Now().Format("2006-01-02"))
       monthlyKey := fmt.Sprintf("tokens:monthly:%s:%s", usage.ProjectID, time.Now().Format("2006-01"))
       
       pipe.IncrBy(ctx, dailyKey, int64(usage.InputTokens+usage.OutputTokens))
       pipe.Expire(ctx, dailyKey, 48*time.Hour)
       pipe.IncrBy(ctx, monthlyKey, int64(usage.InputTokens+usage.OutputTokens))
       pipe.Expire(ctx, monthlyKey, 32*24*time.Hour)
       
       _, err := pipe.Exec(ctx)
       if err != nil {
           return err
       }

       // Slow path: Persist to TiDB for analytics
       go t.persistToTiDB(usage)
       
       return nil
   }

   func (t *TokenTracker) GetBudgetStatus(ctx context.Context, projectID string) (*BudgetStatus, error) {
       dailyKey := fmt.Sprintf("tokens:daily:%s:%s", projectID, time.Now().Format("2006-01-02"))
       monthlyKey := fmt.Sprintf("tokens:monthly:%s:%s", projectID, time.Now().Format("2006-01"))
       
       daily, _ := t.redis.Get(ctx, dailyKey).Int64()
       monthly, _ := t.redis.Get(ctx, monthlyKey).Int64()
       
       // Get budget limits from TiDB
       budget := t.getBudgetLimits(projectID)
       
       return &BudgetStatus{
           DailyUsed:        daily,
           DailyLimit:       budget.DailyLimit,
           DailyRemaining:   budget.DailyLimit - daily,
           MonthlyUsed:      monthly,
           MonthlyLimit:     budget.MonthlyLimit,
           MonthlyRemaining: budget.MonthlyLimit - monthly,
           EstimatedCost:    t.calculateCost(monthly),
       }, nil
   }
   ```

**Acceptance Criteria:**
- [ ] Token usage recorded per request
- [ ] Daily/monthly counters in Redis
- [ ] Budget status queryable

#### Day 18-19: Build Context Window Monitoring

**Tasks:**
1. Implement context window gauge (Rust + Elm)
   ```rust
   // ai/context_monitor.rs
   pub struct ContextWindowMonitor {
       max_tokens: usize,
       current_tokens: usize,
       history: VecDeque<TokenSnapshot>,
   }

   impl ContextWindowMonitor {
       pub fn estimate_tokens(&self, text: &str) -> usize {
           // Fast tiktoken estimation
           tiktoken::cl100k_base()
               .encode_with_special_tokens(text)
               .len()
       }

       pub fn get_status(&self) -> ContextWindowStatus {
           let percentage = (self.current_tokens as f64 / self.max_tokens as f64) * 100.0;
           
           ContextWindowStatus {
               percentage_full: percentage,
               tokens_used: self.current_tokens,
               tokens_remaining: self.max_tokens - self.current_tokens,
               warning_level: self.get_warning_level(percentage),
           }
       }

       fn get_warning_level(&self, percentage: f64) -> WarningLevel {
           match percentage {
               p if p < 70.0 => WarningLevel::Normal,
               p if p < 85.0 => WarningLevel::Caution,
               p if p < 95.0 => WarningLevel::Warning,
               _ => WarningLevel::Critical,
           }
       }
   }
   ```

2. Create Elm gauge component
   ```elm
   -- AI/ContextWindowGauge.elm
   view : Model -> Html Msg
   view model =
       div [ class "context-window-gauge" ]
           [ radialGauge model.percentageFull model.warningLevel
           , div [ class "gauge-stats" ]
               [ stat "Used" (formatTokens model.tokensUsed)
               , stat "Remaining" (formatTokens model.tokensRemaining)
               , stat "% Full" (String.fromFloat model.percentageFull ++ "%")
               ]
           , warningBanner model.warningLevel
           ]

   radialGauge : Float -> WarningLevel -> Html Msg
   radialGauge percentage level =
       svg [ viewBox "0 0 200 200", width 200, height 200 ]
           [ circle [ cx "100", cy "100", r "80", fill "none", stroke "#e0e0e0", strokeWidth "20" ] []
           , circle 
               [ cx "100", cy "100", r "80"
               , fill "none"
               , stroke (warningColor level)
               , strokeWidth "20"
               , strokeDasharray (String.fromFloat (percentage * 5.03) ++ " 503")
               , transform "rotate(-90 100 100)"
               ] []
           , text_ [ x "100", y "110", textAnchor "middle", fontSize "32" ]
               [ text (String.fromFloat percentage ++ "%") ]
           ]
   ```

**Acceptance Criteria:**
- [ ] Context window percentage visible
- [ ] Warning colors at 70/85/95%
- [ ] Real-time updates via SSE

#### Day 20: Deploy Estimation AI (Phi-3 Mini)

**Tasks:**
1. Configure Phi-3 Mini for lightweight estimation
   ```go
   // api/ai/estimation_ai.go
   package ai

   type EstimationAI struct {
       client *ollama.Client
       model  string  // "phi3:mini"
   }

   func NewEstimationAI() *EstimationAI {
       return &EstimationAI{
           client: ollama.NewClient("http://localhost:11434"),
           model:  "phi3:mini",
       }
   }

   func (e *EstimationAI) EstimateTokens(text string) (int, error) {
       // Use DevGuide's native tiktoken for speed
       // Only fall back to Phi-3 for complex estimation
       return tiktoken.CountTokens(text), nil
   }

   func (e *EstimationAI) SuggestTools(context string) ([]ToolSuggestion, error) {
       prompt := fmt.Sprintf(`Given this context: %s
       
       Suggest which DevGuide tools would be most helpful.
       Return JSON array: [{"tool": "name", "reason": "why"}]`, context)

       resp, err := e.client.Generate(e.model, prompt)
       if err != nil {
           return nil, err
       }

       var suggestions []ToolSuggestion
       json.Unmarshal([]byte(resp), &suggestions)
       return suggestions, nil
   }

   func (e *EstimationAI) PredictBurnRate(history []TokenSnapshot) float64 {
       // Simple linear regression for burn rate
       // Phi-3 only used for complex patterns
       if len(history) < 7 {
           return 0
       }
       
       avgDaily := calculateAverage(history)
       return avgDaily
   }
   ```

2. Configure request routing
   ```go
   // api/ai/router.go
   type AIRouter struct {
       mainAI       *ClaudeProvider
       estimationAI *EstimationAI
       devguideTools *DevGuideTools
   }

   func (r *AIRouter) Route(ctx context.Context, req AIRequest) (AIResponse, error) {
       // 1. Check if DevGuideTools can handle (90% of cases)
       if tool := r.extractToolName(req); tool != "" {
           if r.devguideTools.CanHandle(tool) {
               return r.devguideTools.Execute(tool, req.Params)
           }
       }

       // 2. Check if estimation AI can handle
       if r.isEstimationTask(req) {
           return r.estimationAI.Handle(ctx, req)
       }

       // 3. Only use main AI for complex reasoning
       return r.mainAI.Complete(ctx, req)
   }
   ```

**Acceptance Criteria:**
- [ ] Phi-3 Mini deployed (local or Azure)
- [ ] Tool suggestions under 100ms
- [ ] Main AI only used for complex tasks
- [ ] 90%+ tool calls handled by DevGuideTools

---

## Month 2: Templates & Integration (Weeks 5-8)

### Week 5: Extension Orchestration MVP

**Deliverable:** Extension proxy layer, dashboard builder UI, community repository

#### Day 21-22: Build Extension Proxy Layer (Go)

**Tasks:**
1. Implement command interceptor (from VSCODE_EXTENSION_ORCHESTRATION_SYSTEM)
   ```go
   // api/extension/proxy.go
   package extension

   type ExtensionProxy struct {
       extensionID      string
       config           *ProxyConfig
       commandHooks     map[string]CommandHook
       transformPipeline *TransformPipeline
       analytics        *AnalyticsCollector
   }

   func (ep *ExtensionProxy) InterceptCommand(
       ctx context.Context, 
       commandID string, 
       args []interface{},
   ) (interface{}, error) {
       // Log command
       ep.analytics.LogCommand(CommandLog{
           ExtensionID: ep.extensionID,
           CommandID:   commandID,
           Args:        args,
           Timestamp:   time.Now().UTC(),
       })

       // Apply pre-hook
       if hook, exists := ep.commandHooks[commandID]; exists && hook.PreHook != nil {
           args, _ = hook.PreHook(ctx, args)
       }

       // Apply input transforms
       args, _ = ep.transformPipeline.TransformInput(args)

       // Inject JIT data
       args = ep.injectJITData(ctx, commandID, args)

       // Execute native command
       result, err := ep.executeNativeCommand(ctx, commandID, args)
       if err != nil {
           return nil, err
       }

       // Apply post-hook
       if hook, exists := ep.commandHooks[commandID]; exists && hook.PostHook != nil {
           result, _ = hook.PostHook(ctx, result)
       }

       // Apply output transforms
       result, _ = ep.transformPipeline.TransformOutput(result)

       return result, nil
   }

   func (ep *ExtensionProxy) injectJITData(ctx context.Context, commandID string, args []interface{}) []interface{} {
       for _, source := range ep.config.JITDataSources {
           data, err := ep.fetchDataSource(ctx, source)
           if err != nil {
               continue
           }
           args = append(args, map[string]interface{}{
               source.Name: data,
           })
       }
       return args
   }
   ```

**Acceptance Criteria:**
- [ ] Commands intercepted transparently
- [ ] Pre/post hooks execute
- [ ] JIT data injected

#### Day 23-24: Create Dashboard Builder UI

**Tasks:**
1. Implement visual dashboard designer (Elm)
   ```elm
   -- Extension/DashboardBuilder.elm
   module Extension.DashboardBuilder exposing (view, update, Model, Msg)

   type alias Model =
       { extensionID : String
       , dashboardName : String
       , components : List DashboardComponent
       , transformRules : List TransformRule
       , jitDataSources : List DataSource
       , preview : Maybe PreviewState
       }

   type DashboardComponent
       = InputControls InputConfig
       | TransformPipeline PipelineConfig
       | OutputPreview OutputConfig
       | CustomPanel CustomPanelConfig

   type Msg
       = AddComponent DashboardComponent
       | RemoveComponent Int
       | ReorderComponents Int Int
       | UpdateTransform Int TransformRule
       | AddJITDataSource DataSource
       | TogglePreview
       | SaveDashboard
       | PublishToCommunity

   view : Model -> Html Msg
   view model =
       div [ class "dashboard-builder" ]
           [ viewToolbar model
           , div [ class "builder-layout" ]
               [ viewComponentPalette
               , viewCanvas model.components
               , viewPropertiesPanel model
               ]
           , if model.preview /= Nothing then
               viewLivePreview model.preview
             else
               text ""
           ]

   viewComponentPalette : Html Msg
   viewComponentPalette =
       div [ class "component-palette" ]
           [ h3 [] [ text "Components" ]
           , draggable (InputControls defaultInput) "Input Controls"
           , draggable (TransformPipeline defaultPipeline) "Transform Pipeline"
           , draggable (OutputPreview defaultOutput) "Output Preview"
           , draggable (CustomPanel defaultCustom) "Custom Panel"
           ]
   ```

**Acceptance Criteria:**
- [ ] Drag-and-drop components
- [ ] Live preview functional
- [ ] Save to TiDB

#### Day 25: Implement Community Repository (MVP Scope)

> ⚠️ **MVP SCOPE REDUCTION:** Day 25 implements browse + install only. Rating and review features deferred to Week 8.

**Tasks:**
1. Build community dashboard API (MVP)
   ```go
   // api/extension/community.go
   type CommunityRepository struct {
       db *TiDBClient
   }

   // GET /api/extensions/community
   func (r *CommunityRepository) ListDashboards(w http.ResponseWriter, req *http.Request) {
       extensionID := req.URL.Query().Get("extension_id")
       sortBy := req.URL.Query().Get("sort")  // "rating", "downloads", "recent"
       
       dashboards, err := r.db.QueryExtensionDashboards(extensionID, sortBy)
       if err != nil {
           http.Error(w, err.Error(), http.StatusInternalServerError)
           return
       }
       
       json.NewEncoder(w).Encode(dashboards)
   }

   // POST /api/extensions/community/publish
   func (r *CommunityRepository) PublishDashboard(w http.ResponseWriter, req *http.Request) {
       var dashboard ExtensionDashboard
       json.NewDecoder(req.Body).Decode(&dashboard)
       
       // Validate dashboard
       if err := r.validateDashboard(&dashboard); err != nil {
           http.Error(w, err.Error(), http.StatusBadRequest)
           return
       }
       
       // Create version record
       version := DashboardVersion{
           DashboardID:   dashboard.ID,
           Version:       dashboard.Version,
           Specification: dashboard.Specification,
           PublishedAt:   time.Now().UTC(),
       }
       
       r.db.CreateDashboardVersion(&version)
       w.WriteHeader(http.StatusCreated)
   }

   // NOTE: Rating and review endpoints deferred to Week 8
   ```

2. Create Elm community browser
   ```elm
   -- Extension/CommunityBrowser.elm
   view : Model -> Html Msg
   view model =
       div [ class "community-browser" ]
           [ viewSearchBar model.searchQuery
           , viewFilters model.filters
           , viewDashboardGrid model.dashboards
           , viewPagination model.pagination
           ]

   viewDashboardCard : ExtensionDashboard -> Html Msg
   viewDashboardCard dashboard =
       div [ class "dashboard-card", onClick (PreviewDashboard dashboard.id) ]
           [ h4 [] [ text dashboard.dashboardName ]
           , p [] [ text dashboard.description ]
           , div [ class "card-footer" ]
               [ starRating dashboard.communityRating
               , span [] [ text (String.fromInt dashboard.downloadCount ++ " downloads") ]
               , if dashboard.isVerified then
                   verifiedBadge
                 else
                   text ""
               ]
           ]
   ```

**Acceptance Criteria:**
- [ ] Browse community dashboards
- [ ] Install with one click
- [ ] Version history visible
- [ ] Rating/review deferred to Week 8

---

### Week 6: Vibe Coding Template (Updated)

**Deliverable:** Complete Vibe Coding template with metadata compliance and AI Cortex

#### Day 26-28: Core Dashboards with Governance Integration

**Tasks:**
1. Update Architecture Canvas with metadata validation
   ```elm
   -- VibeCode/Architecture.elm (updated)
   module VibeCode.Architecture exposing (view, update)

   type alias Model =
       { architectureStyle : ArchitectureStyle
       , components : List Component
       , relationships : List Relationship
       , metadataCompliance : ComplianceStatus  -- NEW
       , aiCortexStatus : AIStatus              -- NEW
       }

   view : Model -> Html Msg
   view model =
       div [ class "architecture-canvas" ]
           [ viewArchitectureSelector model.architectureStyle
           , viewComponentPalette
           , viewCanvas model.components model.relationships
           , viewAIAssistant
           -- NEW: Embedded panels
           , viewMetadataCompliancePanel model.metadataCompliance
           , viewAICortexMiniPanel model.aiCortexStatus
           ]

   viewMetadataCompliancePanel : ComplianceStatus -> Html Msg
   viewMetadataCompliancePanel status =
       div [ class "compliance-panel embedded" ]
           [ h4 [] [ text "Metadata Compliance" ]
           , complianceGauge status.rate
           , if status.violations > 0 then
               violationAlert status.violations
             else
               text ""
           ]

   viewAICortexMiniPanel : AIStatus -> Html Msg
   viewAICortexMiniPanel status =
       div [ class "ai-cortex-mini" ]
           [ h4 [] [ text "AI Context" ]
           , miniContextGauge status.percentageFull
           , tokenCounter status.tokensRemaining
           ]
   ```

2. Update Tech Stack Matrix with LPDS naming suggestions
   ```elm
   -- VibeCode/TechStack.elm (updated)
   viewCompatibilityWarnings : Model -> Html Msg
   viewCompatibilityWarnings model =
       div [ class "warnings-panel" ]
           [ if List.length model.languages > 3 then
               warningBanner "Max 3 languages per repo (Atomic Standard)"
             else
               text ""
           , viewNamingConventionSuggestions model  -- NEW
           ]

   viewNamingConventionSuggestions : Model -> Html Msg
   viewNamingConventionSuggestions model =
       let
           suggestedRepoName = generateLPDSRepoName model
       in
       div [ class "naming-suggestion" ]
           [ h5 [] [ text "Suggested Repository Name (LPDS)" ]
           , code [] [ text suggestedRepoName ]
           , copyButton suggestedRepoName
           ]
   ```

**Acceptance Criteria:**
- [ ] Architecture Canvas renders with governance panel
- [ ] Tech Stack Matrix enforces 3-language limit
- [ ] LPDS naming suggestions provided
- [ ] AI Cortex status visible

#### Day 29-30: Propagation Rules with Metadata Triggers

**Tasks:**
1. Add metadata-aware propagation rules
   ```yaml
   # templates/vibe-coding/template.yaml (updated)
   propagation_rules:
     - when: tech_stack.database_changed
       propagate_to:
         - deployment.config
         - security.auth_config
     
     # NEW: Metadata propagation rules
     - when: metadata.standard_added
       propagate_to:
         - all_dashboards  # Re-validate all
     
     - when: metadata.timestamp_format_changed
       propagate_to:
         - api_documentation
         - audit_logs
   ```

**Acceptance Criteria:**
- [ ] Tech stack changes trigger deployment update
- [ ] Metadata standard changes trigger re-validation
- [ ] Circular dependencies detected

---

### Week 7: Data Science & Note Taking Templates (Updated)

**Deliverable:** Two additional templates with governance integration

#### Day 31-33: Data Science Template

**Tasks:**
1. Create template with AI Cortex integration
   ```yaml
   # templates/data-science/template.yaml (updated)
   name: Data Science Workflow
   version: 1.0.0
   
   dashboards:
     - id: data-sources
       elm_module: DataScience.DataSources
       
     - id: experiments
       elm_module: DataScience.Experiments
       
     - id: models
       elm_module: DataScience.Models
       
     - id: ai-cortex  # NEW
       elm_module: AI.CortexPanel
       embeddable: true
   
   guardrails:
     - id: data-privacy
       rules:
         - no_pii_in_git
         - encryption_required_for_datasets
     
     # NEW: Metadata guardrails
     - id: timestamp-compliance
       rules:
         - all_timestamps_zulu
         - experiment_dates_iso8601
   
   ai_cortex_config:
     default_budget_daily: 50000    # 50K tokens/day
     context_window_warning: 80     # Warn at 80%
     estimation_ai_enabled: true
   ```

**Acceptance Criteria:**
- [ ] Data sources can be configured
- [ ] Experiments tracked with timestamps
- [ ] Guardrails prevent PII leaks
- [ ] AI Cortex panel available

#### Day 34-35: Note Taking Template

**Tasks:**
1. Create minimal template with metadata compliance
   ```yaml
   # templates/note-taking/template.yaml (updated)
   name: Note Taking System
   version: 1.0.0
   
   dashboards:
     - id: notes
       elm_module: NoteTaking.Notes
       
     - id: tags
       elm_module: NoteTaking.Tags
   
   guardrails:
     - id: markdown-syntax
       severity: warning
     
     # NEW: File naming guardrail
     - id: note-naming
       rules:
         - note_files_use_lpds_pattern
   
   metadata_config:
     file_naming_pattern: "note_<topic>-<date>_ver-<version>.md"
     timestamp_format: "iso8601_zulu"
   ```

**Acceptance Criteria:**
- [ ] Markdown rendering works
- [ ] Notes follow LPDS naming
- [ ] Search finds notes

---

### Week 8: Template Testing & Community Governance (Updated)

**Deliverable:** All templates tested, community contribution process with governance

#### Day 36-38: Template Testing with Compliance Verification

**Tasks:**
1. Create compliance test suite
   ```bash
   #!/bin/bash
   # test/compliance_check.sh

   # Test all templates for metadata compliance
   for template in templates/*/; do
       echo "Testing: $template"
       
       # Check timestamp formats
       grep -rE '\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}[^Z]' "$template" && \
           echo "ERROR: Non-Zulu timestamps found"
       
       # Check file naming
       for file in "$template"*.{elm,go,rs}; do
           if [[ ! "$file" =~ ^[a-z]+_[a-z]+-[a-z]+.*\.[a-z]+$ ]]; then
               echo "WARNING: File does not follow LPDS: $file"
           fi
       done
       
       # Check version strings
       grep -rE 'version.*[^0-9]\d+\.\d+[^.\d]' "$template" && \
           echo "WARNING: Non-SemVer versions found"
   done
   ```

**Acceptance Criteria:**
- [ ] All 3 templates pass compliance
- [ ] Issues documented
- [ ] Fixes deployed

#### Day 39-40: Community Governance with Metadata Requirements

**Tasks:**
1. Update contribution process
   ```markdown
   # CONTRIBUTING.md (updated)
   
   ## Template Contribution Process
   
   ### Metadata Compliance Requirements (NEW)
   
   All contributed templates MUST:
   
   1. **Timestamps**: Use ISO 8601 with Z suffix (e.g., `2026-01-01T14:30:45Z`)
   2. **File Naming**: Follow LPDS pattern where possible
   3. **Versions**: Use Semantic Versioning 2.0.0
   4. **Branch Naming**: `<type>/<TICKET>-<description>`
   
   ### Pre-Submission Checklist
   
   - [ ] Run `make compliance-check` - all tests pass
   - [ ] No non-Zulu timestamps in any files
   - [ ] template.yaml includes `metadata_config` section
   - [ ] AI Cortex budget limits defined
   ```

**Acceptance Criteria:**
- [ ] Contribution docs include metadata requirements
- [ ] Submission API validates metadata compliance
- [ ] Non-compliant submissions rejected with suggestions

---

## Month 3: Polish & Beta Launch (Weeks 9-12)

### Week 9: Offline Architecture

**Deliverable:** PWA works offline with service worker (unchanged from v2.0)

#### Day 41-45: Service Worker & Background Sync

(Tasks unchanged from v2.0 - see original implementation)

**Acceptance Criteria:**
- [ ] App loads offline
- [ ] Changes queue in IndexedDB
- [ ] Sync works when reconnected

---

### Week 10: Migration System & Governance Documentation

**Deliverable:** Version upgrade wizard + complete governance documentation

#### Day 46-48: Migration Wizard (unchanged)

(Tasks unchanged from v2.0)

#### Day 49-50: Governance Documentation (NEW)

**Tasks:**
1. Create Metadata Standards Guide
   ```markdown
   # docs/metadata-standards-guide.md
   
   ## Temporal Standards
   
   All timestamps in DevGuide Cockpit must use ISO 8601 format with 
   Zulu (UTC) timezone suffix.
   
   ### Acceptable Formats
   
   | Context | Format | Example |
   |---------|--------|---------|
   | API Responses | Extended | `2026-01-01T14:30:45.123Z` |
   | File Names | Basic | `20260101T143045Z` |
   | Database | TIMESTAMP(6) | `2026-01-01 14:30:45.123456` |
   
   ### Common Mistakes
   
   ❌ `2026-01-01T14:30:45+00:00` - Use Z instead of +00:00
   ❌ `2026-01-01 14:30:45` - Missing T and timezone
   ❌ `01/01/2026` - Wrong format entirely
   
   ### Auto-Fix
   
   The Bin will automatically suggest fixes for common violations.
   ```

2. Create Compliance Dashboard Tutorial
   ```markdown
   # docs/compliance-dashboard-tutorial.md
   
   ## Using the Metadata Governance Dashboard
   
   ### Quick Start
   
   1. Open Master Control → Governance Panel
   2. View compliance rate (target: >98%)
   3. Click violations to see details
   4. Use "Fix All" for auto-fixable issues
   
   ### Understanding Violation Severities
   
   | Severity | Action | Blocks Commit |
   |----------|--------|---------------|
   | ERROR | Must fix | Yes |
   | WARNING | Should fix | No |
   | INFO | Can ignore | No |
   ```

**Acceptance Criteria:**
- [ ] Metadata standards guide published
- [ ] Compliance tutorial with screenshots
- [ ] Troubleshooting section complete

---

### Week 11: Documentation & Governance Polish

**Deliverable:** Complete documentation + governance performance optimization

#### Day 51-53: User Documentation (updated)

**Tasks:**
1. Add governance sections to all guides
2. Include AI Cortex usage guide
3. Document extension orchestration

**Acceptance Criteria:**
- [ ] All docs include governance context
- [ ] AI Cortex tutorial complete
- [ ] Extension setup guide ready

#### Day 54-55: Governance Performance Optimization (NEW)

**Tasks:**
1. Optimize validation performance
   ```go
   // api/metadata/cache.go
   type StandardsCache struct {
       standards []MetadataStandard
       compiled  map[string]*regexp.Regexp
       mutex     sync.RWMutex
       ttl       time.Duration
       lastLoad  time.Time
   }

   func (c *StandardsCache) GetCompiledRegex(standardID string) *regexp.Regexp {
       c.mutex.RLock()
       defer c.mutex.RUnlock()
       
       // Reload if TTL expired
       if time.Since(c.lastLoad) > c.ttl {
           go c.reload()
       }
       
       return c.compiled[standardID]
   }
   ```

2. Add bulk violation resolution
   ```go
   // api/metadata/bulk_operations.go
   func (h *MetadataHandler) BulkResolve(w http.ResponseWriter, r *http.Request) {
       var req BulkResolveRequest
       json.NewDecoder(r.Body).Decode(&req)
       
       // Use batch update for performance
       tx, _ := h.db.BeginTx(r.Context(), nil)
       defer tx.Rollback()
       
       stmt, _ := tx.Prepare(`
           UPDATE metadata_violations 
           SET status = 'resolved', resolved_at = NOW(), resolved_by = ?
           WHERE id = ?
       `)
       
       for _, id := range req.ViolationIDs {
           stmt.Exec(req.UserID, id)
       }
       
       tx.Commit()
       w.WriteHeader(http.StatusOK)
   }
   ```

**Acceptance Criteria:**
- [ ] Validation under 50ms (p99)
- [ ] Bulk resolve handles 1000+ violations
- [ ] Cache reduces DB queries by 90%

---

### Week 12: Beta Launch Preparation

**Deliverable:** Production deployment + 100 beta users

#### Day 56-58: Production Deployment (updated)

**Tasks:**
1. Deploy all new infrastructure
   ```yaml
   # infrastructure/production.yaml
   services:
     # Existing services...
     
     # NEW: Clock sync monitoring
     clock-sync-monitor:
       image: devguide/clock-sync:v1
       environment:
         - NTP_SERVER=time.aws.com
         - CHECK_INTERVAL=30s
       deploy:
         replicas: 1
     
     # NEW: Estimation AI (Phi-3 Mini)
     estimation-ai:
       image: ollama/ollama:latest
       volumes:
         - ./models:/root/.ollama/models
       command: ["serve"]
       deploy:
         resources:
           limits:
             memory: 4G
   ```

2. Configure monitoring
   ```yaml
   # prometheus/alerts.yml (updated)
   groups:
     - name: metadata_governance
       rules:
         - alert: ComplianceRateLow
           expr: metadata_compliance_rate < 0.95
           for: 5m
           annotations:
             summary: "Compliance rate below 95% for {{ $labels.project_id }}"
         
         - alert: ViolationBacklog
           expr: metadata_violations_open > 100
           annotations:
             summary: "Over 100 open violations"
     
     - name: ai_cortex
       rules:
         - alert: ContextWindowCritical
           expr: ai_context_window_percent > 95
           annotations:
             summary: "Context window >95% full"
         
         - alert: TokenBudgetExhausted
           expr: ai_token_budget_remaining < 5000
           annotations:
             summary: "Token budget critically low"
   ```

**Acceptance Criteria:**
- [ ] Production URL accessible
- [ ] All new services deployed
- [ ] Monitoring dashboards live

#### Day 59: Beta User Onboarding

**Tasks:**
1. Send beta invitations (100 users)
2. Include governance onboarding guide
3. Setup support channel (Discord)

**Acceptance Criteria:**
- [ ] 100 invitations sent
- [ ] 50+ users signed up
- [ ] Support channel active

#### Day 60: Launch Day 🚀

**Tasks:**
1. Monitor production metrics
2. Track compliance rate
3. Respond to user feedback
4. Fix critical bugs
5. Celebrate!

**Success Metrics:**
- [ ] System uptime > 99%
- [ ] Error rate < 0.1%
- [ ] p95 latency < 200ms
- [ ] 50+ active users
- [ ] Metadata compliance rate > 95%

---

## Post-MVP Roadmap (Months 4-6)

### Month 4: Advanced Features

**Features:**
- Multi-provider AI routing (Claude + OpenAI + local)
- Real-time collaboration (multiple users editing)
- GitHub integration (auto-sync to Git with LPDS naming)
- External API guardrails (call external validators)
- **Timestomping detection UI** (security forensics)

### Month 5: Enterprise Features

**Features:**
- Organization support (teams, SSO)
- Advanced RBAC (role-based access control)
- Audit log UI with **temporal forensics**
- Custom guardrail editor (visual rule builder)
- **Clock drift monitoring dashboard**

### Month 6: Ecosystem Growth

**Features:**
- Template marketplace (community submissions)
- Plugin system (custom dashboard modules)
- Webhook integrations (Slack, Discord, etc.)
- Mobile app (iOS, Android)
- **Extension dashboard marketplace**

---

## Team Structure & Roles

### Core Team (7 engineers) - UPDATED

> ⚠️ **CRITICAL:** The Observability Engineer (7th hire) must be onboarded and ready before Week 1, Day 1. Budget 2 weeks for hiring/contracting and 1 week for project context ramp-up BEFORE the 3-month timeline begins.

**Elm Frontend Team (2 engineers):**
- **Elm Lead**: Dashboard architecture, template system, governance UI
- **UI/UX Engineer**: Design system, accessibility, violation workflows

**Go Backend Team (2 engineers):**
- **API Lead**: REST API, SSE, TiDB integration, extension proxy
- **AI Integration Engineer**: AI provider abstraction, context management, estimation AI

**Rust Kernel Team (1 engineer):**
- **Validation Lead**: The Bin logic, guardrails, WASM compilation, MetadataValidator

**DevOps Team (2 engineers) - EXPANDED:**
- **Infrastructure Lead**: Azure deployment, CI/CD, monitoring, TiDB
- **Observability Engineer (NEW)**: Clock sync, metrics, alerting, Phi-3 deployment

### Responsibilities Matrix (Updated)

| Role | Month 1 | Month 2 | Month 3 |
|------|---------|---------|---------|
| **Elm Lead** | The Bin UI, Governance Panel | Template system, Extension Builder | Wizard UI, Polish |
| **UI/UX Engineer** | Violation UI, Compliance Gauge | Template UIs, Community Browser | Documentation |
| **API Lead** | HTTP API, SSE, Metadata API | Extension Proxy, Community API | Migration API |
| **AI Integration** | Token Tracker, Estimation AI | AI-assisted wizard, Context Monitor | Offline AI cache |
| **Validation Lead** | MetadataValidator, Layer 0 | Template Guardrails | Breaking change detection |
| **Infrastructure Lead** | CI/CD, TiDB setup, 11 tables | Extension infra | Production deployment |
| **Observability Engineer** | Clock sync, NTP monitoring | AI Cortex metrics | Alerting, dashboards |

---

## Risk Management (Updated)

### Critical Risks

**Risk 1: Clock Synchronization Drift** (NEW)
- **Probability**: Medium
- **Impact**: High (temporal integrity compromised)
- **Mitigation**: AWS Time Sync, chrony monitoring, alerts at 10ms drift

**Risk 2: Estimation AI Latency**
- **Probability**: Medium
- **Impact**: Medium
- **Mitigation**: Fallback to tiktoken, cache predictions, timeout at 100ms

**Risk 3: Extension Proxy Overhead**
- **Probability**: Medium
- **Impact**: Medium
- **Mitigation**: Benchmark all hooks, optimize hot paths, circuit breaker

**Risk 4: Metadata Validation Performance**
- **Probability**: Low
- **Impact**: High (blocks all commits)
- **Mitigation**: Pre-compiled regexes, caching, async violation storage

**Risk 5: Community Dashboard Security**
- **Probability**: Medium
- **Impact**: High
- **Mitigation**: Sandboxed JavaScript execution, rate limiting, code review

**Risk 6: Team Onboarding Delays** (NEW in v2.1.1)
- **Probability**: Medium
- **Impact**: High (delays entire project start)
- **Mitigation**: 
  - Begin 7th engineer hiring 4 weeks before project start
  - Have fallback candidates identified
  - Infrastructure Lead can absorb observability tasks temporarily if needed
  - Provide Elm/Rust onboarding documentation for new team members

### Mitigation Strategies (Updated)

**Weekly Risk Reviews:**
- Every Friday: Team discusses blockers
- Clock sync status review
- AI Cortex budget check
- Compliance rate tracking

---

## Budget & Resources (Updated)

### Infrastructure Costs

**MVP (Updated):**
- TiDB Cloud Starter: $0/month
- Azure Free Credits: $200/month
- Domain (devguide.app): $1/month
- **Clock sync service (AWS Time Sync): $20/month** (NEW)
- **Estimation AI hosting (Phi-3 Mini): $30/month** (NEW)
- **Total: ~$150/month**

**Production (Updated):**
- TiDB Cloud Scalable: ~$800/month
- Azure App Service: ~$200/month
- Azure CDN: ~$100/month
- **NTP/PTP monitoring: ~$100/month** (NEW)
- **Phi-3 Mini scaling: ~$100/month** (NEW)
- **Total: ~$1,300/month**

### Tooling

**Free Tier:**
- GitHub (free for public repos)
- Playwright (open source)
- Prometheus + Grafana (self-hosted)
- chrony (open source)

**Paid:**
- Anthropic Claude API: ~$100/month for 100 beta users
- Ollama (Phi-3 Mini): Self-hosted, free

### Total Budget Summary

| Phase | v2.0 | v2.1.0 | Delta |
|-------|------|--------|-------|
| MVP | $100/mo | $150/mo | +$50 |
| Production | $1,100/mo | $1,300/mo | +$200 |

---

## Metrics & KPIs (Updated)

### Technical Metrics

**Performance:**
- Dashboard load time: < 500ms (p95)
- API latency: < 200ms (p95)
- TiDB query time: < 50ms (p95)
- **Metadata validation: < 50ms (p99)** (NEW)
- **Estimation AI response: < 100ms (p95)** (NEW)

**Reliability:**
- Uptime: > 99.9%
- Error rate: < 0.1%
- Data loss: 0 (verified daily)
- **Clock drift: < 10ms average** (NEW)

### Governance Metrics (NEW)

**Compliance:**
- Metadata compliance rate: > 98%
- Violation resolution time: < 24h (auto-fixable)
- False positive rate: < 2%

**AI Cortex:**
- Tool call reduction rate: > 90%
- Context window efficiency: > 85%
- Token budget adherence: > 95%

---

## Success Definition (Updated)

**MVP Success (End of Month 3):**
- [ ] 100 beta users signed up
- [ ] 50+ daily active users
- [ ] 500+ projects created
- [ ] System uptime > 99%
- [ ] NPS score > 50
- [ ] **Metadata compliance rate > 98%** (NEW)
- [ ] **Clock drift < 10ms** (NEW)
- [ ] **AI Cortex operational** (NEW)
- [ ] **Extension Orchestration MVP live** (NEW)

**Product-Market Fit (Month 6):**
- [ ] 1,000+ active users
- [ ] 10,000+ projects created
- [ ] Revenue > $10K/month (post-beta pricing)
- [ ] Community template marketplace active
- [ ] **100+ community extension dashboards** (NEW)

---

## Appendix: Artifact Cross-Reference (Updated)

This implementation plan synthesizes all 20 artifacts:

| Artifact | How Used in Implementation Plan |
|----------|----------------------------------|
| **01: Vision** | Defines overall mission and success criteria |
| **02: Terminology** | Standardizes vocabulary across all tasks |
| **03: The Bin** | Week 1-3: Core implementation + Layer 0 |
| **04: Unified AI Layer** | Week 4: AI provider integration + timestamp normalization |
| **05: Data Model** | Week 1: TiDB schema setup (11 new tables) |
| **06: Master Control** | Week 4: Dashboard orchestration + governance/AI panels |
| **07: Propagation Engine** | Week 6: Propagation rules implementation |
| **08: Knowledge Sync** | Week 1: Validation layer architecture |
| **09: Lock Mechanism** | Week 1: Project initialization |
| **10: Template System** | Week 5: Template format and loader |
| **11: Dashboard Catalog** | Week 6-7: 3 template implementations |
| **12: Community Governance** | Week 8: Contribution process + metadata requirements |
| **13: Security Model** | Week 1-4: Temporal security capabilities |
| **14: Customization Pathways** | Week 5: AI-assisted wizard |
| **15: ADRs** | All weeks: Architectural decisions enforced |
| **16: Tech Stack Update** | Week 1: TiDB migration + HTAP justification |
| **17: Offline Architecture** | Week 9: PWA and service worker |
| **18: Migration System** | Week 10: Version upgrade wizard |
| **19: This Document** | Master implementation timeline |
| **20: Metadata Governance** | Week 1-3: Temporal standards, nomenclature, validation |

### New System Cross-References

| System | Primary Artifacts | Weeks |
|--------|------------------|-------|
| **Metadata Governance Dashboard** | ARTIFACT_20, ARTIFACT_03, ARTIFACT_05 | 1-3, 10-11 |
| **AI Cortex Management** | AI_CORTEX_MANAGEMENT_DASHBOARD, ARTIFACT_04 | 4, 6-7 |
| **Extension Orchestration** | VSCODE_EXTENSION_ORCHESTRATION_SYSTEM | 5, 8 |

---

## Final Notes

This plan is **ambitious but achievable** with:
- Dedicated 7-person team (up from 6)
- Clear scope boundaries (MVP + 3 new systems)
- Weekly risk reviews including governance metrics
- Monthly milestone reviews

**Key to success:**
1. **Stick to MVP scope** - resist feature creep
2. **Test continuously** - CI/CD prevents regressions
3. **Communicate daily** - stand-ups + Slack
4. **Celebrate wins** - weekly demos boost morale
5. **Monitor compliance** - governance is foundational (NEW)
6. **Track AI efficiency** - optimize context usage (NEW)

Good luck! 🚀

---

**End of ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md**

*Version: 2.1.1*  
*Last Updated: 2026-01-02T01:00:00Z*  
*Metadata Mini-Protocol Compliant: ✅*
