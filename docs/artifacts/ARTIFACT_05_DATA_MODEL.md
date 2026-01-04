# Data Model Specification (TiDB Schema)

**Document Type:** Core Architecture  
**Version:** 1.1.0  
**Date:** 2026-01-01T14:30:45Z  
**Database:** TiDB (MySQL-compatible, HTAP)  
**Status:** Updated with Metadata Governance, AI Cortex, and Extension Orchestration schemas

---

## Schema Overview

**Hierarchy:**
```
Project → Profile → Dashboard → Module → The Bin (SyncBuffer)
```

**Key Tables (41+):**
1. Core entities (projects, profiles, dashboards, modules)
2. Configuration (templates, guardrails, capabilities)
3. Synchronization (sync_buffer, propagation_log)
4. Audit & History (audit_logs, version_history)
5. AI Layer (context_windows, cached_responses)
6. **Metadata Governance** (metadata_standards, metadata_violations, filesystem_timestamps, clock_sync_status) ← NEW
7. **AI Cortex** (token_usage, api_configurations, tool_registry, external_tool_access) ← NEW
8. **Extension Orchestration** (extension_dashboards, project_extension_mappings, dashboard_versions) ← NEW

---

## Core Entity Tables

### projects

```sql
CREATE TABLE projects (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_user_id VARCHAR(255) NOT NULL,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    workflow_template VARCHAR(100),  -- 'vibe_coding', 'data_science', 'note_taking'
    git_repository VARCHAR(500),
    INDEX idx_owner (owner_user_id),
    INDEX idx_template (workflow_template)
) ENGINE=InnoDB;
```

### profiles

```sql
CREATE TABLE profiles (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    active_dashboard_id VARCHAR(36),  -- Single Active Screen enforcement
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project (project_id)
) ENGINE=InnoDB;
```

### dashboards

```sql
CREATE TABLE dashboards (
    id VARCHAR(36) PRIMARY KEY,
    profile_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    template_id VARCHAR(36),
    layout JSON NOT NULL,  -- Elm dashboard configuration
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    position INT NOT NULL,  -- Display order in Master Control
    FOREIGN KEY (profile_id) REFERENCES profiles(id) ON DELETE CASCADE,
    INDEX idx_profile (profile_id),
    INDEX idx_template (template_id)
) ENGINE=InnoDB;
```

### modules

```sql
CREATE TABLE modules (
    id VARCHAR(36) PRIMARY KEY,
    dashboard_id VARCHAR(36) NOT NULL,
    module_type VARCHAR(100) NOT NULL,  -- 'checklist', 'decision_log', 'file_manager'
    name VARCHAR(255) NOT NULL,
    configuration JSON NOT NULL,
    position INT NOT NULL,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    FOREIGN KEY (dashboard_id) REFERENCES dashboards(id) ON DELETE CASCADE,
    INDEX idx_dashboard (dashboard_id),
    INDEX idx_type (module_type)
) ENGINE=InnoDB;
```

---

## Configuration Tables

### templates

```sql
CREATE TABLE templates (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    category VARCHAR(100) NOT NULL,  -- 'workflow', 'dashboard', 'module'
    description TEXT,
    specification JSON NOT NULL,
    created_by VARCHAR(255),
    community_approved BOOLEAN DEFAULT FALSE,
    download_count INT DEFAULT 0,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    INDEX idx_category (category),
    INDEX idx_approved (community_approved)
) ENGINE=InnoDB;
```

### guardrails

```sql
CREATE TABLE guardrails (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36),  -- NULL for system guardrails
    rule_id VARCHAR(100) NOT NULL,
    description TEXT NOT NULL,
    severity ENUM('CRITICAL', 'HIGH', 'MEDIUM', 'LOW') NOT NULL,
    auto_reject BOOLEAN DEFAULT FALSE,
    configuration JSON,
    enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project (project_id),
    INDEX idx_severity (severity)
) ENGINE=InnoDB;
```

