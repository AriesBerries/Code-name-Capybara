# DevGuide Cockpit: Terminology Glossary

**Document Type:** Foundation  
**Version:** 1.1.0  
**Last Updated:** 2026-01-02T00:00:00Z

---

## Core Concepts

### The Bin
**Definition:** A three-fold construct serving as:
1. **Data Structure:** Rust `SyncBuffer<T>` holding pending changes
2. **Conceptual Checkpoint:** Validation gate before propagation
3. **UI Component:** Visual representation in every dashboard

**Purpose:** Enforce synchronization and prevent desynchronized state

**Example:** User edits API spec → change queued in The Bin → validation runs → user accepts/rejects → propagation to affected artifacts

---

### Capability
**Definition:** Permission-gated action that modules can perform.

**Categories:**
- File System (read, write, delete)
- Network (HTTP, WebSocket)
- Git Operations (commit, push, merge)
- AI Access (context, reasoning, code generation)
- External Tools (GitHub API, Jira API)

**Enforcement:** Three layers (compile-time, runtime, transaction)

---

### Dashboard
**Definition:** NOT a passive display of information, but an **active workflow interface** for a specific phase of development.

**Types:**
- **Single-Module Dashboard:** Monitors one workflow module
- **Pipeline Dashboard:** Orchestrates multiple modules in sequence
- **Master Control Dashboard:** Orchestration layer for all dashboards

**Example:** "Code Review Dashboard" (not just showing PR status, but providing AI-assisted review suggestions, approval workflows, and propagation to deployment)

---

### Guardrail
**Definition:** Validation rule enforced at The Bin checkpoint.

**Types:**
- **System Guardrails:** Non-negotiable (e.g., no plaintext secrets)
- **Project Guardrails:** User-configured (e.g., test coverage >80%)
- **AI Guardrails:** Reasoning-based (e.g., "Does this change violate single responsibility?")

**Example:** `no_deployment_without_tests` → blocks propagation to deployment dashboard if tests missing

---

### HTAP (Hybrid Transactional/Analytical Processing)
**Definition:** TiDB's dual capability for both OLTP and OLAP workloads.

**Benefit:** Real-time analytics without ETL pipelines

---

### Knowledge Synchronization Engine
**Definition:** Conceptual framework ensuring truth has single origin point.

**Six Layers:**
1. REALITY (what runs)
2. SPECIFICATION (what we promised)
3. DOCUMENTATION (what we told humans)
4. DECISIONS (why we built this)
5. VALIDATION (checks alignment)
6. MERGE GATE (The Bin)

---

### Master Control Dashboard
**Definition:** Orchestration layer providing:
- Drag-and-drop dashboard selection
- Version management
- Impact assessment (high-impact changes visualization)
- Accept/reject workflow for propagated updates

**Cross-Reference:** See *Metadata Governance Dashboard* for standardization enforcement

---

### Module
**Definition:** Reusable, pluggable component within a dashboard.

**Built-In Modules:**
- Checklist Module
- Decision Log Module
- File Manager Module
- Code Snippet Module
- Documentation Generator Module
- Git Integration Module

**Extensibility:** Users can create custom modules via community templates

---

### Offline Mode
**Definition:** PWA capability allowing read-only access and change queueing without network.

**Capabilities:**
- View cached dashboards
- Queue changes to The Bin
- Sync when reconnected

---

### Profile
**Definition:** A user's collection of active dashboards for a specific project or workflow.

**Properties:**
- One profile per project
- Profiles can be exported/imported (YAML format)
- Templates provide pre-configured profiles

---

### Propagation
**Definition:** Automatic update of dependent artifacts when source changes.

**Modes:**
- **Auto-Propagate:** Changes flow immediately (requires explicit config)
- **Gated Propagation:** Changes flagged for review (DEFAULT)
- **Hybrid:** Auto for low-impact, gated for high-impact

---

### Single Active Screen
**Definition:** Architectural constraint where user can only edit one dashboard at a time.

**Rationale:** Prevents concurrent write conflicts in single-user MVP

---

### SSE (Server-Sent Events)
**Definition:** Real-time update mechanism for dashboard synchronization.

