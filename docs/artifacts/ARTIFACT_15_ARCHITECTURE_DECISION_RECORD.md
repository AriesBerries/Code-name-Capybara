# DevGuide Cockpit: Architecture Decision Records

**Document Type:** Technical Implementation (Tier 5)  
**Status:** Active  
**Version:** 1.1  
**Date:** 2026-01-02T00:00:00Z  
**Dependencies:** All handoffs and completed artifacts

---

## Executive Summary

This artifact documents the **six critical architectural decisions** that define DevGuide Cockpit's technical foundation. Each ADR follows the [Markdown Architectural Decision Records (MADR)](https://adr.github.io/madr/) format.

**ADRs Documented:**
1. **ADR-001**: Elm-Chromium Stack (No React)
2. **ADR-002**: TiDB Over PostgreSQL (HTAP Architecture)
3. **ADR-003**: Three-Language Limit (Elm/Go/Rust Only)
4. **ADR-004**: Provider-Agnostic AI Layer
5. **ADR-007**: Zulu-First Temporal Architecture
6. **ADR-008**: LPDS (Layered Prefix Descriptor System) Nomenclature

---

## ADR-001: Elm-Chromium Stack (No React)

**Status:** ✅ Accepted  
**Date:** 2025-12-28  
**Deciders:** Architecture Team  

### Context and Problem Statement

DevGuide Cockpit requires a frontend framework for building dashboards. The framework must:
- Support complex state management (The Bin, propagation engine)
- Enable type-safe code generation (Elm templates for dashboards)
- Guarantee zero runtime exceptions in production
- Integrate seamlessly with Rust/WASM validation kernel
- Support offline-first PWA architecture

### Decision Drivers

* **Critical:** No runtime exceptions - dashboards must be bulletproof
* **Critical:** Type-safe code generation - templates must compile correctly
* **Important:** Predictable performance - no React virtual DOM overhead
* **Important:** Small bundle size - offline app must load fast
* **Nice-to-have:** Developer familiarity (React has larger community)

### Considered Options

1. **Elm** - Functional, no runtime exceptions, compiler-enforced architecture
2. **React + TypeScript** - Industry standard, large ecosystem
3. **Svelte** - Fast, reactive, smaller bundle size than React
4. **Vue** - Progressive framework, good documentation

### Decision Outcome

**Chosen option: Elm**, because:
- **Zero runtime exceptions** - compiler catches all errors at build time
- **Enforced architecture** - The Elm Architecture (TEA) prevents spaghetti code
- **Immutable by default** - critical for The Bin's conflict-free updates
- **Superior refactoring** - compiler guarantees correct refactors
- **Excellent error messages** - compiler guides developers through fixes

**Negative consequences:**
- Smaller talent pool (Elm developers less common than React)
- Fewer third-party libraries (custom implementations needed)
- Steeper learning curve for new contributors
- No escape hatch to JavaScript when needed (must use ports)

### Pros and Cons of the Options

**Elm:**
* ✅ No runtime exceptions (guaranteed by compiler)
* ✅ Immutable data structures (perfect for The Bin)
* ✅ Time-traveling debugger (critical for dashboard development)
* ✅ Enforced TEA architecture (prevents anti-patterns)
* ✅ Excellent refactoring support
* ❌ Smaller ecosystem
* ❌ Learning curve for contributors
* ❌ Ports required for JS interop

**React + TypeScript:**
* ✅ Large ecosystem and talent pool
* ✅ Many third-party libraries
* ✅ JSX familiar to most developers
* ❌ Runtime exceptions possible (null checks, async errors)
* ❌ Virtual DOM overhead
* ❌ Easy to write bad code (no enforced architecture)
* ❌ TypeScript doesn't prevent runtime errors

**Svelte:**
* ✅ Fast (compiles to vanilla JS, no virtual DOM)
* ✅ Small bundle size
* ✅ Reactive by default
* ❌ Runtime exceptions possible
* ❌ Smaller ecosystem than React
* ❌ Less mature than Elm for large apps