### capabilities

```sql
CREATE TABLE capabilities (
    id VARCHAR(36) PRIMARY KEY,
    module_id VARCHAR(36) NOT NULL,
    capability_type VARCHAR(100) NOT NULL,  -- 'file_write', 'network_http', 'git_commit'
    granted BOOLEAN DEFAULT FALSE,
    granted_at TIMESTAMP(6),
    revoked_at TIMESTAMP(6),
    justification TEXT,
    FOREIGN KEY (module_id) REFERENCES modules(id) ON DELETE CASCADE,
    INDEX idx_module (module_id),
    INDEX idx_type (capability_type)
) ENGINE=InnoDB;
```

---

## Synchronization Tables

### sync_buffer

```sql
CREATE TABLE sync_buffer (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    change_type VARCHAR(100) NOT NULL,
    source_module_id VARCHAR(36),
    payload JSON NOT NULL,
    status ENUM('PENDING', 'VALIDATED', 'REJECTED', 'PROPAGATED') NOT NULL DEFAULT 'PENDING',
    validation_results JSON,
    impact_analysis JSON,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    validated_at TIMESTAMP(6),
    propagated_at TIMESTAMP(6),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project_status (project_id, status),
    INDEX idx_created (created_at)
) ENGINE=InnoDB;
```

### propagation_log

```sql
CREATE TABLE propagation_log (
    id VARCHAR(36) PRIMARY KEY,
    sync_buffer_id VARCHAR(36) NOT NULL,
    affected_dashboard_id VARCHAR(36) NOT NULL,
    propagation_type VARCHAR(100) NOT NULL,
    status ENUM('SUCCESS', 'PARTIAL', 'FAILED') NOT NULL,
    error_message TEXT,
    token_cost INT,
    duration_ms INT,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    FOREIGN KEY (sync_buffer_id) REFERENCES sync_buffer(id) ON DELETE CASCADE,
    INDEX idx_sync_buffer (sync_buffer_id),
    INDEX idx_dashboard (affected_dashboard_id),
    INDEX idx_created (created_at)
) ENGINE=InnoDB;
```

---

## Audit & History Tables

### audit_logs

```sql
CREATE TABLE audit_logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    user_id VARCHAR(255) NOT NULL,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(255),
    status ENUM('success', 'failure', 'partial') NOT NULL,
    metadata JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    INDEX idx_user_timestamp (user_id, timestamp),
    INDEX idx_action (action),
    INDEX idx_resource (resource_type, resource_id),
    INDEX idx_timestamp (timestamp)
) ENGINE=InnoDB;
```

### version_history

```sql
CREATE TABLE version_history (
    id VARCHAR(36) PRIMARY KEY,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(255) NOT NULL,
    version_number INT NOT NULL,
    snapshot JSON NOT NULL,
    change_description TEXT,
    created_by VARCHAR(255) NOT NULL,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    INDEX idx_resource (resource_type, resource_id),
    INDEX idx_created (created_at)
) ENGINE=InnoDB;
```

---

## AI Layer Tables

### context_windows

```sql
CREATE TABLE context_windows (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    conversation_history JSON NOT NULL,
    current_tokens INT NOT NULL,
    max_tokens INT NOT NULL DEFAULT 200000,
    last_message_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project (project_id)
) ENGINE=InnoDB;
```

### cached_responses

```sql
CREATE TABLE cached_responses (
    id VARCHAR(36) PRIMARY KEY,
    prompt_hash VARCHAR(64) NOT NULL UNIQUE,
    prompt TEXT NOT NULL,
    response JSON NOT NULL,
    provider VARCHAR(100) NOT NULL,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    hit_count INT DEFAULT 0,
    INDEX idx_hash (prompt_hash),
    INDEX idx_provider (provider),
    INDEX idx_created (created_at)
) ENGINE=InnoDB;
```

---

## Metadata Governance Schema

