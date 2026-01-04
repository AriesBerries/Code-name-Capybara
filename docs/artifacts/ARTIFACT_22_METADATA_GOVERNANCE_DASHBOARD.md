# ARTIFACT_20: METADATA GOVERNANCE DASHBOARD

**Date:** January 1, 2026  
**Artifact:** METADATA_GOVERNANCE_DASHBOARD.md  
**Priority:** Tier 1 - Foundation (Cross-Cutting)  
**Dependencies:** All other artifacts (cross-cutting governance)  
**Status:** Ready for Implementation

---

## 1. Executive Summary

This specification defines the **Metadata Governance Dashboard**, a foundational cross-cutting system that standardizes temporal references, nomenclature protocols, and versioning metadata across all DevGuide Cockpit dashboards.

### Core Value Proposition

| Challenge | Solution |
|-----------|----------|
| Timezone ambiguity in global teams | Zulu-First paradigm (UTC everywhere) |
| AI retrieval failures from poor naming | LPDS entity-based nomenclature |
| Version traceability gaps | 5-layer versioning stack |
| Inconsistent metadata across dashboards | Unified enforcement via The Bin |

---

## 2. Temporal Standardization Architecture

### 2.1 Canonical Timestamp Formats

| Context | Format | Example |
|---------|--------|---------|
| Database Storage | TIMESTAMP(6) WITH TIME ZONE | `2026-01-01 14:30:45.123456Z` |
| API Responses | ISO 8601 Extended (RFC 3339) | `2026-01-01T14:30:45.123Z` |
| File Naming | ISO 8601 Basic (no punctuation) | `20260101T143045Z` |
| Git Commits | Unix Epoch + TZ Offset | `1735742445 +0000` |
| Artifact Versioning | SemVer + Zulu Build Metadata | `1.2.3+20260101T143045Z` |

### 2.2 Clock Synchronization Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLOCK SYNCHRONIZATION HIERARCHY               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Stratum 0: GPS/Atomic Reference (AWS Time Sync)               │
│        │                                                         │
│        ▼                                                         │
│   Stratum 1: TiDB Database Cluster (PTP, < 1μs)                 │
│        │                                                         │
│        ▼                                                         │
│   Stratum 2: Go API Servers (NTP/chrony, < 10ms)                │
│        │                                                         │
│        ▼                                                         │
│   Client-Derived: Elm Frontend (Server-authoritative JWT)       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Temporal Validation Rules

```rust
// Rust WASM Kernel - Temporal Validator
pub struct TemporalValidator;

impl TemporalValidator {
    pub fn validate_iso8601(timestamp: &str) -> Result<(), ValidationError> {
        // Extended format: 2026-01-01T14:30:45.123Z
        let extended_regex = r"^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?Z$";
        
        // Basic format: 20260101T143045Z
        let basic_regex = r"^\d{8}T\d{6}Z$";
        
        if Regex::new(extended_regex).unwrap().is_match(timestamp) 
            || Regex::new(basic_regex).unwrap().is_match(timestamp) {
            Ok(())
        } else {
            Err(ValidationError::InvalidTimestampFormat {
                value: timestamp.to_string(),
                expected: "ISO 8601 with Z suffix".to_string(),
            })
        }
    }
    
    pub fn ensure_utc(timestamp: &str) -> Result<(), ValidationError> {
        if !timestamp.ends_with('Z') && !timestamp.contains("+00:00") {
            Err(ValidationError::NonUtcTimestamp {
                suggestion: "Convert to UTC and append 'Z' suffix".to_string(),
            })
        } else {
            Ok(())
        }
    }
}
```

---

## 3. AI-Optimized Nomenclature System

### 3.1 Design Philosophy

The nomenclature system optimizes for **AI context retrieval** as the primary use case, with human readability as a secondary concern. In vibe coding environments, AI agents construct prompts from:

1. File paths and names
2. Symbol graphs (imports, exports)
3. Recent edit history
4. Semantic embeddings

Ambiguous names like `util.js` or `temp123.py` produce low relevance scores in vector embedding space, causing retrieval failures.

### 3.2 Repository Naming Convention

**Pattern:** `<domain>-<technology>-<maturity>-<topology>`

| Segment | Valid Values | Purpose |
|---------|--------------|---------|
| domain | `user`, `order`, `payment`, `auth` | Business domain (AI-searchable) |
| technology | `elm`, `go`, `rust`, `wasm`, `sql` | Primary language/stack |
| maturity | `dev`, `staging`, `release`, `archive` | Lifecycle stage |
| topology | `primary`, `replica`, `edge`, `local` | Deployment location |

**Examples:**
- `devguide-cockpit-release-primary`
- `user-service-go-staging-edge`
- `governance-scripts-python-dev-local`