**Vue:**
* ✅ Progressive (can adopt incrementally)
* ✅ Good documentation
* ✅ Template syntax familiar to HTML developers
* ❌ Runtime exceptions possible
* ❌ Reactivity system can be confusing
* ❌ Composition API less type-safe than Elm

### Validation

**Success Criteria:**
- [ ] Zero production runtime errors in dashboards (measured quarterly)
- [ ] Dashboard template compilation success rate > 99%
- [ ] Contributor onboarding time < 2 weeks (Elm tutorials required)
- [ ] Bundle size < 500 KB (gzipped) for MVP dashboards

**Risks:**
- **Risk**: Elm compiler slow for large projects  
  **Mitigation**: Incremental compilation, split modules into packages
- **Risk**: Third-party JS library integration difficult  
  **Mitigation**: Use ports, create Elm wrappers for critical libraries
- **Risk**: Difficulty hiring Elm developers  
  **Mitigation**: Invest in Elm training, document best practices

---

## ADR-002: TiDB Over PostgreSQL (HTAP Architecture)

**Status:** ✅ Accepted  
**Date:** 2025-12-29  
**Deciders:** Architecture Team, Data Team  

### Context and Problem Statement

DevGuide Cockpit requires a database that supports:
- **OLTP**: Real-time dashboard updates, The Bin validation, project CRUD
- **OLAP**: Analytics queries (change impact analysis, token usage trends)
- Horizontal scaling for future growth
- Strong consistency for The Bin atomic commits
- Cloud deployment on Microsoft Azure

Original plan specified PostgreSQL, but analytics queries are expected to slow down OLTP performance as the system grows.

### Decision Drivers

* **Critical:** HTAP capability - run analytics without degrading OLTP
* **Critical:** Horizontal scaling - handle 1000+ concurrent projects
* **Critical:** Azure integration - leverage Microsoft Fabric for AI
* **Important:** Zero infrastructure cost for MVP (free tier)
* **Important:** Familiar SQL syntax (team knows PostgreSQL)
* **Nice-to-have:** Compatibility with existing PostgreSQL tools

### Considered Options

1. **TiDB Cloud on Azure** - HTAP, horizontally scalable MySQL-compatible database
2. **PostgreSQL + ClickHouse** - Separate OLTP and OLAP databases with sync
3. **Azure Cosmos DB** - Globally distributed NoSQL with analytics
4. **PostgreSQL + Citus** - Distributed PostgreSQL extension

### Decision Outcome

**Chosen option: TiDB Cloud on Azure**, because:
- **HTAP in one database** - no need to sync between OLTP and OLAP systems
- **TiFlash columnar storage** - analytics queries run on separate storage without impacting OLTP
- **Horizontal scaling** - add TiKV nodes as data grows
- **Azure Fabric integration** - direct connection for AI/ML workloads
- **Free starter tier** - $0 infrastructure cost during MVP (5 GiB storage, 50M RU/month)
- **MySQL compatibility** - most PostgreSQL queries work with minor changes

**Supersedes:** Previous decision to use PostgreSQL (no formal ADR existed)

**Migration Impact:**
- SQL DDL changes required (PostgreSQL-specific features like JSONB → JSON)
- Query rewrites for TiDB syntax (some PostgreSQL functions unavailable)
- CI/CD pipeline updates (use TiDB client instead of psql)
- Team learning curve (TiDB-specific features like TiFlash)

### Pros and Cons of the Options

**TiDB Cloud:**
* ✅ HTAP (OLTP + OLAP in single database)
* ✅ Horizontal scaling without downtime
* ✅ Azure Fabric integration (AI/analytics)
* ✅ Free starter tier ($0 for MVP)
* ✅ MySQL-compatible (familiar SQL)
* ✅ Distributed transactions (critical for The Bin atomicity)
* ❌ Newer technology (less mature than PostgreSQL)
* ❌ Smaller community and fewer tools
* ❌ Learning curve for TiDB-specific features