The Metadata Governance subsystem enforces temporal standardization, nomenclature protocols, and versioning consistency across all DevGuide Cockpit dashboards. These tables support the 5-layer versioning stack:

**5-Layer Versioning Stack:**
1. **Layer 1 - Filesystem Timestamps:** Linux inode attributes (atime, mtime, ctime)
2. **Layer 2 - Git Versioning:** Commit history with Author/Committer dates, SHA-256
3. **Layer 3 - Database MVCC:** TiDB timestamp-based Multi-Version Concurrency Control
4. **Layer 4 - Application Versioning:** SemVer for artifacts, dashboards, API contracts
5. **Layer 5 - Project Snapshots:** Full project state checkpoints with rollback

**Temporal Precision Rationale:**
All timestamps use `TIMESTAMP(6)` for microsecond precision (1,000,000 divisions per second). This enables:
- Clock drift detection at <10ms target (10,000μs resolution is ample)
- Precise ordering of distributed events across TiDB nodes
- MAC time (atime, mtime, ctime) correlation with database timestamps

### metadata_standards

Central registry of all metadata validation standards. The Bin queries this table to validate submissions.

```sql
CREATE TABLE metadata_standards (
    id VARCHAR(36) PRIMARY KEY,
    standard_name VARCHAR(100) NOT NULL UNIQUE,
    category ENUM('temporal', 'nomenclature', 'versioning') NOT NULL,
    pattern_regex TEXT NOT NULL,
    validation_message TEXT,
    enforcement_level ENUM('error', 'warning', 'info') NOT NULL DEFAULT 'error',
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    INDEX idx_category (category),
    INDEX idx_active (is_active)
) ENGINE=InnoDB;
```

**Example Standards:**
```sql
-- Seed data: ISO 8601 Zulu timestamp requirement
INSERT INTO metadata_standards (id, standard_name, category, pattern_regex, validation_message, enforcement_level)
VALUES (
    'ts-001-zulu',
    'Timestamp Zulu Format',
    'temporal',
    '^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?Z$',
    'Timestamp must use ISO 8601 format with Z suffix (e.g., 2026-01-01T14:30:45.123456Z)',
    'error'
);
```

### metadata_violations

Tracks all detected metadata standard violations for compliance monitoring and resolution.

```sql
CREATE TABLE metadata_violations (
    id VARCHAR(36) PRIMARY KEY,
    standard_id VARCHAR(36) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(255) NOT NULL,
    violation_value TEXT,
    suggested_fix TEXT,
    status ENUM('open', 'resolved', 'suppressed') NOT NULL DEFAULT 'open',
    detected_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    resolved_at TIMESTAMP(6),
    resolved_by VARCHAR(36),
    FOREIGN KEY (standard_id) REFERENCES metadata_standards(id) ON DELETE RESTRICT,
    INDEX idx_resource (resource_type, resource_id),
    INDEX idx_status (status),
    INDEX idx_standard (standard_id),
    INDEX idx_detected (detected_at DESC)
) ENGINE=InnoDB;
```

### filesystem_timestamps

Captures MAC time attributes from filesystem for correlation with git and database timestamps.

```sql
CREATE TABLE filesystem_timestamps (
    id VARCHAR(36) PRIMARY KEY,
    file_path TEXT NOT NULL,
    atime TIMESTAMP(6) NOT NULL,  -- Access time (last read)
    mtime TIMESTAMP(6) NOT NULL,  -- Modification time (content changed)
    ctime TIMESTAMP(6) NOT NULL,  -- Change time (metadata changed)
    captured_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    git_commit_sha VARCHAR(40),   -- Associated git commit (if applicable)
    project_id VARCHAR(36),
    INDEX idx_file_path (file_path(255)),
    INDEX idx_mtime (mtime),
    INDEX idx_project (project_id),
    INDEX idx_captured (captured_at DESC)
) ENGINE=InnoDB;
```

### clock_sync_status

Monitors clock synchronization across all system components to ensure <10ms drift target.