### 3.3 Branch Naming Convention

**Pattern:** `<type>/<ticket>-<description>`

| Prefix | Use Case | Example |
|--------|----------|---------|
| `feat/` | New feature | `feat/DGC-42-metadata-dashboard` |
| `fix/` | Bug fix | `fix/DGC-99-timestamp-drift` |
| `ai/` | AI-generated code (vibe coding) | `ai/DGC-55-copilot-auth-flow` |
| `refactor/` | Code restructuring | `refactor/DGC-77-propagation-engine` |
| `docs/` | Documentation | `docs/DGC-12-api-reference` |
| `chore/` | Maintenance | `chore/DGC-101-dependency-update` |

### 3.4 File Naming Convention (LPDS Standard)

**Pattern:** `<domain>_<type>-<variant>_<proc>-<stage>_<ver>-<semver>.<ext>`

**Entity Keys:**

| Key | Description | Valid Values |
|-----|-------------|--------------|
| `domain` | Business domain | `bin`, `dashboard`, `propagation`, `ai`, `sync` |
| `type` | Functional type | `validator`, `handler`, `repository`, `model`, `view` |
| `proc` | Processing stage | `raw`, `validated`, `transformed`, `production` |
| `ver` | Semantic version | `1.0.0`, `2.3.1` |

**Examples:**
```
bin_validator-guardrail_proc-production_ver-1.2.3.rs
dashboard_model-config_proc-validated_ver-2.0.0.elm
propagation_handler-batch_proc-transformed_ver-1.1.0.go
```

### 3.5 AI Relevance Scoring

The nomenclature system includes a relevance scoring algorithm based on cosine similarity between query embeddings and file name embeddings:

```
Cosine Similarity = (A · B) / (||A|| × ||B||)

Where:
- A = embedding vector of user query
- B = embedding vector of file name + metadata
```

**Scoring Thresholds:**
- **> 0.85:** High relevance (include in top-K results)
- **0.70 - 0.85:** Medium relevance (include if context budget allows)
- **< 0.70:** Low relevance (exclude from context)

---

## 4. Multi-Layer Versioning Architecture

### 4.1 Five-Layer Versioning Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                    VERSIONING STACK                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Layer 5: PROJECT SNAPSHOTS                                     │
│   └── Full project state checkpoints with rollback               │
│                                                                  │
│   Layer 4: APPLICATION VERSIONING                                │
│   └── SemVer for artifacts, dashboards, API contracts            │
│                                                                  │
│   Layer 3: DATABASE MVCC                                         │
│   └── TiDB timestamp-based Multi-Version Concurrency Control     │
│                                                                  │
│   Layer 2: GIT VERSIONING                                        │
│   └── Commit history with Author/Committer dates, SHA-256        │
│                                                                  │
│   Layer 1: FILESYSTEM TIMESTAMPS                                 │
│   └── Linux inode attributes: atime, mtime, ctime                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Linux Filesystem Timestamp Integration

| Attribute | Trigger | Governance Application |
|-----------|---------|------------------------|
| `atime` | File read/execution | AI model inference tracking, dependency audits |
| `mtime` | Data content change | Version trigger, build systems, dataset lineage |
| `ctime` | Metadata modification | Permission audits, ownership tracking, security |

### 4.3 Forensic Timestamp Rules

1. **Rule of Copying:** `mtime < ctime` → File likely copied from another volume
2. **Rule of Batch Processing:** Clustered `atime` values → Automated (non-human) activity
3. **Rule of Integrity:** `mtime = ctime` → File unmodified since last move/permission change
4. **Timestomping Detection:** Monitor `utimensat` syscalls via `auditd`

---

## 5. Data Model Specification

### 5.1 Core Tables

```sql
-- Central registry of metadata standards
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
    INDEX idx_category (category)
) ENGINE=InnoDB;

-- Tracks compliance violations
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
    INDEX idx_status (status)
) ENGINE=InnoDB;

-- Filesystem timestamp cache
CREATE TABLE filesystem_timestamps (
    id VARCHAR(36) PRIMARY KEY,
    file_path TEXT NOT NULL,
    atime TIMESTAMP(6) NOT NULL,
    mtime TIMESTAMP(6) NOT NULL,
    ctime TIMESTAMP(6) NOT NULL,
    captured_at TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
    git_commit_sha VARCHAR(40),
    INDEX idx_file_path (file_path(255)),
    INDEX idx_mtime (mtime)
) ENGINE=InnoDB;

-- Clock synchronization status
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
) ENGINE=InnoDB;
```

### 5.2 Default Standards Seeding