**PostgreSQL + ClickHouse:**
* ✅ Mature technologies, large community
* ✅ Best-in-class OLTP (PostgreSQL) and OLAP (ClickHouse)
* ❌ Two databases to maintain
* ❌ Synchronization complexity (ETL pipeline required)
* ❌ No Azure Fabric integration out-of-the-box
* ❌ Higher infrastructure cost (two databases)

**Azure Cosmos DB:**
* ✅ Globally distributed
* ✅ Multi-model (document, graph, key-value)
* ✅ Native Azure integration
* ❌ NoSQL (requires schema redesign)
* ❌ Query language (SQL API not full SQL)
* ❌ Higher cost than TiDB free tier
* ❌ Not ideal for relational data (projects, dashboards)

**PostgreSQL + Citus:**
* ✅ Distributed PostgreSQL (familiar)
* ✅ Horizontal scaling via sharding
* ❌ No separate OLAP storage (analytics slow down OLTP)
* ❌ Shard key design complexity
* ❌ Limited Azure support (not fully managed)
* ❌ Higher cost than TiDB starter tier

### Validation

**Success Criteria:**
- [ ] Analytics queries (p95) < 500ms using TiFlash
- [ ] OLTP query latency unchanged with analytics workload
- [ ] Horizontal scaling demonstration (add TiKV node without downtime)
- [ ] Zero data loss during TiDB cluster upgrades

**Risks:**
- **Risk**: TiDB less mature than PostgreSQL  
  **Mitigation**: Use TiDB Cloud (managed service), leverage PingCAP support
- **Risk**: Unexpected incompatibilities with PostgreSQL  
  **Mitigation**: Comprehensive migration testing, fallback plan to PostgreSQL
- **Risk**: Cost overruns on TiDB Cloud  
  **Mitigation**: Monitor RU usage, implement query caching, optimize before scaling

### References