```sql
CREATE TABLE clock_sync_status (
    id VARCHAR(36) PRIMARY KEY,
    component_name VARCHAR(100) NOT NULL,
    protocol ENUM('NTP', 'PTP', 'Browser', 'Server') NOT NULL,
    stratum INT,  -- NTP stratum level (0=reference, 1=primary, etc.)
    offset_microseconds BIGINT NOT NULL,  -- Current offset from reference
    last_sync_at TIMESTAMP(6) NOT NULL,
    status ENUM('synchronized', 'drifting', 'error') NOT NULL,
    error_message TEXT,
    INDEX idx_component (component_name),
    INDEX idx_status (status),
    INDEX idx_last_sync (last_sync_at DESC)
) ENGINE=InnoDB;
```

---

## AI Cortex Schema

The AI Cortex subsystem provides centralized observability and control for all AI operations. These tables support token tracking, API configuration management, tool registry, and external integrations.

**Design Principles:**
- **Encrypted Storage:** API keys use VARBINARY for encrypted storage (application-layer encryption)
- **Time-Series Analytics:** token_usage is optimized for HTAP workloads with TiFlash columnar storage
- **Soft References:** High-throughput tables use soft FK references for performance
- **Provider Agnostic:** Supports multiple AI providers (Claude, OpenAI, local models)

### token_usage

Time-series table tracking token consumption across all AI operations. Designed for HTAP analytics.

```sql
CREATE TABLE token_usage (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    provider VARCHAR(50) NOT NULL,           -- 'claude', 'openai', 'phi3', 'tinyllama'
    model VARCHAR(100) NOT NULL,             -- 'claude-sonnet-4-5-20250929', 'gpt-4-turbo'
    input_tokens INT NOT NULL DEFAULT 0,
    output_tokens INT NOT NULL DEFAULT 0,
    total_cost DECIMAL(10,4) NOT NULL DEFAULT 0.0000,
    operation_type VARCHAR(50),              -- 'chat', 'estimation', 'tool_call'
    dashboard_context VARCHAR(100),          -- Which dashboard triggered this
    timestamp TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    INDEX idx_project_timestamp (project_id, timestamp DESC),
    INDEX idx_provider_timestamp (provider, timestamp DESC),
    INDEX idx_model (model),
    INDEX idx_timestamp (timestamp DESC)
) ENGINE=InnoDB;

-- TiFlash replica for analytics queries
-- ALTER TABLE token_usage SET TIFLASH REPLICA 1;
```