```sql
-- Temporal Standards
INSERT INTO metadata_standards (id, standard_name, category, pattern_regex, validation_message, enforcement_level) VALUES
('ts-001', 'api_timestamp_format', 'temporal', '^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?Z$', 'API timestamps must use ISO 8601 extended format with Z suffix', 'error'),
('ts-002', 'file_timestamp_format', 'temporal', '^\d{8}T\d{6}Z$', 'File name timestamps must use ISO 8601 basic format (no punctuation)', 'error'),
('ts-003', 'db_timestamp_format', 'temporal', '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}(\.\d{1,6})?(\+00:00|Z)$', 'Database timestamps must be UTC with timezone indicator', 'error');

-- Nomenclature Standards
INSERT INTO metadata_standards (id, standard_name, category, pattern_regex, validation_message, enforcement_level) VALUES
('nm-001', 'repository_naming', 'nomenclature', '^[a-z][a-z0-9]*(-[a-z][a-z0-9]*){2,3}$', 'Repository names must follow <domain>-<tech>-<maturity>-<topology> pattern', 'error'),
('nm-002', 'branch_naming', 'nomenclature', '^(feat|fix|ai|refactor|docs|chore)/[A-Z]+-\d+-[a-z0-9-]+$', 'Branch names must follow <type>/<TICKET>-<description> pattern', 'warning'),
('nm-003', 'file_naming_lpds', 'nomenclature', '^[a-z]+_[a-z]+-[a-z]+(_[a-z]+-[a-z0-9.]+)*\.[a-z]+$', 'File names should follow LPDS entity-based pattern', 'warning');

-- Versioning Standards
INSERT INTO metadata_standards (id, standard_name, category, pattern_regex, validation_message, enforcement_level) VALUES
('vs-001', 'semver_format', 'versioning', '^\d+\.\d+\.\d+(-[a-zA-Z0-9]+(\.[a-zA-Z0-9]+)*)?(\+[a-zA-Z0-9]+(\.[a-zA-Z0-9]+)*)?$', 'Version strings must follow Semantic Versioning 2.0.0', 'error'),
('vs-002', 'semver_build_zulu', 'versioning', '^\d+\.\d+\.\d+\+\d{8}T\d{6}Z$', 'Build metadata should include Zulu timestamp', 'info');
```

---

## 6. Dashboard User Interface

### 6.1 Workflow Modules

| Module | Capabilities |
|--------|--------------|
| **Standards Registry** | CRUD for standards; regex testing; bulk import/export |
| **Compliance Monitor** | Real-time violations; severity heatmaps; trend analysis |
| **Nomenclature Validator** | Interactive validation; AI relevance scoring; batch rename |
| **Timestamp Analyzer** | Clock drift monitoring; MAC forensics; timezone utilities |

### 6.2 Elm Module Structure

```elm
module Dashboard.MetadataGovernance exposing 
    ( Model
    , Msg
    , init
    , update
    , view
    , subscriptions
    )

-- MODEL

type alias Model =
    { activeTab : Tab
    , standards : List MetadataStandard
    , violations : List Violation
    , clockStatus : List ClockSyncStatus
    , validationInput : String
    , validationResult : Maybe ValidationResult
    }

type Tab
    = StandardsRegistry
    | ComplianceMonitor
    | NomenclatureValidator
    | TimestampAnalyzer

type alias MetadataStandard =
    { id : String
    , name : String
    , category : Category
    , patternRegex : String
    , enforcementLevel : EnforcementLevel
    , isActive : Bool
    }

type Category
    = Temporal
    | Nomenclature
    | Versioning

type EnforcementLevel
    = Error
    | Warning
    | Info

-- VIEW

view : Model -> Html Msg
view model =
    div [ class "metadata-governance-dashboard" ]
        [ viewHeader model
        , viewTabNavigation model.activeTab
        , viewActiveContent model
        , viewFooterStats model
        ]

viewActiveContent : Model -> Html Msg
viewActiveContent model =
    case model.activeTab of
        StandardsRegistry ->
            viewStandardsRegistry model.standards
        
        ComplianceMonitor ->
            viewComplianceMonitor model.violations
        
        NomenclatureValidator ->
            viewNomenclatureValidator model.validationInput model.validationResult
        
        TimestampAnalyzer ->
            viewTimestampAnalyzer model.clockStatus
```

---

## 7. The Bin Integration

### 7.1 Validation Layer Position

The Metadata Governance Validator operates as **Layer 0** in The Bin validation pipeline—the first check before any other validation layers execute.

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE BIN VALIDATION PIPELINE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   → Layer 0: METADATA GOVERNANCE (This Dashboard)               │
│       └── Timestamp formats, file names, version strings         │
│                                                                  │
│   → Layer 1: RUNTIME (Code execution verification)               │
│   → Layer 2: SPECIFICATION (Config schema validation)            │
│   → Layer 3: DOCUMENTATION (Generated docs sync)                 │
│   → Layer 4: DECISIONS (Rationale captured)                      │
│   → Layer 5: PROPAGATION (Impact analysis)                       │
│   → Layer 6: COMMIT (Atomic database + Git commit)               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Rust WASM Validation Kernel