* [TiDB Documentation](https://docs.pingcap.com/tidb/stable)
* [TiDB Cloud Pricing](https://www.pingcap.com/tidb-cloud-pricing/)
* [Microsoft Fabric + TiDB Integration](https://www.pingcap.com/blog/tidb-microsoft-fabric/)

---

## ADR-003: Three-Language Limit (Elm/Go/Rust Only)

**Status:** ✅ Accepted  
**Date:** 2025-12-28  
**Deciders:** Architecture Team  

### Context and Problem Statement

Software projects often accumulate programming languages over time, leading to:
- Increased cognitive load for contributors
- Duplicated tooling (linters, formatters, CI pipelines)
- Difficulty onboarding new developers
- Security vulnerabilities spread across language ecosystems

DevGuide Cockpit must enforce a strict language limit to prevent "language sprawl."

### Decision Drivers

* **Critical:** Minimize cognitive load - contributors should master 3 languages, not 10
* **Critical:** Clear separation of concerns - each language has a specific role
* **Important:** Interoperability - languages must communicate efficiently
* **Important:** Tooling consolidation - reduce CI/CD complexity
* **Nice-to-have:** Performance - languages should be fast enough for their roles

### Considered Options

1. **Elm + Go + Rust** (chosen)
2. **TypeScript + Node.js + Python** (fullstack JS/Python)
3. **React + Java + Python** (enterprise stack)
4. **Svelte + Rust + Deno** (modern stack)

### Decision Outcome

**Chosen option: Elm + Go + Rust**, because:

**Language Roles:**
1. **Elm** - Frontend dashboards (UI, state management, user interactions)
2. **Go** - API orchestration (HTTP handlers, business logic, TiDB queries)
3. **Rust** - Validation kernel (guardrails, The Bin logic, WASM compilation)

**Rationale:**
- **Elm**: Zero runtime exceptions, enforced architecture (perfect for dashboards)
- **Go**: Fast compilation, excellent concurrency (goroutines for SSE, API), simple deployment
- **Rust**: Memory safety, WASM support (critical for browser-based validation), zero-cost abstractions

**Interoperability:**
```
Elm (Frontend)
  ↔ HTTP/SSE
Go API
  ↔ FFI / Subprocess
Rust Validation Kernel
  ↔ WASM
Browser (Elm runtime)
```

**Negative consequences:**
- Steeper learning curve (3 languages to learn vs 1-2 in typical stacks)
- Fewer developers proficient in all three
- Cannot use JavaScript libraries directly in Elm (must use ports)

### Pros and Cons of the Options

**Elm + Go + Rust:**
* ✅ Each language best-in-class for its role
* ✅ Strong type systems (compile-time guarantees)
* ✅ Excellent performance (Go fast enough, Rust zero-overhead)
* ✅ Clear separation of concerns
* ❌ Learning curve for contributors
* ❌ Smaller talent pool (especially Elm)

**TypeScript + Node.js + Python:**
* ✅ JavaScript familiarity (large talent pool)
* ✅ Unified language for frontend/backend (TypeScript)
* ✅ Rich ecosystem (npm, PyPI)
* ❌ Runtime exceptions (not eliminated, only mitigated by TypeScript)
* ❌ Performance issues (Node.js slower than Go, Python slower than Rust)
* ❌ Two runtimes (Node + Python interpreter)

**React + Java + Python:**
* ✅ Enterprise-standard stack
* ✅ Strong typing (TypeScript + Java)
* ✅ Mature ecosystems
* ❌ Three runtimes (Node, JVM, Python)
* ❌ Heavy infrastructure (JVM memory overhead)
* ❌ Complexity (Spring Boot, React boilerplate)

**Svelte + Rust + Deno:**
* ✅ Modern stack (cutting-edge technologies)
* ✅ Performance (Svelte + Rust both fast)
* ✅ Unified runtime (Deno for backend and build tools)
* ❌ Deno less mature than Node.js or Go
* ❌ Smaller ecosystems (Svelte, Deno)
* ❌ Runtime exceptions (Svelte not as safe as Elm)

### Enforcement Mechanisms

**CI/CD Pipeline:**
```yaml
# .github/workflows/language-lint.yml
- name: Detect Forbidden Languages
  run: |
    # Fail if any files with forbidden extensions found
    FORBIDDEN=$(find . -type f \( \
      -name "*.py" -o \
      -name "*.js" -o \
      -name "*.ts" -o \
      -name "*.java" \
    \) | grep -v node_modules | grep -v vendor)
    
    if [ -n "$FORBIDDEN" ]; then
      echo "Forbidden language files detected:"
      echo "$FORBIDDEN"
      exit 1
    fi
```

**Exceptions:**
- **JavaScript**: Allowed only in service worker (`service-worker.js`) and build scripts (`build.js`)
- **Shell scripts**: Allowed for CI/CD and deployment automation
- **Configuration files**: YAML, TOML, JSON allowed

### Validation

**Success Criteria:**
- [ ] 100% of application code in Elm/Go/Rust (excluding exceptions)
- [ ] Contributors can become proficient in all 3 languages within 3 months
- [ ] CI/CD detects forbidden languages and fails build
- [ ] No runtime exceptions from type errors (measured in production)

**Risks:**
- **Risk**: Cannot hire developers proficient in all 3 languages  
  **Mitigation**: Hire for 1-2 languages, train on the others, pair programming
- **Risk**: Third-party library requires JavaScript  
  **Mitigation**: Create Elm ports, or reimplement in Rust/Go if critical
- **Risk**: Performance bottleneck in a language  
  **Mitigation**: Profile first, then optimize or rewrite critical path in Rust

---

## ADR-004: Provider-Agnostic AI Layer

**Status:** ✅ Accepted  
**Date:** 2025-12-29  
**Deciders:** Architecture Team, AI Team  

### Context and Problem Statement

DevGuide Cockpit relies on AI for:
- Governance decision assistance (The Bin validation)
- Context-aware help across dashboards
- Code generation and refactoring suggestions
- Documentation generation

The AI provider landscape is rapidly evolving. Locking into a single provider (e.g., Claude) creates:
- Vendor lock-in risk
- Inability to leverage multiple models for different tasks
- Pricing unpredictability
- Compliance issues (some orgs cannot use certain providers)

### Decision Drivers

* **Critical:** Vendor independence - must be able to switch providers without code changes
* **Critical:** Multi-provider support - use Claude for reasoning, local model for code completion
* **Important:** Graceful degradation - system functions without AI (reduced capabilities)
* **Important:** Consistent interface - dashboards interact with AI the same way regardless of provider
* **Nice-to-have:** Cost optimization - route queries to cheapest suitable model

### Considered Options

1. **Provider-Agnostic Interface** (chosen) - abstraction layer with swappable backends
2. **Claude-Only** - hard-code Claude API, optimize for single provider
3. **LangChain** - use LangChain framework for multi-provider support
4. **OpenAI SDK + Adapters** - use OpenAI SDK, write adapters for other providers

### Decision Outcome

**Chosen option: Provider-Agnostic Interface**, because:
- **Abstraction layer**: All dashboards call `AIProvider` trait, never directly call Claude/OpenAI/etc.
- **Dependency injection**: Provider swapped via config without code changes
- **Multi-provider routing**: Can use Claude for reasoning, local model for code completion simultaneously
- **Graceful degradation**: If AI unavailable, dashboards fall back to static suggestions

**Architecture:**

```rust
// Trait defining AI provider interface
pub trait AIProvider: Send + Sync {
    async fn complete(&self, prompt: &str, max_tokens: u32) -> Result<AIResponse, AIError>;
    async fn embed(&self, text: &str) -> Result<Vec<f32>, AIError>;
    fn supports_tool_use(&self) -> bool;
}

// Concrete implementations
pub struct ClaudeProvider { /* ... */ }
pub struct OpenAIProvider { /* ... */ }
pub struct LocalLlamaProvider { /* ... */ }

impl AIProvider for ClaudeProvider {
    async fn complete(&self, prompt: &str, max_tokens: u32) -> Result<AIResponse, AIError> {
        // Call Anthropic API
    }
}

// Dependency injection
pub fn create_provider(config: &AIConfig) -> Box<dyn AIProvider> {
    match config.provider_type {
        ProviderType::Claude => Box::new(ClaudeProvider::new(config.api_key)),
        ProviderType::OpenAI => Box::new(OpenAIProvider::new(config.api_key)),
        ProviderType::LocalLlama => Box::new(LocalLlamaProvider::new(config.model_path)),
    }
}
```

**MVP Default:** Claude (Sonnet 4.5) - superior reasoning for governance decisions

**Negative consequences:**
- Additional abstraction layer complexity
- Cannot use provider-specific features (e.g., Claude's extended thinking)
- Must define lowest common denominator interface

### Pros and Cons of the Options

**Provider-Agnostic Interface:**
* ✅ Vendor independence (switch providers without code changes)
* ✅ Multi-provider support (route by task type)
* ✅ Future-proof (new providers easy to add)
* ✅ Graceful degradation (fallback to cached responses)
* ❌ Abstraction overhead (cannot use provider-specific features)
* ❌ Lowest common denominator API
* ❌ More code to maintain (adapter for each provider)

**Claude-Only:**
* ✅ Simpler code (no abstraction layer)
* ✅ Can use Claude-specific features (extended thinking, computer use)
* ✅ Optimized for single provider
* ❌ Vendor lock-in (cannot switch without rewrite)
* ❌ Single point of failure (if Claude down, no AI)
* ❌ Pricing risk (Claude price changes impact us directly)

**LangChain:**
* ✅ Multi-provider support out-of-the-box
* ✅ Many pre-built integrations
* ✅ Prompt templating and chaining
* ❌ Heavy dependency (Python-based, requires Python runtime)
* ❌ Overkill for our needs (we don't need chains/agents)
* ❌ Abstracts away too much (harder to optimize)

**OpenAI SDK + Adapters:**
* ✅ Industry-standard API (OpenAI-compatible providers common)
* ✅ Simple SDK
* ❌ Still vendor-specific (assumes OpenAI-compatible format)
* ❌ Adapter complexity (must map non-OpenAI providers to OpenAI format)
* ❌ Missing features (Claude's extended thinking not in OpenAI format)

### Implementation Strategy

**Phase 1 (MVP):** Claude-only implementation behind `AIProvider` interface
**Phase 2 (Post-MVP):** Add OpenAI provider for comparison
**Phase 3:** Add local model (Llama) for offline/privacy scenarios
**Phase 4:** Multi-provider routing (Claude for reasoning, local for code completion)

**Configuration:**

```yaml
# config/ai.yaml
ai:
  providers:
    - name: claude
      type: anthropic
      api_key: ${CLAUDE_API_KEY}
      model: claude-sonnet-4-20250514
      max_tokens: 4000
      enabled: true
      
    - name: openai
      type: openai
      api_key: ${OPENAI_API_KEY}
      model: gpt-4-turbo
      max_tokens: 4000
      enabled: false  # Disabled for MVP
      
    - name: local
      type: llama
      model_path: /models/llama-3-8b.gguf
      max_tokens: 2000
      enabled: false  # Disabled for MVP
  
  routing:
    default: claude
    code_completion: local  # Post-MVP
    reasoning: claude
```

### Validation

**Success Criteria:**
- [ ] Can switch from Claude to OpenAI with config change only (no code changes)
- [ ] Multi-provider routing works (Claude for reasoning, local for code completion)
- [ ] Graceful degradation when AI provider unavailable
- [ ] 100% of AI calls go through `AIProvider` interface (no direct API calls)

**Risks:**
- **Risk**: Abstraction layer limits performance  
  **Mitigation**: Profile AI calls, optimize hot paths, cache aggressively
- **Risk**: Provider-specific features unavailable  
  **Mitigation**: Define optional capabilities (e.g., `supports_tool_use()`), use when available
- **Risk**: Adapter complexity for non-standard providers  
  **Mitigation**: Focus on OpenAI-compatible providers first (Claude, OpenAI, Anthropic)

### References

* [Anthropic API Documentation](https://docs.anthropic.com/en/api/messages)
* [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
* [Unified AI Provider Pattern](https://github.com/features/copilot/api)

---

## ADR-007: Zulu-First Temporal Architecture

**Status:** ✅ Accepted  
**Date:** 2026-01-01T00:00:00Z  
**Deciders:** Architecture Team  
**Technical Area:** Time Handling

### Context and Problem Statement

DevGuide Cockpit operates across time zones (distributed team, global users). Timestamp inconsistencies cause merge conflicts, audit trail confusion, and incorrect event ordering.

### Decision Drivers

* **Critical:** Consistent event ordering across all systems
* **Critical:** Unambiguous timestamp interpretation
* **Important:** Simplified debugging and log analysis
* **Important:** Standards compliance
* **Nice-to-have:** Tool compatibility without reconfiguration

### Considered Options

1. **Zulu-First (UTC with Z suffix)** - All internal timestamps use ISO 8601 with UTC
2. **Server Local Time** - Store timestamps in server's timezone
3. **User Local Time** - Store timestamps in each user's timezone
4. **Unix Timestamps** - Store as integer seconds since epoch

### Decision Outcome

**Chosen option: Zulu-First (UTC with Z suffix)**, because all internal timestamps use ISO 8601 with UTC ("Z" suffix). Local time rendering happens only at presentation layer.

**Rationale:**
- **Consistency:** No ambiguity in event ordering
- **Portability:** Same timestamp renders correctly everywhere
- **Debugging:** Log analysis doesn't require timezone conversion
- **Standards:** ISO 8601 is widely supported

### Implementation

1. **TiDB** stores `TIMESTAMP(6)` fields (microsecond precision)
2. **Go APIs** accept only Zulu-format timestamps
3. **Elm UI** converts to local time for display only
4. **Rust validation** rejects non-Z timestamps

**Example Format:**
```
2026-01-01T14:30:00.123456Z
```

**Validation Regex (Rust):**
```rust
const ZULU_TIMESTAMP: &str = r"^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?Z$";
```

**Go API Validation:**
```go
func ValidateZuluTimestamp(ts string) error {
    _, err := time.Parse(time.RFC3339Nano, ts)
    if err != nil || !strings.HasSuffix(ts, "Z") {
        return errors.New("timestamp must be in Zulu format (UTC with Z suffix)")
    }
    return nil
}
```

**Elm Display Conversion:**
```elm
displayLocalTime : Time.Zone -> Time.Posix -> String
displayLocalTime zone posix =
    -- Convert UTC timestamp to user's local timezone for display only
    formatTime zone posix
```

### Consequences

**Positive:**
* ✅ Eliminates timezone bugs
* ✅ Simplifies logging and debugging
* ✅ Ensures consistent event ordering
* ✅ Reduces merge conflicts in collaborative editing

**Negative:**
* ❌ Users must mentally convert for debugging raw data
* ❌ Existing tools may need reconfiguration

**Neutral:**
- Display code slightly more complex (but centralized in presentation layer)

### Validation Criteria

- [ ] 100% of new timestamps use Z suffix
- [ ] CI/CD rejects commits with local timestamps in code
- [ ] Documentation updated with examples
- [ ] All API responses include only Zulu timestamps

**Risks:**
- **Risk**: Legacy data contains non-Zulu timestamps  
  **Mitigation**: Migration script to normalize existing data
- **Risk**: Third-party integrations expect local time  
  **Mitigation**: Transform at integration boundary only

### Related Decisions

- ADR-002: TiDB Selection (timestamp storage with microsecond precision)
- ADR-008: LPDS Nomenclature (consistency principle)

---

## ADR-008: LPDS (Layered Prefix Descriptor System) Nomenclature

**Status:** ✅ Accepted  
**Date:** 2026-01-01T00:00:00Z  
**Deciders:** Architecture Team  
**Technical Area:** File Naming

### Context and Problem Statement

Inconsistent file naming across the codebase makes discovery difficult and creates confusion about file purpose. Developers waste time searching for files and understanding their role in the system.

### Decision Drivers

* **Critical:** Fast file discovery across large codebase
* **Critical:** Self-documenting file structure
* **Important:** Machine-readable patterns for tooling
* **Important:** Team-wide consistency
* **Nice-to-have:** IDE autocomplete friendliness

### Considered Options

1. **LPDS Pattern** - `<entity>_<type>-<descriptor>.<ext>`
2. **Flat Naming** - Simple descriptive names without structure
3. **Directory-Based** - Organize by directory, use simple filenames
4. **Hungarian Notation** - Type prefix on all files

### Decision Outcome

**Chosen option: LPDS (Layered Prefix Descriptor System)**, because all new files follow the pattern: `<entity>_<type>-<descriptor>.<ext>`

**Rationale:**
- **Discoverability:** Files sort logically by entity
- **Machine-Readable:** Pattern is regex-parseable for tooling
- **Self-Documenting:** Filename describes contents
- **Consistency:** Team-wide standard reduces cognitive load

### Implementation

**Pattern Structure:**
```
<entity>_<type>-<descriptor>.<ext>

Where:
  entity     = Primary domain (bin, api, dashboard, config, etc.)
  type       = File category (validator, handler, panel, model, etc.)
  descriptor = Specific purpose (guardrails, auth, metrics, etc.)
  ext        = File extension (.rs, .go, .elm, etc.)
```

**Examples:**

| File Name | Entity | Type | Descriptor | Language |
|-----------|--------|------|------------|----------|
| `bin_validator-guardrails.rs` | bin | validator | guardrails | Rust |
| `api_handler-auth.go` | api | handler | auth | Go |
| `dashboard_panel-metrics.elm` | dashboard | panel | metrics | Elm |
| `config_schema-project.yaml` | config | schema | project | YAML |
| `test_unit-propagation.rs` | test | unit | propagation | Rust |

**Validation Regex:**
```rust
const LPDS_PATTERN: &str = r"^[a-z]+_[a-z]+-[a-z]+\.[a-z]+$";
```

**GitHub Action Enforcement:**
```yaml
# .github/workflows/lpds-check.yml
- name: Validate LPDS Naming
  run: |
    INVALID=$(find src -type f \( -name "*.rs" -o -name "*.go" -o -name "*.elm" \) \
      | grep -vE '^[a-z]+_[a-z]+-[a-z]+\.[a-z]+$' || true)
    if [ -n "$INVALID" ]; then
      echo "Files not following LPDS pattern:"
      echo "$INVALID"
      exit 1
    fi
```

### Consequences

**Positive:**
* ✅ Faster file discovery (sort by entity)
* ✅ Self-documenting structure
* ✅ Reduces naming conflicts
* ✅ Enables automated tooling (linting, documentation generation)

**Negative:**
* ❌ Longer filenames (but more descriptive)
* ❌ Requires team training on pattern

**Neutral:**
- Existing files can be grandfathered or migrated slowly
- IDE support varies (most handle long names well)

### Validation Criteria

- [ ] >90% of new files follow LPDS pattern
- [ ] GitHub Action enforces pattern on PRs
- [ ] Documentation updated with examples
- [ ] Team onboarding includes LPDS training

**Risks:**
- **Risk**: Inconsistent adoption during transition  
  **Mitigation**: Grandfather existing files, enforce only on new files initially
- **Risk**: Edge cases don't fit pattern  
  **Mitigation**: Document exceptions, allow override with justification in PR

### Related Decisions

- ADR-007: Zulu-First Temporal Architecture (consistency principle)
- ADR-003: Three-Language Limit (language-specific extensions)

---

## Cross-ADR Dependencies

**Dependency Graph:**

```
ADR-003 (Languages)
  ├─> ADR-001 (Elm)       # Elm is one of the 3 languages
  └─> ADR-004 (AI Layer)   # AI layer implemented in Go (orchestration) + Rust (validation)

ADR-002 (TiDB)
  └─> ADR-001 (Elm)       # Elm dashboards query TiDB via Go API
  └─> ADR-007 (Zulu-First) # TiDB stores timestamps with microsecond precision

ADR-004 (AI Layer)
  └─> ADR-001 (Elm)       # Elm dashboards call AI via ports

ADR-007 (Zulu-First)
  └─> ADR-002 (TiDB)      # Timestamp storage format
  └─> ADR-008 (LPDS)      # Consistency principle shared

ADR-008 (LPDS)
  └─> ADR-003 (Languages) # File extensions tied to language limits
  └─> ADR-007 (Zulu-First) # Consistency principle shared
```

**Conflicts:** None - all ADRs are compatible.

---

## ADR Review Schedule

**Frequency:** Quarterly  
**Next Review:** April 1, 2026  

**Review Criteria:**
- Are the decisions still valid given new information?
- Have any negative consequences materialized?
- Should any ADRs be superseded?

**Change Process:**
1. Propose new ADR or amendment to existing ADR
2. Present to Architecture Team with evidence
3. Discuss alternatives and trade-offs
4. Vote (requires 2/3 majority to accept)
5. Update ADR status (Accepted/Superseded/Deprecated)
6. Communicate changes to all contributors

---

**End of ARTIFACT_15_ARCHITECTURE_DECISION_RECORD.md**