**Partitioning Recommendation (Future):**
```sql
-- Partition by month for efficient retention and analytics
ALTER TABLE token_usage
PARTITION BY RANGE (TO_DAYS(timestamp)) (
    PARTITION p202601 VALUES LESS THAN (TO_DAYS('2026-02-01')),
    PARTITION p202602 VALUES LESS THAN (TO_DAYS('2026-03-01')),
    PARTITION p202603 VALUES LESS THAN (TO_DAYS('2026-04-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### api_configurations

Stores AI provider configurations with encrypted API keys. Supports per-project customization.

```sql
CREATE TABLE api_configurations (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    provider VARCHAR(50) NOT NULL,           -- 'anthropic', 'openai', 'azure', 'local'
    api_key_encrypted VARBINARY(512),        -- AES-256-GCM encrypted
    model VARCHAR(100) NOT NULL,
    temperature DECIMAL(3,2) DEFAULT 0.70,
    max_tokens INT DEFAULT 4096,
    enabled BOOLEAN NOT NULL DEFAULT TRUE,
    priority INT DEFAULT 0,                  -- For provider fallback ordering
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    UNIQUE KEY uk_project_provider (project_id, provider),
    INDEX idx_enabled (enabled)
) ENGINE=InnoDB;
```

### tool_registry

Catalog of all available tools that can be invoked by AI agents.

```sql
CREATE TABLE tool_registry (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    category VARCHAR(50) NOT NULL,           -- 'file_ops', 'database', 'git', 'ai', 'external'
    description TEXT NOT NULL,
    implementation_type ENUM('python', 'api', 'database', 'wasm') NOT NULL,
    config JSON NOT NULL,                    -- Tool-specific configuration schema
    is_system BOOLEAN NOT NULL DEFAULT FALSE, -- System tools vs user-defined
    requires_capability VARCHAR(100),        -- Security capability required
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    INDEX idx_category (category),
    INDEX idx_system (is_system)
) ENGINE=InnoDB;
```

### external_tool_access

Manages OAuth tokens and access credentials for external service integrations.

```sql
CREATE TABLE external_tool_access (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    tool_id VARCHAR(36) NOT NULL,
    access_type ENUM('read', 'write', 'admin') NOT NULL DEFAULT 'read',
    oauth_config JSON,                       -- OAuth tokens, refresh tokens, scopes
    enabled BOOLEAN NOT NULL DEFAULT TRUE,
    last_accessed_at TIMESTAMP(6),
    expires_at TIMESTAMP(6),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (tool_id) REFERENCES tool_registry(id) ON DELETE CASCADE,
    UNIQUE KEY uk_project_tool (project_id, tool_id),
    INDEX idx_enabled (enabled),
    INDEX idx_expires (expires_at)
) ENGINE=InnoDB;
```

---

## Extension Orchestration Schema

The Extension Orchestration subsystem elevates VS Code extensions to first-class customizable components. These tables support dashboard templates, per-project mappings, and version history for extension configurations.

**Design Principles:**
- **Community Repository:** extension_dashboards stores shareable templates (like Claude MCP servers)
- **Per-Project Customization:** project_extension_mappings allows different dashboards per project
- **Version Tracking:** dashboard_versions enables rollback and change history
- **Template Inheritance:** Custom configs can extend base templates

### extension_dashboards

Community repository of dashboard templates for VS Code extensions.

```sql
CREATE TABLE extension_dashboards (
    id VARCHAR(36) PRIMARY KEY,
    extension_id VARCHAR(255) NOT NULL,      -- VS Code extension identifier
    dashboard_id VARCHAR(255) NOT NULL,      -- Internal dashboard identifier
    name VARCHAR(255) NOT NULL,
    description TEXT,
    config JSON NOT NULL,                    -- Dashboard configuration schema
    author VARCHAR(255),
    community_approved BOOLEAN NOT NULL DEFAULT FALSE,
    download_count INT DEFAULT 0,
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    INDEX idx_extension_id (extension_id),
    INDEX idx_approved (community_approved),
    INDEX idx_downloads (download_count DESC)
) ENGINE=InnoDB;
```

### project_extension_mappings

Per-project customization of which dashboard template is used for each extension.

```sql
CREATE TABLE project_extension_mappings (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    extension_id VARCHAR(255) NOT NULL,
    dashboard_id VARCHAR(36) NOT NULL,       -- References extension_dashboards.id
    enabled BOOLEAN NOT NULL DEFAULT TRUE,
    custom_config JSON,                      -- Project-specific overrides
    position INT DEFAULT 0,                  -- Display order
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    updated_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (dashboard_id) REFERENCES extension_dashboards(id) ON DELETE RESTRICT,
    UNIQUE KEY uk_project_extension (project_id, extension_id),
    INDEX idx_project (project_id),
    INDEX idx_enabled (enabled)
) ENGINE=InnoDB;
```

### dashboard_versions

Version history for extension dashboard configurations, enabling rollback.

```sql
CREATE TABLE dashboard_versions (
    id VARCHAR(36) PRIMARY KEY,
    dashboard_id VARCHAR(36) NOT NULL,       -- References extension_dashboards.id
    version VARCHAR(50) NOT NULL,            -- SemVer: '1.0.0', '1.1.0'
    config JSON NOT NULL,                    -- Complete config snapshot
    changelog TEXT,                          -- What changed in this version
    created_by VARCHAR(255),
    created_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    FOREIGN KEY (dashboard_id) REFERENCES extension_dashboards(id) ON DELETE CASCADE,
    INDEX idx_dashboard_created (dashboard_id, created_at DESC),
    INDEX idx_version (version)
) ENGINE=InnoDB;
```

---

## Performance Indexes (NEW)

### Metadata Governance Indexes

```sql
-- Fast lookup of open violations for compliance dashboard
CREATE INDEX idx_violations_open_detected ON metadata_violations(status, detected_at DESC)
    WHERE status = 'open';