**Purpose:** Push updates from Go API to Elm frontend without polling

---

### Template
**Definition:** Pre-configured workflow specification including:
- Dashboard layouts
- Module configurations
- Default guardrails
- AI prompt templates

**MVP Templates:** Vibe Coding, Data Science, Note Taking

---

### Token Budget
**Definition:** Limit on AI API token consumption.

**Scopes:**
- Per session: 100K tokens/hour
- Per dashboard: 10K tokens/request
- Per tool call: 4K tokens

**Purpose:** Prevent runaway costs and cascading AI calls

---

## Metadata & Temporal Concepts

### 5-Layer Versioning Stack
**Definition:** Comprehensive version tracking hierarchy across the DevGuide Cockpit system.

**Layers:**
1. **Filesystem Layer:** Linux timestamps (atime, mtime, ctime)
2. **Git Layer:** Commit history with author/committer dates
3. **Database Layer:** TiDB MVCC timestamps
4. **Application Layer:** Semantic Versioning with Zulu build metadata
5. **Project Layer:** Checkpoint snapshots

**Cross-Reference:** See *MAC Time*, *Zulu-First Paradigm*

---

### Clock Drift
**Definition:** Deviation between a system clock and an authoritative time source (NTP/PTP). Target threshold: <10ms across all Go API servers.

**Monitoring:** Tracked via `clock_sync_status` table in TiDB

**Mitigation:** NTP synchronization with multiple stratum-1 servers

---

### LPDS (Layered Prefix Descriptor System)
**Definition:** Machine-actionable file naming convention that embeds semantic context in filenames.

**Pattern:** `<entity>_<type>-<descriptor>.<ext>`

**Examples:**
- `bin_validator-guardrails.rs`
- `dashboard_template-vibe-coding.yaml`
- `api_endpoint-user-auth.go`

**Purpose:** Enables automated categorization, search, and dependency analysis

---

### MAC Time
**Definition:** Forensic timestamp triad from Linux filesystems used for audit trails.

**Components:**
- **mtime (Modified):** Data content change timestamp
- **atime (Accessed):** File read/execution timestamp
- **ctime (Changed):** Metadata modification timestamp

**Cross-Reference:** See *5-Layer Versioning Stack*, *Timestomping*

---

### Metadata Governance Dashboard
**Definition:** The foundational cross-cutting dashboard that standardizes temporal references, nomenclature protocols, and versioning metadata across all DevGuide Cockpit dashboards.

**Responsibilities:**
- Enforce Zulu-first timestamps
- Validate LPDS naming conventions
- Track version consistency across layers

**Enforcement:** Compliance validated via The Bin validation layer

**Cross-Reference:** See *The Bin*, *Zulu-First Paradigm*, *LPDS*

---

### Metadata Mini-Protocol
**Definition:** Abbreviated "dogfooding" implementation of metadata governance standards for the DevGuide Cockpit project itself.

**Three Core Rules:**
1. All timestamps end in Z (Zulu/UTC only)
2. Names are machine-searchable (lowercase, hyphens, descriptive)
3. Versions follow SemVer (MAJOR.MINOR.PATCH)

**Purpose:** Ensures project artifacts practice what the system preaches

---

### Timestomping
**Definition:** Malicious practice of modifying filesystem timestamps to hide evidence of file changes or access.

**Detection Strategy:** Monitor `utimensat` syscalls via auditd for unauthorized timestamp modifications

**Mitigation:** Immutable audit logs in TiDB with MVCC protection

**Cross-Reference:** See *MAC Time*

---

### Zulu-First Paradigm
**Definition:** Architectural principle where all internal system logic operates exclusively in UTC (Coordinated Universal Time) with "Z" suffix indicator.

**Implementation:**
- Storage: All timestamps stored as UTC
- APIs: All datetime fields use ISO 8601 with Z suffix
- Presentation: Local time calculated only at UI rendering layer

**Rationale:** Eliminates timezone ambiguity in distributed systems and audit logs

**Cross-Reference:** See *Metadata Governance Dashboard*, *5-Layer Versioning Stack*

---

**End of ARTIFACT_02_TERMINOLOGY_GLOSSARY.md**