```rust
// governance_kernel/src/metadata_validator.rs

use wasm_bindgen::prelude::*;
use serde::{Deserialize, Serialize};
use regex::Regex;

#[derive(Serialize, Deserialize)]
pub struct MetadataValidationResult {
    pub is_valid: bool,
    pub violations: Vec<MetadataViolation>,
    pub warnings: Vec<MetadataViolation>,
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

#[wasm_bindgen]
pub fn validate_bin_item(item_json: &str) -> String {
    let item: BinItem = serde_json::from_str(item_json).unwrap();
    let mut result = MetadataValidationResult {
        is_valid: true,
        violations: vec![],
        warnings: vec![],
    };
    
    // Validate timestamps
    validate_timestamps(&item, &mut result);
    
    // Validate file names
    validate_nomenclature(&item, &mut result);
    
    // Validate version strings
    validate_versions(&item, &mut result);
    
    result.is_valid = result.violations.is_empty();
    serde_json::to_string(&result).unwrap()
}
```

---

## 8. Enforcement Mechanisms

### 8.1 Three-Layer Enforcement

| Layer | Mechanism | Standards Enforced |
|-------|-----------|-------------------|
| Pre-commit | Git hooks (Husky) | File naming, branch naming, commit message |
| CI/CD | GitHub Actions | All nomenclature, SemVer, PR title format |
| Runtime | The Bin Validator | Timestamp format, API contracts, config schemas |

### 8.2 GitHub Actions Workflows

**lint-filenames.yml:**
```yaml
name: Lint File Names

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate File Names
        uses: ls-lint/action@v2
        with:
          config: |
            ls:
              packages/**/*.{ts,tsx}:
                - regex:^[a-z]+_[a-z]+-[a-z]+.*$
              packages/**/*.{go,rs}:
                - regex:^[a-z]+_[a-z]+-[a-z]+.*$
```

**semantic-pr.yml:**
```yaml
name: Semantic PR

on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: amannn/action-semantic-pull-request@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          types: |
            feat
            fix
            docs
            refactor
            chore
          requireScope: false
          subjectPattern: ^[A-Z]+-\d+.*$
          subjectPatternError: |
            PR title must include ticket number (e.g., "DGC-42: Add feature")
```

---

## 9. Implementation Roadmap

### Phase 1: Foundation (Weeks 1-2)
- [ ] Database schema deployment
- [ ] Default standards seeding
- [ ] Basic Elm UI shell
- [ ] Go API endpoints for CRUD

### Phase 2: Validation Kernel (Weeks 3-4)
- [ ] Rust WASM validation logic
- [ ] The Bin integration
- [ ] Pre-commit hooks (Husky)
- [ ] SSE violation streaming

### Phase 3: Enforcement Automation (Weeks 5-6)
- [ ] GitHub Actions workflows
- [ ] AI relevance scoring integration
- [ ] Compliance heatmaps
- [ ] Batch rename tooling

### Phase 4: Advanced Features (Weeks 7-8)
- [ ] Timestamp forensics module
- [ ] `auditd` integration for timestomping detection
- [ ] Clock drift monitoring dashboard
- [ ] ADR documentation generation

---

## 10. Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Compliance Rate | > 98% | Artifacts passing validation within 30 days |
| Clock Drift | < 10ms | Average deviation across Go API servers |
| AI Retrieval Accuracy | > 85% | Context retriever relevance scores |
| Violation Resolution Time | < 24h | Average for auto-fixable issues |
| False Positive Rate | < 2% | Manual override frequency |

---

## 11. Cross-Dashboard Propagation

This dashboard's standards automatically apply to all other dashboards via The Bin. Any change to metadata standards triggers:

1. **Immediate Validation:** All pending Bin items re-validated
2. **Retroactive Audit:** Historical artifacts scanned for new violations
3. **Documentation Update:** API contracts and ADRs regenerated
4. **Notification:** SSE broadcast to all active dashboard sessions

---

## 12. Open Questions for Resolution

1. **Clock Drift Tolerance:** Should components with > 100ms drift be automatically quarantined?
2. **Legacy File Migration:** Auto-rename existing files or grandfather them?
3. **AI Branch Review:** Require different AI model to review `ai/` branch PRs?
4. **Timestamp Precision:** Is microsecond precision sufficient, or do we need nanoseconds for certain operations?

---

**End of ARTIFACT_20: METADATA_GOVERNANCE_DASHBOARD**