-- Standard-based violation aggregation
CREATE INDEX idx_violations_standard_status ON metadata_violations(standard_id, status);
```

### AI Cortex Indexes

```sql
-- Composite index for project token analytics (most common query pattern)
CREATE INDEX idx_token_usage_project_provider_timestamp 
    ON token_usage(project_id, provider, timestamp DESC);

-- Cost analysis by model
CREATE INDEX idx_token_usage_model_cost ON token_usage(model, total_cost DESC);
```

### Extension Orchestration Indexes

```sql
-- Fast lookup of project's enabled extensions
CREATE INDEX idx_project_extensions_enabled 
    ON project_extension_mappings(project_id, enabled)
    WHERE enabled = TRUE;

-- Version history navigation
CREATE INDEX idx_dashboard_versions_dashboard_created 
    ON dashboard_versions(dashboard_id, created_at DESC);
```

---

## Performance Optimizations

**Indexes:**
- Composite indexes on frequent query patterns
- Covering indexes for read-heavy queries
- Partial indexes for status-filtered queries

**Partitioning Strategy:**

```sql
-- Partition audit_logs by month
ALTER TABLE audit_logs
PARTITION BY RANGE (TO_DAYS(timestamp)) (
    PARTITION p202601 VALUES LESS THAN (TO_DAYS('2026-02-01')),
    PARTITION p202602 VALUES LESS THAN (TO_DAYS('2026-03-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Partition token_usage by month (high-volume time-series)
ALTER TABLE token_usage
PARTITION BY RANGE (TO_DAYS(timestamp)) (
    PARTITION p202601 VALUES LESS THAN (TO_DAYS('2026-02-01')),
    PARTITION p202602 VALUES LESS THAN (TO_DAYS('2026-03-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

**Query Caching:**
- TiDB query cache for static data (templates)
- 5-minute TTL for profile configs
- 1-hour TTL for template catalog

**TiFlash Columnar Storage:**
```sql
-- Enable TiFlash replicas for analytics tables
ALTER TABLE token_usage SET TIFLASH REPLICA 1;
ALTER TABLE audit_logs SET TIFLASH REPLICA 1;
ALTER TABLE metadata_violations SET TIFLASH REPLICA 1;
```

---

## HTAP Queries (Analytics)

**Example: High-Impact Changes Report**

```sql
SELECT 
    sb.id AS change_id,
    sb.change_type,
    COUNT(DISTINCT pl.affected_dashboard_id) AS affected_dashboards,
    SUM(pl.token_cost) AS total_token_cost,
    AVG(pl.duration_ms) AS avg_duration_ms
FROM sync_buffer sb
JOIN propagation_log pl ON sb.id = pl.sync_buffer_id
WHERE sb.created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY)
GROUP BY sb.id, sb.change_type
HAVING affected_dashboards > 3
ORDER BY total_token_cost DESC
LIMIT 10;
```

**Example: Token Usage Analytics (30-Day Trend)**

```sql
-- Runs on TiFlash for sub-second response
SELECT 
    DATE(timestamp) AS usage_date,
    provider,
    SUM(input_tokens) AS total_input,
    SUM(output_tokens) AS total_output,
    SUM(total_cost) AS daily_cost
FROM token_usage
WHERE project_id = ? 
    AND timestamp >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(timestamp), provider
ORDER BY usage_date DESC;
```

**Example: Metadata Compliance Dashboard**

```sql
-- Real-time compliance score
SELECT 
    ms.category,
    ms.standard_name,
    COUNT(mv.id) AS violation_count,
    COUNT(CASE WHEN mv.status = 'open' THEN 1 END) AS open_violations,
    ROUND(
        (1 - COUNT(CASE WHEN mv.status = 'open' THEN 1 END) * 1.0 / NULLIF(COUNT(mv.id), 0)) * 100, 
        2
    ) AS compliance_rate
FROM metadata_standards ms
LEFT JOIN metadata_violations mv ON ms.id = mv.standard_id
WHERE ms.is_active = TRUE
GROUP BY ms.id, ms.category, ms.standard_name
ORDER BY compliance_rate ASC;
```

**TiFlash Acceleration:**
- Analytics queries run on TiFlash (columnar storage)
- OLTP transactions on TiKV (row storage)
- Automatic query routing based on workload type

---

## Foreign Key Strategy

**Hard Constraints (CASCADE DELETE):**
- projects → profiles → dashboards → modules (core hierarchy)
- projects → api_configurations, project_extension_mappings (project-scoped)
- extension_dashboards → dashboard_versions (version history)

**Soft References (Performance):**
- token_usage.project_id (high-throughput, no FK)
- filesystem_timestamps.project_id (high-volume capture)
- clock_sync_status (monitoring data, no FK)

**RESTRICT on Delete:**
- metadata_violations → metadata_standards (preserve audit trail)
- project_extension_mappings → extension_dashboards (prevent template deletion)

---

## Migration Notes

**Version 1.0.0 → 1.1.0:**
- **Added:** 11 new tables (4 metadata + 4 AI cortex + 3 extension)
- **Added:** 6+ new performance indexes
- **Modified:** Core tables now use TIMESTAMP(6) for microsecond precision
- **Backward Compatible:** No breaking changes to existing tables
- **Migration Complexity:** Low (new tables only)

**Rollback Strategy:**
```sql
-- If rollback needed, drop new tables in reverse dependency order
DROP TABLE IF EXISTS dashboard_versions;
DROP TABLE IF EXISTS project_extension_mappings;
DROP TABLE IF EXISTS extension_dashboards;
DROP TABLE IF EXISTS external_tool_access;
DROP TABLE IF EXISTS tool_registry;
DROP TABLE IF EXISTS api_configurations;
DROP TABLE IF EXISTS token_usage;
DROP TABLE IF EXISTS clock_sync_status;
DROP TABLE IF EXISTS filesystem_timestamps;
DROP TABLE IF EXISTS metadata_violations;
DROP TABLE IF EXISTS metadata_standards;
```

---

## Schema Validation Queries

**Verify All Tables Exist:**
```sql
SELECT TABLE_NAME, TABLE_ROWS, CREATE_TIME
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'devguide_cockpit'
ORDER BY TABLE_NAME;
```

**Verify TIMESTAMP(6) Precision:**
```sql
SELECT TABLE_NAME, COLUMN_NAME, COLUMN_TYPE
FROM information_schema.COLUMNS
WHERE TABLE_SCHEMA = 'devguide_cockpit'
    AND DATA_TYPE = 'timestamp'
ORDER BY TABLE_NAME, COLUMN_NAME;
```

**Verify Index Coverage:**
```sql
SELECT TABLE_NAME, INDEX_NAME, COLUMN_NAME, SEQ_IN_INDEX
FROM information_schema.STATISTICS
WHERE TABLE_SCHEMA = 'devguide_cockpit'
ORDER BY TABLE_NAME, INDEX_NAME, SEQ_IN_INDEX;
```

---

**End of DATA_MODEL_SPECIFICATION.md**

*Version: 1.1.0*  
*Last Updated: 2026-01-01T14:30:45Z*  
*Status: Ready for TIER 2 dependencies*
