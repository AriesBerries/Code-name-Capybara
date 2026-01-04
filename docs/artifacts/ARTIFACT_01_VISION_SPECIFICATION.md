# DevGuide Cockpit: Vision Specification

**Document Type:** Foundation  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** None (root document)

---

## Executive Summary

**DevGuide Cockpit** is an **Internal Developer Platform (IDP) with a low-code/no-code governance layer** that transforms software development workflows into a synchronized, AI-assisted ecosystem. Unlike traditional dashboards that merely display information, DevGuide Cockpit provides:

- **Active Workflow Orchestration:** Modular dashboards that execute development tasks, not just monitor them
- **Structural Impossibility of Desynchronization:** Single source of truth with guaranteed propagation across all artifacts
- **AI-Native Architecture:** Unified reasoning layer providing context-aware assistance throughout the development lifecycle
- **Extreme UI/UX Customization:** Every dashboard, module, and interaction pattern can be tailored without code changes

---

## Product Identity

### What DevGuide Cockpit Is

**Primary Classification:** Internal Developer Platform (IDP)  
**Secondary Classification:** Low-Code/No-Code Governance Layer  
**Tertiary Classification:** AI-Augmented Development Environment

**Core Value Proposition:**
> "Truth has a single origin point, and everything else (code, docs, tests, specs) are projections of that truth. The system is structurally impossible to desynchronize."

### What DevGuide Cockpit Is NOT

- ❌ **Not a task tracking tool** (like Jira or Linear)
- ❌ **Not a CI/CD platform** (like Jenkins or GitHub Actions) - though it integrates with them
- ❌ **Not a code editor** (like VS Code) - though it orchestrates editing workflows
- ❌ **Not a documentation generator** (like Sphinx or Docusaurus) - though it ensures docs stay synchronized

DevGuide Cockpit sits **above** these tools, orchestrating them into a coherent workflow where changes propagate automatically and governance is enforced at every checkpoint.

---

## Problem Statement

### The Desynchronization Crisis

Traditional software development suffers from **structural desynchronization**:

1. **Documentation Drift:** README describes v1.0 features while codebase is at v3.2
2. **Test Staleness:** Test suite validates outdated specifications
3. **Decision Amnesia:** Architectural decisions lost in Slack threads or email archives
4. **Checklist Fatigue:** Manual checklists (pre-commit, deployment, security) ignored under time pressure
5. **Tool Fragmentation:** 10+ disconnected tools (IDE, issue tracker, docs, CI/CD, monitoring)

**Result:** Cognitive overhead, bugs from inconsistent state, onboarding nightmares.

### Why Existing Solutions Fail

| Tool Category | Why It Fails |
|---------------|--------------|
| **IDEs** | Focus on code editing, not workflow orchestration |
| **Project Management** | Track tasks, not knowledge synchronization |
| **CI/CD Platforms** | Automate deployment, not decision validation |
| **Documentation Generators** | Render static content, no bidirectional sync |
| **AI Coding Assistants** | Provide suggestions, no governance enforcement |

**Key Insight:** These tools solve **individual problems** but create **integration debt**. DevGuide Cockpit provides the **orchestration layer** that binds them into a synchronized system.

---

## Solution Architecture

### The Three-Layer Design

```
┌────────────────────────────────────────────────────────────┐
│  LAYER 1: Dashboard Presentation (Elm)                      │
│  - Drag-and-drop interface                                  │
│  - Real-time updates via Server-Sent Events                 │
│  - Offline-first with PWA capabilities                      │
└────────────────┬───────────────────────────────────────────┘
                 │
┌────────────────▼───────────────────────────────────────────┐
│  LAYER 2: Unified AI Layer (Provider-Agnostic)              │
│  - Context-aware assistance across all dashboards           │
│  - Guardrail validation reasoning                           │
│  - Token budget enforcement                                 │
└────────────────┬───────────────────────────────────────────┘
                 │
┌────────────────▼───────────────────────────────────────────┐
│  LAYER 3: The Bin (Rust Validation Kernel)                  │
│  - Synchronization checkpoint                               │
│  - Capability enforcement                                   │
│  - Dependency impact analysis                               │
└────────────────┬───────────────────────────────────────────┘
                 │
┌────────────────▼───────────────────────────────────────────┐
│  LAYER 4: Persistence & Orchestration (Go + TiDB)           │
│  - HTAP database for analytics + transactions               │
│  - Git integration for version control                      │
│  - External tool integrations (GitHub, Jira, Slack)         │
└────────────────────────────────────────────────────────────┘
```

---

## MVP Scope

### Workflow Templates (MVP)

**Included:**
1. **Vibe Coding:** 6 dashboards covering Project Inception → Deployment
2. **Data Science:** 7 dashboards covering Research Question → Publication  
3. **Note Taking:** 4 dashboards covering Note Capture → Knowledge Graph

### Technology Stack (MVP)

- **Frontend:** Elm + Chromium Embedded (CEF3)
- **Backend:** Go (API) + Rust (validation kernel → WASM)
- **Database:** TiDB (HTAP)
- **AI:** Claude Sonnet 4.5 (provider-agnostic interface)

---

## Success Criteria

| Metric | Target |
|--------|--------|
| Dashboard Load Time | < 500ms |
| Bin Validation Latency | < 200ms |
| AI Response Time | < 3s |
| Test Coverage | Elm: 90%, Rust: 95%, Go: 80% |
| Accessibility Score | ≥ 90 (WCAG AA) |

---

**End of ARTIFACT_01_VISION_SPECIFICATION.md**
