# Software Development Checklists & Templates
## A Unified Framework for AI-Assisted and Vibe Coding Projects

**Version:** 2.0.0  
**Date:** December 30, 2025  
**Status:** Comprehensive Reference Framework  
**Scope:** Universal (applies to greenfield, brownfield, and AI-native projects)

---

## Table of Contents

1. [Framework Overview](#1-framework-overview)
2. [Phase 0: Project Inception](#2-phase-0-project-inception)
3. [Phase 1: Architecture & Design](#3-phase-1-architecture--design)
4. [Phase 2: Technology Stack Decision](#4-phase-2-technology-stack-decision)
5. [Phase 3: Repository & Infrastructure Setup](#5-phase-3-repository--infrastructure-setup)
6. [Phase 4: Walking Skeleton Implementation](#6-phase-4-walking-skeleton-implementation)
7. [Phase 5: Core Feature Implementation](#7-phase-5-core-feature-implementation)
8. [Phase 6: Testing & Quality Assurance](#8-phase-6-testing--quality-assurance)
9. [Phase 7: Documentation](#9-phase-7-documentation)
10. [Phase 8: Deployment & Release](#10-phase-8-deployment--release)
11. [Phase 9: Maintenance & Evolution](#11-phase-9-maintenance--evolution)
12. [Appendix A: Templates](#appendix-a-templates)
13. [Appendix B: Atomic Coding Standards Quick Reference](#appendix-b-atomic-coding-standards-quick-reference)
14. [Appendix C: AI/Vibe Coding Best Practices](#appendix-c-aivibe-coding-best-practices)

---

## 1. Framework Overview

### 1.1 Purpose

This document provides a complete, phase-by-phase checklist system for software development projects. It synthesizes industry standards (arc42, MADR, Walking Skeleton) with AI-native development patterns and atomic coding principles.

**Designed for:**
- **Traditional development** (human-driven teams)
- **AI-assisted development** (Claude, Copilot, Continue.dev)
- **Vibe coding** (AI agents generating entire applications with minimal intervention)
- **Hybrid approaches** (human oversight with AI execution)

### 1.2 Core Principles

The framework is built on five foundational principles from the Multi-Phase AI-Assisted Development methodology:

| Principle | Description | Application |
|-----------|-------------|-------------|
| **Phase-Based Workflow** | Development divided into distinct phases with isolated context | Each phase has own Claude Project/context |
| **Atomic Steps** | Tasks broken into smallest meaningful units | One function, one test, one commit |
| **Just-in-Time Knowledge** | Load only information needed for current step | Progressive disclosure pattern |
| **Chain-of-Thought Protocols** | Decision-class-specific reasoning patterns | Debate format for architecture, hypothesis-test for debugging |
| **Artifact Hand-Off** | Completed artifacts become input for next phase | Phase outputs feed next phase |

### 1.3 Phase Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DEVELOPMENT PHASE FLOW                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Phase 0          Phase 1           Phase 2           Phase 3               │
│  ┌─────────┐      ┌─────────┐       ┌─────────┐       ┌─────────┐          │
│  │ PROJECT │  →   │ ARCH &  │   →   │  TECH   │   →   │  REPO   │          │
│  │INCEPTION│      │ DESIGN  │       │  STACK  │       │  SETUP  │          │
│  └────┬────┘      └────┬────┘       └────┬────┘       └────┬────┘          │
│       │                │                 │                 │                │
│       ▼                ▼                 ▼                 ▼                │
│   Charter.md      ArchReq.md        TechStack.md      Repo Ready            │
│                                                                              │
│  Phase 4          Phase 5           Phase 6           Phase 7               │
│  ┌─────────┐      ┌─────────┐       ┌─────────┐       ┌─────────┐          │
│  │ WALKING │  →   │  CORE   │   →   │ TESTING │   →   │  DOCS   │          │
│  │SKELETON │      │FEATURES │       │   & QA  │       │         │          │
│  └────┬────┘      └────┬────┘       └────┬────┘       └────┬────┘          │
│       │                │                 │                 │                │
│       ▼                ▼                 ▼                 ▼                │
│   E2E Proof       Working Code      Quality Report    Complete Docs         │
│                                                                              │
│  Phase 8          Phase 9                                                   │
│  ┌─────────┐      ┌─────────┐                                              │
│  │ DEPLOY  │  →   │ MAINTAIN│                                              │
│  │& RELEASE│      │& EVOLVE │                                              │
│  └────┬────┘      └─────────┘                                              │
│       │                                                                      │
│       ▼                                                                      │
│   Production                                                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.4 How to Use This Document

**For Traditional Development:**
1. Use checklists as quality gates
2. Use templates as documentation standards
3. Customize for your team's workflow

**For AI-Assisted Development:**
1. Complete each phase's checklist before proceeding
2. Feed completed templates to AI as context
3. Use atomic step patterns for AI prompts

**For Vibe Coding:**
1. Complete Phase 0-2 manually (critical human decisions)
2. Feed architecture docs to AI for Phase 3+
3. Maintain human-in-the-loop at phase boundaries

---

## 2. Phase 0: Project Inception

**Purpose:** Define the project's existence justification and high-level scope.  
**Duration:** 1-4 hours (small project) / 1-2 days (large project)  
**Output:** Project Charter Document  
**Human-in-the-Loop:** REQUIRED (strategic decisions)

### 2.1 Minimum Checklist

- [ ] **Problem Statement**: One-sentence description of the problem being solved
- [ ] **Value Proposition**: Why this solution matters (business/personal value)
- [ ] **Scope Boundaries**: What is explicitly IN scope and OUT of scope
- [ ] **Success Criteria**: 3-5 measurable outcomes that define "done"
- [ ] **Stakeholder Identification**: Who cares about this project
- [ ] **Initial Timeline Estimate**: Rough calendar estimate (weeks/months)
- [ ] **Resource Assessment**: Skills, tools, budget, and constraints available
- [ ] **Risk Identification**: Top 3-5 risks with initial mitigation ideas

### 2.2 Extended Checklist (Enterprise Projects)

- [ ] Market/Competitor Analysis completed
- [ ] Business case document created
- [ ] Budget approved
- [ ] Compliance requirements identified (legal, regulatory, security)
- [ ] Integration requirements mapped (existing systems)
- [ ] Team structure and roles defined
- [ ] Communication plan established
- [ ] Governance structure approved

### 2.3 Template: Project Charter

```markdown
# Project Charter: [Project Name]

**Version:** 1.0.0  
**Date:** [YYYY-MM-DD]  
**Status:** [Draft | Approved | Active]  
**Owner:** [Name/Role]

---

## 1. Problem Statement

[One paragraph: What problem does this solve? Who experiences this problem?]

**The Problem:** [One sentence]
**Impact:** [Quantify if possible - time lost, money wasted, errors caused]
**Current State:** [How is this handled today?]

---

## 2. Value Proposition

[Why should this project exist? What value does it create?]

**For [target user]** who [has this need],  
**[Project Name]** is a [category]  
**that** [key benefit].  
**Unlike** [current alternative],  
**our solution** [key differentiator].

---

## 3. Scope

### 3.1 In Scope
| ID | Feature/Capability | Priority | Description |
|----|-------------------|----------|-------------|
| S1 | | P0 (Must Have) | |
| S2 | | P1 (Should Have) | |
| S3 | | P2 (Nice to Have) | |

### 3.2 Out of Scope
- [Explicitly excluded item 1 - and why]
- [Explicitly excluded item 2 - and why]
- [Explicitly excluded item 3 - and why]

---

## 4. Success Criteria

| ID | Criterion | Measurement | Target | Deadline |
|----|-----------|-------------|--------|----------|
| SC1 | | | | |
| SC2 | | | | |
| SC3 | | | | |

---

## 5. Stakeholders

| Role | Name/Group | Interest Level | Influence | Communication Needs |
|------|------------|----------------|-----------|---------------------|
| Sponsor | | High | High | Weekly status |
| Primary User | | High | Medium | Feature demos |
| Technical Lead | | High | High | Daily standups |
| Operations | | Medium | Medium | Deployment plans |

---

## 6. Timeline & Resources

### 6.1 Timeline Estimate
| Phase | Duration | Start | End |
|-------|----------|-------|-----|
| Inception | | | |
| Architecture | | | |
| Implementation | | | |
| Testing | | | |
| Deployment | | | |

### 6.2 Resource Requirements
| Resource Type | Requirement | Availability |
|--------------|-------------|--------------|
| Personnel | | |
| Budget | | |
| Tools/Licenses | | |
| Infrastructure | | |

---

## 7. Constraints

| Constraint Type | Description | Impact |
|-----------------|-------------|--------|
| Technical | [e.g., Must run on Windows 11] | |
| Organizational | [e.g., Solo developer] | |
| Financial | [e.g., No cloud budget] | |
| Timeline | [e.g., Must launch by Q2] | |
| Regulatory | [e.g., GDPR compliance] | |

---

## 8. Risks

| ID | Risk | Likelihood | Impact | Mitigation Strategy |
|----|------|------------|--------|---------------------|
| R1 | | High/Med/Low | High/Med/Low | |
| R2 | | | | |
| R3 | | | | |

---

## 9. Approval

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Sponsor | | | |
| Technical Lead | | | |
| | | | |

---

**Document History:**
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | | | Initial creation |
```

### 2.4 Acceptance Criteria

Before proceeding to Phase 1, verify:
- [ ] Problem statement is specific and measurable
- [ ] At least one success criterion is quantifiable
- [ ] Scope boundaries are explicitly documented
- [ ] Constraints are known and documented
- [ ] Key stakeholders have reviewed (if applicable)
- [ ] Timeline is realistic given constraints
- [ ] Risks are identified with mitigation strategies

---

## 3. Phase 1: Architecture & Design

**Purpose:** Define WHAT the system does and HOW it's structured.  
**Duration:** 2-8 hours (small) / 1-2 weeks (large)  
**Output:** Architecture Requirements Document  
**Template Standard:** arc42 (simplified for AI consumption)  
**Human-in-the-Loop:** REQUIRED (structural decisions have long-term impact)

### 3.1 Minimum Checklist

**Goals & Context:**
- [ ] **Project Goal**: What is the software FOR? (one paragraph)
- [ ] **Key Features**: 3-7 core capabilities identified and prioritized
- [ ] **User Identification**: Who uses this system? (personas)
- [ ] **External Systems**: What does it integrate with? (APIs, databases, services)

**Quality & Constraints:**
- [ ] **Quality Attributes**: Non-functional requirements defined (performance, security, scalability)
- [ ] **Technical Constraints**: Platform, language, framework limitations documented
- [ ] **Organizational Constraints**: Team size, skills, timeline, budget documented

**Structure:**
- [ ] **Architecture Pattern**: Selected (Layered, Hexagonal, Microservices, Modular Monolith, DDMR)
- [ ] **Component Identification**: High-level modules/services identified
- [ ] **Component Responsibilities**: Each component has clear, single responsibility
- [ ] **Data Model Sketch**: Entities and relationships (rough ERD)
- [ ] **Integration Points**: How components communicate (sync/async, protocols)

### 3.2 Extended Checklist (Complex Projects)

- [ ] Context Diagram (C4 Level 1) completed
- [ ] Container Diagram (C4 Level 2) completed
- [ ] Component Diagram (C4 Level 3) completed
- [ ] Data Flow Diagrams created
- [ ] Security Architecture documented
- [ ] Deployment Architecture defined
- [ ] Disaster Recovery plan outlined
- [ ] Decision Log maintained with rationale

### 3.3 DDMR-Specific Checklist

If following DDMR (Distributed Development, Monolithic Runtime):

- [ ] **Boundary Principle**: Module boundaries defined with public/internal separation
- [ ] **Substitution Principle**: APIs designed for both in-process and remote calls
- [ ] **Traceability Principle**: Error propagation strategy includes module metadata
- [ ] **Dependency Coherence**: Version lock policy defined
- [ ] **Statelessness Principle**: Pure functions for business logic, side effects isolated

### 3.4 Template: Architecture Requirements Document (arc42 Lite)

```markdown
# Architecture Requirements Document
**Project:** [Name]  
**Version:** [X.Y.Z]  
**Date:** [YYYY-MM-DD]  
**Status:** [Draft | Review | Approved]

---

## 1. Introduction & Goals

### 1.1 Requirements Overview

**Project Goal:**  
[One paragraph: What is this software FOR? What problem does it solve?]

**Essential Features:**

| ID | Feature | Priority | Description | Success Metric |
|----|---------|----------|-------------|----------------|
| F1 | | P0 (Must) | | |
| F2 | | P0 (Must) | | |
| F3 | | P1 (Should) | | |
| F4 | | P2 (Could) | | |

### 1.2 Quality Goals

| Priority | Quality Attribute | Scenario | Measurement |
|----------|-------------------|----------|-------------|
| 1 | Performance | [e.g., Response < 200ms for 95th percentile] | [How measured] |
| 2 | Security | [e.g., All data encrypted at rest] | [How measured] |
| 3 | Maintainability | [e.g., New dev productive in 1 day] | [How measured] |
| 4 | Reliability | [e.g., 99.9% uptime] | [How measured] |

### 1.3 Stakeholders

| Role | Concerns | Expectations |
|------|----------|--------------|
| End User | Ease of use, reliability | Intuitive interface, fast response |
| Developer | Code quality, tooling | Clear architecture, good docs |
| Operator | Deployability, monitoring | Simple deployment, good observability |
| Security | Data protection, compliance | Encryption, audit trails |

---

## 2. Constraints

### 2.1 Technical Constraints

| Constraint | Rationale | Impact on Architecture |
|------------|-----------|------------------------|
| [e.g., Windows 11 Native] | [e.g., Target user environment] | [e.g., No Linux-only deps] |
| [e.g., Python 3.11+] | [e.g., Team expertise] | [e.g., Type hints required] |
| [e.g., Air-gapped possible] | [e.g., Enterprise requirement] | [e.g., No external API deps for core] |

### 2.2 Organizational Constraints

| Constraint | Rationale | Impact on Architecture |
|------------|-----------|------------------------|
| [e.g., Solo developer] | [e.g., Resource limitation] | [e.g., Simpler stack preferred] |
| [e.g., 3-month deadline] | [e.g., Business requirement] | [e.g., MVP scope only] |
| [e.g., No cloud budget] | [e.g., Cost constraint] | [e.g., Local-first design] |

### 2.3 Conventions

| Convention | Description |
|------------|-------------|
| Coding Standards | [Reference to standards doc] |
| Documentation | [e.g., Markdown, inline comments] |
| Testing | [e.g., 80% coverage minimum] |
| Version Control | [e.g., Git Flow, Trunk-based] |

---

## 3. Context & Scope

### 3.1 Business Context

**System Context Diagram:**
```
                    ┌─────────────────┐
                    │    [User A]     │
                    └────────┬────────┘
                             │
                             ▼
┌─────────────┐      ┌───────────────┐      ┌─────────────┐
│ [External   │ ──── │  [YOUR        │ ──── │ [External   │
│  System A]  │      │   SYSTEM]     │      │  System B]  │
└─────────────┘      └───────────────┘      └─────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   [Database]    │
                    └─────────────────┘
```

### 3.2 Technical Context

| External System | Protocol | Data Exchanged | Purpose |
|-----------------|----------|----------------|---------|
| [e.g., GitHub API] | REST/HTTPS | [e.g., Repos, PRs] | [e.g., Repository management] |
| [e.g., PostgreSQL] | TCP/5432 | [e.g., Governance data] | [e.g., Persistence] |
| [e.g., Redis] | TCP/6379 | [e.g., Cache data] | [e.g., Caching] |

---

## 4. Solution Strategy

### 4.1 Architecture Pattern

**Selected Pattern:** [e.g., Modular Monolith / DDMR]

**Rationale:**
- [Reason 1]
- [Reason 2]
- [Reason 3]

**Alternatives Considered:**
| Alternative | Why Not Selected |
|-------------|------------------|
| Microservices | [e.g., Operational complexity too high for team size] |
| Traditional Monolith | [e.g., No boundary enforcement, harder to evolve] |

### 4.2 Technology Decisions Summary

| Decision Area | Choice | Alternatives | Rationale |
|---------------|--------|--------------|-----------|
| Primary Language | | | |
| Framework | | | |
| Database | | | |
| Frontend | | | |
| CI/CD | | | |

*(Full rationale in Tech Stack Decision Document)*

### 4.3 Key Design Decisions

| ID | Decision | Rationale | Consequences |
|----|----------|-----------|--------------|
| D1 | | | |
| D2 | | | |
| D3 | | | |

---

## 5. Building Block View

### 5.1 Level 1: System Context

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           [SYSTEM NAME]                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                │
│   │ Component A │    │ Component B │    │ Component C │                │
│   │             │    │             │    │             │                │
│   │ [Purpose]   │ → │ [Purpose]   │ → │ [Purpose]   │                │
│   └─────────────┘    └─────────────┘    └─────────────┘                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Component Catalog

| Component | Responsibility | Dependencies | Public Interface |
|-----------|----------------|--------------|------------------|
| | | | |
| | | | |
| | | | |

### 5.3 Important Interfaces

| Interface | From | To | Protocol | Data |
|-----------|------|-----|----------|------|
| | | | | |

---

## 6. Runtime View

### 6.1 Key Scenarios

**Scenario 1: [Name]**
```
Actor → Component A → Component B → Database
         │              │              │
         │ 1. Request   │              │
         │──────────────>              │
         │              │ 2. Query     │
         │              │──────────────>
         │              │ 3. Data      │
         │              │<──────────────
         │ 4. Response  │              │
         │<──────────────              │
```

---

## 7. Deployment View

### 7.1 Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  DEVELOPMENT              CI/CD                PRODUCTION                │
│  ───────────              ─────                ──────────                │
│                                                                          │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────────────────┐ │
│  │ Local Dev   │ ──── │ GitHub      │ ──── │ [Deployment Target]     │ │
│  │ Environment │      │ Actions     │      │                         │ │
│  └─────────────┘      └─────────────┘      └─────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Deployment Modes

| Mode | Components | Use Case |
|------|------------|----------|
| Local Only | CLI + Core | Air-gapped development |
| CI Only | CLI + GitHub Action | Minimal deployment |
| Full Stack | All components | Enterprise deployment |

---

## 8. Cross-Cutting Concerns

### 8.1 Security Concept

| Concern | Approach |
|---------|----------|
| Authentication | [e.g., JWT, OAuth] |
| Authorization | [e.g., RBAC, Policy-based] |
| Data Encryption | [e.g., AES-256 at rest, TLS in transit] |
| Audit Logging | [e.g., All state changes logged] |

### 8.2 Error Handling Strategy

- **Domain Errors:** [e.g., Return Result types, no exceptions in domain]
- **Infrastructure Errors:** [e.g., Wrap in domain errors, include context]
- **Traceability:** [e.g., All errors include module origin, correlation ID]

### 8.3 Logging & Monitoring

| Aspect | Approach |
|--------|----------|
| Structured Logging | [e.g., JSON format, module context required] |
| Metrics | [e.g., Prometheus format] |
| Tracing | [e.g., OpenTelemetry] |
| Alerting | [e.g., Threshold-based, on-call rotation] |

---

## 9. Design Decisions (Architecture Decision Records)

### ADR-001: [Decision Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded]  
**Date:** [YYYY-MM-DD]  
**Context:** [What is the issue that we're seeing that is motivating this decision?]  
**Decision:** [What is the change that we're proposing and/or doing?]  
**Consequences:** [What becomes easier or more difficult to do because of this change?]

---

## 10. Quality Requirements

### 10.1 Quality Tree

```
Quality
├── Performance
│   ├── Response Time < 200ms (95th percentile)
│   └── Throughput > 1000 req/sec
├── Security
│   ├── Data encrypted at rest
│   └── Zero critical vulnerabilities
├── Maintainability
│   ├── New developer productive in 1 day
│   └── Test coverage > 80%
└── Reliability
    ├── 99.9% uptime
    └── Recovery time < 15 minutes
```

### 10.2 Quality Scenarios

| ID | Quality | Scenario | Measurement | Target |
|----|---------|----------|-------------|--------|
| Q1 | Performance | API request under normal load | Response time | < 200ms |
| Q2 | Maintainability | Add new governance rule | Implementation time | < 4 hours |
| Q3 | Reliability | Database failure | Recovery time | < 15 minutes |

---

## 11. Risks & Technical Debt

### 11.1 Identified Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|------------|--------|------------|
| R1 | | | | |
| R2 | | | | |

### 11.2 Known Technical Debt

| ID | Item | Reason | Plan to Address |
|----|------|--------|-----------------|
| TD1 | | | |

---

## 12. Glossary

| Term | Definition |
|------|------------|
| | |

---

**Document Status:** [Draft | Review | Approved]  
**Next Review:** [Date]  
**Owner:** [Name/Team]
```

### 3.5 Acceptance Criteria

Before proceeding to Phase 2:
- [ ] All key features are documented with priorities
- [ ] External system integrations are identified
- [ ] Quality attributes are defined with measurable scenarios
- [ ] Component boundaries are clear
- [ ] Technical and organizational constraints are documented
- [ ] Architecture pattern is selected with rationale
- [ ] At least 3 quality scenarios are defined

---

## 4. Phase 2: Technology Stack Decision

**Purpose:** Select and document technology choices with rationale.  
**Duration:** 1-4 hours (small) / 2-5 days (large)  
**Output:** Tech Stack Decision Document  
**Template Standard:** MADR (Markdown Any Decision Record)  
**Human-in-the-Loop:** REQUIRED (technology choices have long-term impact)

### 4.1 Minimum Checklist

**For Each Major Decision:**
- [ ] **Problem Statement**: What technology gap needs filling?
- [ ] **Constraints Review**: Constraints from Phase 1 considered
- [ ] **Options Identification**: At least 2-3 alternatives evaluated
- [ ] **Evaluation Criteria**: How options were compared (matrix)
- [ ] **Decision Recorded**: Chosen option with clear rationale
- [ ] **Consequences Documented**: Positive, negative, neutral implications
- [ ] **Rejected Options Noted**: Why alternatives weren't chosen

### 4.2 Decision Categories Checklist

**Core Decisions (REQUIRED):**
- [ ] Programming Language(s)
- [ ] Runtime/Platform
- [ ] Primary Framework(s)

**Data Decisions (if applicable):**
- [ ] Primary Database
- [ ] Caching Strategy
- [ ] File Storage
- [ ] Search (if needed)

**Infrastructure Decisions:**
- [ ] Deployment Target (local, cloud, hybrid)
- [ ] Containerization (Docker, none)
- [ ] Orchestration (K8s, none)
- [ ] CI/CD Platform

**Development Experience:**
- [ ] IDE/Editor
- [ ] Version Control Strategy
- [ ] Package Manager(s)
- [ ] Linting/Formatting Tools
- [ ] Testing Framework(s)

**AI/Integration (if AI-assisted):**
- [ ] AI Assistant Integration (Claude, Copilot, Continue.dev)
- [ ] MCP Server Strategy (if applicable)
- [ ] Rules/Governance Strategy

### 4.3 Template: MADR Decision Record

```markdown
# ADR-[NUMBER]: [Short Title of Decision]

**Date:** [YYYY-MM-DD]  
**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-XXX]  
**Deciders:** [Names or roles]  
**Technical Story:** [Link to issue/story if applicable]

---

## Context and Problem Statement

**Problem:** [One-sentence description of the problem]

**Context:**
[2-3 sentences providing background. Why is this decision needed now?]

**Constraints from Architecture:**
- [Constraint 1 from Phase 1]
- [Constraint 2 from Phase 1]
- [Constraint 3 from Phase 1]

---

## Decision Drivers

* [Driver 1: e.g., Team expertise in Python]
* [Driver 2: e.g., Must run in air-gapped environments]
* [Driver 3: e.g., Performance requirement of <200ms response]
* [Driver 4: e.g., Long-term maintainability]

---

## Considered Options

### Option 1: [Name]

**Description:** [1-2 sentences]

**Pros:**
- [Pro 1]
- [Pro 2]
- [Pro 3]

**Cons:**
- [Con 1]
- [Con 2]

**Fit with Constraints:**
| Constraint | Fit | Notes |
|------------|-----|-------|
| [Constraint 1] | ✅/⚠️/❌ | |
| [Constraint 2] | ✅/⚠️/❌ | |

---

### Option 2: [Name]

**Description:** [1-2 sentences]

**Pros:**
- [Pro 1]
- [Pro 2]

**Cons:**
- [Con 1]
- [Con 2]
- [Con 3]

**Fit with Constraints:**
| Constraint | Fit | Notes |
|------------|-----|-------|
| [Constraint 1] | ✅/⚠️/❌ | |
| [Constraint 2] | ✅/⚠️/❌ | |

---

### Option 3: [Name]

**Description:** [1-2 sentences]

**Pros:**
- [Pro 1]

**Cons:**
- [Con 1]
- [Con 2]

---

## Decision Outcome

**Chosen Option:** [Option X]

**Reasoning:**
[2-3 sentences explaining WHY this option was selected over others. Reference decision drivers.]

---

## Consequences

### Positive
- [Expected benefit 1]
- [Expected benefit 2]

### Negative
- [Known trade-off 1]
- [Known trade-off 2]

### Neutral
- [Other implication 1]

---

## Validation

**How will we validate this decision?**
- [ ] [Proof of concept by DATE]
- [ ] [Metric to track]
- [ ] [Review milestone]

---

## Links

- [Related ADR 1]
- [Architecture Requirements Document]
- [External documentation]
```

### 4.4 Example: Complete Tech Stack Decision Summary Table

```markdown
## Tech Stack Decision Summary

| Category | Decision | Alternatives Considered | Rationale | ADR |
|----------|----------|------------------------|-----------|-----|
| **Languages** | | | | |
| Primary Backend | Python 3.11 | Go, Rust, Node.js | Team expertise, prototyping speed, portability | ADR-001 |
| Performance-Critical | Rust | C++, Go | Type safety, WASM compilation, correctness | ADR-002 |
| API Server | Go | Node, Python | Single binary, concurrency, wazero for WASM | ADR-003 |
| Frontend | Elm | React, Vue | No runtime exceptions, type safety | ADR-004 |
| **Data** | | | | |
| Primary Database | PostgreSQL 16 | SQLite, MongoDB | Relational needs, concurrency, maturity | ADR-005 |
| Cache | Redis 7 | Memcached | Pub/sub, data structures, persistence | ADR-006 |
| **Infrastructure** | | | | |
| Containerization | Docker | Podman, None | Industry standard, team familiarity | ADR-007 |
| CI/CD | GitHub Actions | GitLab CI, Jenkins | Ecosystem fit, marketplace, cost | ADR-008 |
| **Development** | | | | |
| IDE | VS Code + Continue.dev | Cursor, JetBrains | MCP support, extensibility, cost | ADR-009 |
| Linting | Ruff (Python), Clippy (Rust) | Pylint, Flake8 | Speed, comprehensiveness | ADR-010 |
```

### 4.5 Acceptance Criteria

Before proceeding to Phase 3:
- [ ] All major technology decisions are documented
- [ ] Each decision has at least 2 alternatives considered
- [ ] Constraints from Phase 1 are explicitly addressed
- [ ] Consequences (positive and negative) are acknowledged
- [ ] No decision contradicts project constraints
- [ ] Tech stack decisions are traceable to architecture requirements
- [ ] Validation plan exists for risky decisions

---

## 5. Phase 3: Repository & Infrastructure Setup

**Purpose:** Establish the development environment and project structure.  
**Duration:** 1-4 hours (small) / 1-2 days (large)  
**Output:** Initialized repository with standard structure, working build  
**AI Delegation:** Moderate (structure is defined, execution can be AI-assisted)

### 5.1 Minimum Checklist

**Repository Setup:**
- [ ] **Git Repository Created**: Repository initialized (GitHub, GitLab, etc.)
- [ ] **Branch Strategy Defined**: Main/develop branches, protection rules
- [ ] **Directory Structure**: Standard folders established per blueprint
- [ ] **Ignore Files**: .gitignore configured for all languages in stack
- [ ] **Initial Commit**: Clean first commit with structure only

**Project Configuration:**
- [ ] **README Written**: Project overview, quick start, links to docs
- [ ] **License Added**: Appropriate license file
- [ ] **Dependency Management**: Package managers configured for all languages
- [ ] **Build System**: Makefile or equivalent with standard targets

**Quality Infrastructure:**
- [ ] **Linting Configured**: All linters configured and passing
- [ ] **Formatting Configured**: Auto-formatters set up
- [ ] **Pre-commit Hooks**: Basic hooks installed and working
- [ ] **Editor Config**: .editorconfig for consistency

### 5.2 Extended Checklist

**CI/CD Pipeline (Basic):**
- [ ] Build workflow configured
- [ ] Test workflow configured
- [ ] Lint workflow configured
- [ ] Branch protection requires CI pass

**Documentation Infrastructure:**
- [ ] docs/ folder structure created
- [ ] Architecture docs moved to repo
- [ ] ADR folder created

**GitHub/GitLab Configuration:**
- [ ] Issue templates created
- [ ] Pull request template created
- [ ] CODEOWNERS file (if team)
- [ ] Labels configured
- [ ] Milestones created

**AI Development Setup (if applicable):**
- [ ] .continue/ folder structure
- [ ] Rules files in .continue/rules/
- [ ] MCP server configuration
- [ ] Claude Project instructions (if using Claude)

### 5.3 Template: Repository Structure

**Universal Polyglot Monorepo Structure:**

```
[project-name]/
│
├── README.md                           # Project overview, quick start
├── LICENSE                             # License file
├── CONTRIBUTING.md                     # Contribution guidelines
├── CHANGELOG.md                        # Version history
├── Makefile                            # Root build commands
├── .gitignore                          # Git ignore patterns
├── .editorconfig                       # Editor consistency
│
├── .github/                            # GitHub Configuration
│   ├── workflows/                      # CI/CD pipelines
│   │   ├── ci.yml                      # Main CI pipeline
│   │   ├── release.yml                 # Release pipeline
│   │   └── security.yml                # Security scanning
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── pull_request_template.md
│   └── CODEOWNERS
│
├── .continue/                          # AI Assistant Configuration
│   ├── config.yaml                     # Continue.dev config
│   ├── rules/                          # Governance rules
│   │   ├── 00-project-context.md
│   │   ├── 01-coding-standards.md
│   │   └── 02-architecture-rules.md
│   └── mcpServers/                     # MCP server configs
│
├── docs/                               # 📚 DOCUMENTATION
│   ├── architecture/
│   │   ├── 01-ARCHITECTURE_REQUIREMENTS.md
│   │   ├── 02-TECH_STACK_DECISION.md
│   │   ├── 03-REPOSITORY_STRUCTURE.md
│   │   └── 04-IMPLEMENTATION_ROADMAP.md
│   ├── adr/                            # Architecture Decision Records
│   │   ├── template.md
│   │   └── 001-initial-architecture.md
│   ├── guides/                         # User/developer guides
│   └── api/                            # API documentation
│
├── apps/                               # 🚀 DEPLOYABLE APPLICATIONS
│   ├── [app-name-1]/                   # Each app has own structure
│   │   ├── [language-specific files]
│   │   ├── src/
│   │   └── tests/
│   └── [app-name-2]/
│
├── packages/                           # 📦 SHARED LIBRARIES
│   ├── [package-name-1]/
│   │   ├── [language-specific files]
│   │   ├── src/
│   │   │   ├── public/                 # Public API
│   │   │   └── internal/               # Private implementation
│   │   └── tests/
│   └── [package-name-2]/
│
├── tools/                              # 🔧 DEVELOPMENT TOOLS
│   ├── scripts/                        # Utility scripts
│   ├── pre-commit/                     # Pre-commit hook configs
│   └── templates/                      # Code generation templates
│
├── infrastructure/                     # 🏗️ DEPLOYMENT
│   ├── docker/                         # Dockerfiles
│   ├── kubernetes/                     # K8s manifests (if applicable)
│   └── terraform/                      # Infrastructure as code
│
├── config/                             # ⚙️ CONFIGURATION
│   ├── development.yaml                # Dev environment config
│   ├── production.yaml                 # Prod environment config
│   └── [project].schema.json           # Config validation schema
│
└── examples/                           # 📖 EXAMPLES
    ├── basic-usage/
    └── advanced-usage/
```

### 5.4 Template: Root Makefile

```makefile
.PHONY: help setup test build lint clean

# Default target
help:
	@echo "Available targets:"
	@echo "  setup    - Install all dependencies"
	@echo "  test     - Run all tests"
	@echo "  build    - Build all components"
	@echo "  lint     - Run all linters"
	@echo "  clean    - Clean build artifacts"
	@echo "  dev      - Start development environment"

# ============================================
# SETUP
# ============================================
setup: setup-python setup-node setup-rust setup-go
	@echo "✅ Development environment ready"

setup-python:
	@echo "📦 Setting up Python..."
	cd packages/[python-package] && pip install -e ".[dev]"

setup-node:
	@echo "📦 Setting up Node..."
	npm install

setup-rust:
	@echo "📦 Setting up Rust..."
	cd packages/[rust-package] && cargo build

setup-go:
	@echo "📦 Setting up Go..."
	cd apps/[go-app] && go mod download

# ============================================
# TEST
# ============================================
test: test-python test-node test-rust test-go
	@echo "✅ All tests passed"

test-python:
	cd packages/[python-package] && pytest

test-node:
	npm test

test-rust:
	cd packages/[rust-package] && cargo test

test-go:
	cd apps/[go-app] && go test ./...

# ============================================
# BUILD
# ============================================
build: build-python build-node build-rust build-go
	@echo "✅ All components built"

# ============================================
# LINT
# ============================================
lint: lint-python lint-node lint-rust lint-go
	@echo "✅ All linting passed"

lint-python:
	cd packages/[python-package] && ruff check .

lint-node:
	npm run lint

lint-rust:
	cd packages/[rust-package] && cargo clippy

lint-go:
	cd apps/[go-app] && golangci-lint run

# ============================================
# CLEAN
# ============================================
clean:
	find . -type d -name "__pycache__" -exec rm -rf {} +
	find . -type d -name "node_modules" -exec rm -rf {} +
	find . -type d -name "target" -exec rm -rf {} +
	find . -type d -name "dist" -exec rm -rf {} +

# ============================================
# DEV
# ============================================
dev:
	@echo "Starting development environment..."
	# Add dev server commands
```

### 5.5 Template: README.md

```markdown
# [Project Name]

[![CI](https://github.com/[org]/[repo]/actions/workflows/ci.yml/badge.svg)](https://github.com/[org]/[repo]/actions/workflows/ci.yml)
[![License](https://img.shields.io/badge/license-[LICENSE]-blue.svg)](LICENSE)

[One-line description of what this project does]

## Overview

[2-3 sentences explaining the project purpose and key features]

## Quick Start

### Prerequisites

- [Prerequisite 1 with version]
- [Prerequisite 2 with version]
- [Prerequisite 3 with version]

### Installation

```bash
# Clone the repository
git clone https://github.com/[org]/[repo].git
cd [repo]

# Install dependencies
make setup

# Verify installation
make test
```

### Basic Usage

```bash
# Example command
[command example]
```

## Features

- ✅ [Feature 1]
- ✅ [Feature 2]
- ✅ [Feature 3]
- 🚧 [Planned Feature 1]

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture/01-ARCHITECTURE_REQUIREMENTS.md) | System architecture and design |
| [Tech Stack](docs/architecture/02-TECH_STACK_DECISION.md) | Technology choices and rationale |
| [API Reference](docs/api/) | API documentation |
| [Contributing](CONTRIBUTING.md) | How to contribute |

## Project Structure

```
[project-name]/
├── apps/           # Deployable applications
├── packages/       # Shared libraries
├── docs/           # Documentation
├── tools/          # Development tools
└── infrastructure/ # Deployment configs
```

## Development

### Setup

```bash
make setup
```

### Running Tests

```bash
make test
```

### Linting

```bash
make lint
```

### Building

```bash
make build
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

[License Type] - See [LICENSE](LICENSE) for details.

## Acknowledgments

- [Acknowledgment 1]
- [Acknowledgment 2]
```

### 5.6 Acceptance Criteria

Before proceeding to Phase 4:
- [ ] Repository is accessible to all team members
- [ ] `make setup` completes without errors
- [ ] `make test` runs (even if no tests yet)
- [ ] `make lint` passes
- [ ] README contains working quick start instructions
- [ ] All architecture docs are in docs/architecture/
- [ ] .gitignore covers all languages in stack
- [ ] Pre-commit hooks are installed
- [ ] CI pipeline runs on push/PR

---

## 6. Phase 4: Walking Skeleton Implementation

**Purpose:** Prove the architecture works end-to-end with minimal functionality.  
**Duration:** 1-3 days  
**Output:** A deployable system that executes "Hello World" through all layers  
**Origin:** Alistair Cockburn (Agile Manifesto co-author)  
**AI Delegation:** Moderate-High (structure is defined, AI can implement)

### 6.1 Core Concept

> "A Walking Skeleton is a tiny implementation of the system that performs a small end-to-end function. It need not use the final architecture, but it should link together the main architectural components."

**Why Walking Skeleton Matters:**

| Without Walking Skeleton | With Walking Skeleton |
|--------------------------|----------------------|
| Integration issues found late | Integration verified first |
| "Works on my machine" problems | Deployment proven early |
| Architectural assumptions untested | Architecture validated |
| Team blocked on dependencies | All layers unblocked |

**Critical for AI/Vibe Coding:** AI excels at writing individual functions but struggles with system integration. A Walking Skeleton forces integration issues to be solved FIRST, before AI generates volumes of code that may not connect properly.

### 6.2 Minimum Checklist

**Layer Connectivity:**
- [ ] **Data Layer**: Database/storage connection works
- [ ] **Backend/API Layer**: Server starts and responds to requests
- [ ] **Frontend Layer** (if applicable): UI loads and renders
- [ ] **Integration Layer**: External system connections work

**Data Flow Verification:**
- [ ] Data can be written to storage
- [ ] Data can be read from storage
- [ ] Request flows from UI → API → Storage → API → UI
- [ ] Error states propagate correctly

**Operational Verification:**
- [ ] Logs output to correct destination
- [ ] Configuration loads from environment
- [ ] Health check endpoint responds
- [ ] Build produces deployable artifact
- [ ] Deployment completes (even locally)

### 6.3 Extended Checklist

- [ ] Authentication flow works (even with stub)
- [ ] Authorization middleware active
- [ ] Metrics/telemetry collecting
- [ ] Structured logging with correlation IDs
- [ ] Graceful shutdown implemented
- [ ] Docker container builds and runs
- [ ] Basic CI pipeline passes

### 6.4 Template: Walking Skeleton Verification

```markdown
# Walking Skeleton Verification Report

**Project:** [Name]  
**Date:** [YYYY-MM-DD]  
**Verified By:** [Name]

---

## 1. Layer Verification

### 1.1 Data Layer

| Test | Command | Expected | Actual | Status |
|------|---------|----------|--------|--------|
| Database starts | `docker-compose up db` | Container running | | ⬜ |
| Connection works | `make test-db-connection` | "Connected" | | ⬜ |
| Write succeeds | `make test-db-write` | Row inserted | | ⬜ |
| Read succeeds | `make test-db-read` | Row returned | | ⬜ |
| Migrations run | `make migrate` | "Migrations complete" | | ⬜ |

**Evidence:**
```
[Paste command output here]
```

### 1.2 Backend/API Layer

| Test | Command | Expected | Actual | Status |
|------|---------|----------|--------|--------|
| Server starts | `make run-api` | Listening on :8080 | | ⬜ |
| Health check | `curl localhost:8080/health` | 200 OK | | ⬜ |
| API endpoint | `curl localhost:8080/api/v1/[resource]` | 200 + JSON | | ⬜ |
| Error handling | `curl localhost:8080/api/v1/invalid` | 404 + Error JSON | | ⬜ |

**Evidence:**
```
[Paste command output here]
```

### 1.3 Frontend Layer (if applicable)

| Test | Expected | Actual | Status |
|------|----------|--------|--------|
| App compiles | No errors | | ⬜ |
| App loads in browser | Page renders | | ⬜ |
| API call succeeds | Data displayed | | ⬜ |
| Error state renders | Error message shown | | ⬜ |

**Evidence:**
[Screenshot or console output]

### 1.4 External Integrations

| Integration | Test | Expected | Actual | Status |
|-------------|------|----------|--------|--------|
| [External API 1] | Connection test | Connected | | ⬜ |
| [External API 2] | Auth test | Token received | | ⬜ |

---

## 2. End-to-End Flow Verification

### 2.1 Primary Flow: [Name of main use case]

```
Step 1: [User action]
  └── Verified: ⬜

Step 2: [System action]
  └── Verified: ⬜

Step 3: [Expected outcome]
  └── Verified: ⬜
```

**Flow Evidence:**
```
[Request/response logs showing full flow]
```

### 2.2 Error Flow: [Name of error case]

```
Step 1: [Trigger error condition]
  └── Verified: ⬜

Step 2: [Error propagates]
  └── Verified: ⬜

Step 3: [User sees error]
  └── Verified: ⬜
```

---

## 3. Deployment Verification

| Test | Command | Expected | Actual | Status |
|------|---------|----------|--------|--------|
| Build completes | `make build` | No errors | | ⬜ |
| Docker image builds | `docker build .` | Image created | | ⬜ |
| Container runs | `docker run [image]` | App starts | | ⬜ |
| Health check in container | `curl localhost:8080/health` | 200 OK | | ⬜ |

---

## 4. Observability Verification

| Aspect | Test | Expected | Status |
|--------|------|----------|--------|
| Logging | Make request, check logs | Request logged with correlation ID | ⬜ |
| Metrics | Check metrics endpoint | Metrics exposed | ⬜ |
| Errors | Trigger error, check logs | Error logged with stack trace | ⬜ |

---

## 5. Summary

| Category | Passed | Failed | Blocked |
|----------|--------|--------|---------|
| Data Layer | /4 | | |
| API Layer | /4 | | |
| Frontend | /4 | | |
| Integration | /2 | | |
| E2E Flow | /2 | | |
| Deployment | /4 | | |
| Observability | /3 | | |
| **TOTAL** | /23 | | |

## 6. Issues Found

| ID | Issue | Severity | Resolution |
|----|-------|----------|------------|
| 1 | | | |

## 7. Sign-Off

| Role | Name | Date | Approved |
|------|------|------|----------|
| Developer | | | ⬜ |
| Tech Lead | | | ⬜ |

---

**Skeleton Status:** ⬜ WALKING / ⬜ CRAWLING / ⬜ BROKEN
```

### 6.5 Acceptance Criteria

Before proceeding to Phase 5:
- [ ] All layers are connected and communicating
- [ ] A request can flow through the entire system
- [ ] Data persists correctly
- [ ] Errors propagate with useful information
- [ ] The skeleton is deployable (at least locally)
- [ ] Logs show complete request lifecycle
- [ ] No hardcoded credentials or paths in code
- [ ] CI pipeline passes with skeleton tests

---

## 7. Phase 5: Core Feature Implementation

**Purpose:** Build actual functionality using atomic, incremental steps.  
**Duration:** Variable (bulk of project time - typically 40-60% of total)  
**Output:** Working features meeting requirements  
**Method:** Atomic Step Development with Human-in-the-Loop  
**AI Delegation:** High (with human review at checkpoints)

### 7.1 Implementation Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ATOMIC IMPLEMENTATION CYCLE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   For each Feature:                                                         │
│                                                                              │
│   ┌─────────────┐                                                           │
│   │  1. PLAN    │  Human: Review requirements, define acceptance criteria   │
│   │   (Human)   │  Output: Feature Implementation Plan                      │
│   └──────┬──────┘                                                           │
│          │                                                                   │
│          ▼                                                                   │
│   ┌─────────────┐                                                           │
│   │ 2. DECOMPOSE│  Human/AI: Break into atomic steps (1-2 hour chunks)     │
│   │ (Human+AI)  │  Output: Ordered task list with dependencies             │
│   └──────┬──────┘                                                           │
│          │                                                                   │
│          ▼                                                                   │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │                    For each Atomic Step:                         │       │
│   │                                                                  │       │
│   │   ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐         │       │
│   │   │ CONTEXT│ →  │  CODE  │ →  │  TEST  │ →  │ REVIEW │ → COMMIT│       │
│   │   │(Human) │    │  (AI)  │    │(AI+Run)│    │(Human) │         │       │
│   │   └────────┘    └────────┘    └────────┘    └────────┘         │       │
│   │                                                                  │       │
│   │   If Review FAIL → Return to CODE with feedback                 │       │
│   │   If Review PASS → Commit and next step                         │       │
│   │                                                                  │       │
│   └─────────────────────────────────────────────────────────────────┘       │
│          │                                                                   │
│          ▼                                                                   │
│   ┌─────────────┐                                                           │
│   │ 3. INTEGRATE│  Human: Review feature end-to-end                        │
│   │   (Human)   │  Action: Merge to main, update docs                      │
│   └─────────────┘                                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Feature Implementation Checklist

For EACH feature, complete:

**Planning (Human):**
- [ ] **Requirement Review**: Re-read feature requirements from Phase 1
- [ ] **Acceptance Criteria**: Define 3-5 testable criteria for "done"
- [ ] **Dependency Check**: Identify prerequisites and blockers
- [ ] **Risk Assessment**: Identify implementation risks

**Decomposition (Human + AI):**
- [ ] **Task Decomposition**: Break into atomic steps (1-2 hour max each)
- [ ] **Dependency Ordering**: Order tasks by dependencies
- [ ] **Interface Design**: Define public API/contract first
- [ ] **Test Strategy**: Identify test approach for each step

**Implementation (AI with Human Review):**
- [ ] **Atomic Step Execution**: Complete each step with test
- [ ] **Code Review**: Human reviews each step before commit
- [ ] **Documentation Updates**: Update relevant docs as you go

**Integration (Human):**
- [ ] **Integration Test**: Test feature end-to-end
- [ ] **Acceptance Test**: Verify all acceptance criteria
- [ ] **Performance Check**: Verify no performance regression
- [ ] **Merge**: Integrate to main branch

### 7.3 Atomic Step Criteria

An atomic step must satisfy ALL of the following:

| Criterion | Description | Example |
|-----------|-------------|---------|
| **Single Purpose** | Does exactly one thing | "Add validation to User.email field" |
| **Completable** | Can finish in 1-2 hours | Not "Implement entire user module" |
| **Testable** | Has clear test criteria | "Email format validates correctly" |
| **Isolated** | Minimal dependencies on uncommitted code | Uses existing interfaces |
| **Reversible** | Can be reverted cleanly | Single commit, no mixed concerns |

### 7.4 Template: Feature Implementation Plan

```markdown
# Feature Implementation Plan: [Feature Name]

**Feature ID:** [F-XXX]  
**Priority:** [P0/P1/P2]  
**Estimated Duration:** [X days/hours]  
**Assigned To:** [Name]

---

## 1. Requirement Summary

**Source:** [Link to requirement in Architecture doc]

**Description:**
[Copy requirement description]

---

## 2. Acceptance Criteria

| ID | Criterion | Test Method | Automated |
|----|-----------|-------------|-----------|
| AC1 | [Testable criterion] | [How to verify] | Yes/No |
| AC2 | [Testable criterion] | [How to verify] | Yes/No |
| AC3 | [Testable criterion] | [How to verify] | Yes/No |

---

## 3. Dependencies

### 3.1 Prerequisites (must be complete before starting)
- [ ] [Prerequisite 1]
- [ ] [Prerequisite 2]

### 3.2 External Dependencies
- [ ] [External system/API]

---

## 4. Task Decomposition

### Step 1: [Name] 
**Estimate:** [X hours]  
**Type:** [Interface | Implementation | Test | Integration]

**Input:** [What this step needs]  
**Output:** [What this step produces]  
**Tests:**
- [ ] [Test case 1]
- [ ] [Test case 2]

**AI Prompt Template:**
```
Context: [Brief context]
Task: [Specific task]
Constraints: [Coding standards, patterns to follow]
Expected Output: [What to produce]
```

---

### Step 2: [Name]
**Estimate:** [X hours]  
**Type:** [Interface | Implementation | Test | Integration]

**Input:** [What this step needs - may reference Step 1 output]  
**Output:** [What this step produces]  
**Tests:**
- [ ] [Test case 1]

---

### Step 3: [Name]
[Continue pattern...]

---

## 5. Interface Design

```[language]
// Public interface for this feature
[Interface definition]
```

---

## 6. Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| | | | |

---

## 7. Progress Tracking

| Step | Status | Started | Completed | Notes |
|------|--------|---------|-----------|-------|
| 1 | ⬜ Not Started | | | |
| 2 | ⬜ Not Started | | | |
| 3 | ⬜ Not Started | | | |

**Status Legend:** ⬜ Not Started | 🟡 In Progress | ✅ Complete | ❌ Blocked

---

## 8. Review Checklist

Before marking feature complete:
- [ ] All acceptance criteria pass
- [ ] Unit tests written and passing
- [ ] Integration tests written and passing
- [ ] Code reviewed and approved
- [ ] Documentation updated
- [ ] No new linting errors
- [ ] Performance acceptable
```

### 7.5 AI/Vibe Coding Prompt Template

For each atomic step, use this structure:

```markdown
## Context

**Project:** [Project Name]
**Feature:** [Feature being implemented]
**Current State:** [What exists so far]

**Reference Documents:**
- Architecture: [file path or key points]
- Tech Stack: [relevant decisions]
- Related Code: [file paths]

---

## Atomic Step

**Step Number:** [X of Y]
**Task:** [Specific, single task - one sentence]
**Type:** [Interface | Implementation | Test | Refactor]

**Input Available:**
- [Input 1]
- [Input 2]

**Expected Output:**
- [Output 1 - be specific]
- [Output 2]

---

## Constraints

**Coding Standards:**
- Max function length: 15 lines
- Max cyclomatic complexity: 5
- Max parameters: 3
- Pure functions for business logic

**Patterns to Follow:**
- [Pattern 1 from codebase]
- [Pattern 2]

**Do NOT:**
- [Anti-pattern to avoid]
- [Thing not to do]

---

## Request

[Specific instruction for this atomic step]

Include:
- Implementation code
- Unit tests
- Brief explanation of approach
```

### 7.6 Acceptance Criteria

Before marking a feature complete:
- [ ] All acceptance criteria from plan verified
- [ ] Unit test coverage ≥ 80% for new code
- [ ] Integration tests pass
- [ ] No new linting errors or warnings
- [ ] Documentation updated
- [ ] Code review approved
- [ ] Performance within acceptable bounds
- [ ] Merged to main branch
- [ ] CI pipeline passes

---

## 8. Phase 6: Testing & Quality Assurance

**Purpose:** Verify the system works correctly and meets quality standards.  
**Duration:** Ongoing (parallel with Phase 5)  
**Output:** Passing test suite, quality reports, confidence in release  
**AI Delegation:** High for test generation, Human for test strategy

### 8.1 Test Pyramid Strategy

```
                    ┌─────────┐
                    │   E2E   │  ← Few (5-10), slow, high confidence
                   ─┼─────────┼─   Cover critical user journeys
                  ┌─┴─────────┴─┐
                  │ Integration │  ← Some (20-50), medium speed
                 ─┼─────────────┼─  Component boundaries
                ┌─┴─────────────┴─┐
                │   Unit Tests    │  ← Many (100s), fast, isolated
                └─────────────────┘   Every public function
```

### 8.2 Minimum Checklist

**Unit Testing:**
- [ ] Every public function has tests
- [ ] Edge cases covered (null, empty, boundary values)
- [ ] Error paths tested
- [ ] Pure functions have property-based tests (where applicable)

**Integration Testing:**
- [ ] API endpoints tested with real database
- [ ] External service integrations tested (can use mocks)
- [ ] Authentication/authorization flows tested
- [ ] Error propagation tested

**End-to-End Testing:**
- [ ] Critical user journeys automated
- [ ] Happy path scenarios covered
- [ ] Key error scenarios covered

**Quality Metrics:**
- [ ] Line coverage ≥ 80%
- [ ] Branch coverage ≥ 70%
- [ ] No critical security vulnerabilities
- [ ] All linting rules pass
- [ ] Type checking passes (if applicable)

### 8.3 Extended Checklist

**Advanced Testing:**
- [ ] Property-based tests for complex logic
- [ ] Mutation testing score ≥ 80%
- [ ] Performance tests (latency, throughput)
- [ ] Load tests (if applicable)
- [ ] Chaos testing (if applicable)

**Security Testing:**
- [ ] SAST (Static Application Security Testing) clean
- [ ] Dependency vulnerability scan clean
- [ ] OWASP Top 10 verified

**Accessibility Testing (if UI):**
- [ ] WCAG 2.1 AA compliance
- [ ] Screen reader testing
- [ ] Keyboard navigation

### 8.4 Template: Quality Gates Configuration

```markdown
# Quality Gates Configuration

**Project:** [Name]  
**Version:** [X.Y.Z]  
**Last Updated:** [Date]

---

## Gate 1: Pre-Commit (Local, <30 seconds)

**Trigger:** Every commit attempt

| Check | Tool | Threshold | Blocking |
|-------|------|-----------|----------|
| Formatting | [prettier/black/rustfmt] | No changes needed | Yes |
| Linting | [eslint/ruff/clippy] | 0 errors | Yes |
| Type Check | [tsc/mypy] | 0 errors | Yes |
| Unit Tests | [jest/pytest] | All pass | Yes |
| Secrets Scan | [gitleaks] | 0 findings | Yes |

**Configuration:**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: lint
        name: Lint
        entry: make lint
        language: system
        pass_filenames: false
      - id: test
        name: Unit Tests
        entry: make test-unit
        language: system
        pass_filenames: false
```

---

## Gate 2: Pull Request (CI, <10 minutes)

**Trigger:** PR created or updated

| Check | Tool | Threshold | Blocking |
|-------|------|-----------|----------|
| All Gate 1 checks | - | Pass | Yes |
| Integration Tests | [pytest/jest] | All pass | Yes |
| Coverage | [coverage/nyc] | ≥ 80% lines | Yes |
| Coverage Delta | - | No decrease | Yes |
| Security Scan | [snyk/dependabot] | 0 critical/high | Yes |
| Build | [make/cargo/go] | Success | Yes |
| Docs Build | [mkdocs/docusaurus] | Success | No |

---

## Gate 3: Pre-Merge (Human Review)

**Trigger:** Before merge approval

| Check | Reviewer | Criteria |
|-------|----------|----------|
| Code Review | Peer | Approved by 1+ reviewer |
| Architecture Review | Tech Lead | If touching core components |
| Acceptance Criteria | Product | All criteria verified |
| Documentation | Any | Relevant docs updated |

---

## Gate 4: Pre-Release (CI, <30 minutes)

**Trigger:** Release branch/tag

| Check | Tool | Threshold | Blocking |
|-------|------|-----------|----------|
| All Gate 2 checks | - | Pass | Yes |
| E2E Tests | [playwright/cypress] | All pass | Yes |
| Performance Tests | [k6/locust] | Latency < 200ms p95 | Yes |
| Full Security Scan | [OWASP ZAP] | 0 high findings | Yes |
| Changelog | Manual | Updated | Yes |
| Version | semver | Bumped correctly | Yes |

---

## Gate 5: Post-Deploy (Monitoring)

**Trigger:** After production deployment

| Check | Tool | Threshold | Action |
|-------|------|-----------|--------|
| Health Check | curl/synthetic | 200 OK | Auto-rollback |
| Error Rate | [datadog/grafana] | < 1% | Alert |
| Latency | [datadog/grafana] | < 500ms p99 | Alert |
| Smoke Tests | [automated] | All pass | Auto-rollback |

---

## Exceptions Process

To bypass a quality gate:
1. Create issue documenting why
2. Get Tech Lead approval
3. Add `skip-gate: [gate-name]` label
4. Create follow-up issue to address
```

### 8.5 Test Coverage Report Template

```markdown
# Test Coverage Report

**Date:** [YYYY-MM-DD]  
**Commit:** [SHA]  
**Branch:** [branch-name]

---

## Summary

| Metric | Current | Target | Status |
|--------|---------|--------|--------|
| Line Coverage | XX% | 80% | ✅/❌ |
| Branch Coverage | XX% | 70% | ✅/❌ |
| Function Coverage | XX% | 90% | ✅/❌ |
| Mutation Score | XX% | 80% | ✅/❌ |

---

## Coverage by Component

| Component | Lines | Branches | Functions | Notes |
|-----------|-------|----------|-----------|-------|
| [component-1] | XX% | XX% | XX% | |
| [component-2] | XX% | XX% | XX% | |
| [component-3] | XX% | XX% | XX% | Needs attention |

---

## Uncovered Critical Paths

| File | Lines | Risk | Action |
|------|-------|------|--------|
| [file.ext] | 45-67 | High | Add tests by [date] |

---

## Test Suite Health

| Suite | Tests | Passing | Failing | Skipped | Duration |
|-------|-------|---------|---------|---------|----------|
| Unit | | | | | |
| Integration | | | | | |
| E2E | | | | | |
```

### 8.6 Acceptance Criteria

Before Phase 6 is considered complete for a release:
- [ ] All quality gates pass
- [ ] Test coverage meets thresholds
- [ ] No critical or high severity bugs open
- [ ] Performance meets requirements
- [ ] Security scan clean
- [ ] Test report generated and reviewed

---

## 9. Phase 7: Documentation

**Purpose:** Create documentation for users, developers, and operators.  
**Duration:** Ongoing (parallel with implementation)  
**Output:** Complete documentation set  
**AI Delegation:** High (documentation is AI's strength)

### 9.1 Documentation Types Checklist

**User Documentation:**
- [ ] Installation/Setup Guide
- [ ] Getting Started Tutorial (< 10 minutes to first success)
- [ ] Feature Documentation (one page per major feature)
- [ ] Configuration Reference
- [ ] FAQ
- [ ] Troubleshooting Guide

**Developer Documentation:**
- [ ] Architecture Overview
- [ ] API Reference (auto-generated where possible)
- [ ] Contributing Guide
- [ ] Development Setup Guide
- [ ] Code Style Guide
- [ ] Testing Guide

**Operator Documentation:**
- [ ] Deployment Guide
- [ ] Configuration Reference
- [ ] Monitoring/Alerting Guide
- [ ] Backup/Recovery Procedures
- [ ] Runbook (incident response)
- [ ] Upgrade Guide

**Project Documentation:**
- [ ] README (project overview)
- [ ] CHANGELOG (version history)
- [ ] CONTRIBUTING (how to contribute)
- [ ] LICENSE
- [ ] SECURITY (vulnerability reporting)

### 9.2 Documentation Quality Checklist

For EACH document, verify:

- [ ] **Title**: Clearly states purpose
- [ ] **Audience**: Identified (user, developer, operator)
- [ ] **Prerequisites**: Listed at top
- [ ] **Structure**: Logical flow, scannable headings
- [ ] **Steps**: Numbered, testable, reproducible
- [ ] **Examples**: Included for complex concepts
- [ ] **Errors**: Common errors documented with solutions
- [ ] **Links**: All internal/external links work
- [ ] **Currency**: Last updated date is recent
- [ ] **Tested**: Someone followed the docs successfully

### 9.3 Template: User Guide Structure

```markdown
# [Feature/Product] User Guide

**Last Updated:** [YYYY-MM-DD]  
**Applies To:** Version [X.Y.Z]+  
**Audience:** [End users / Administrators / Developers]

---

## Overview

[2-3 sentences: What is this? What can you do with it?]

---

## Prerequisites

Before you begin, ensure you have:

- [ ] [Prerequisite 1]
- [ ] [Prerequisite 2]
- [ ] [Prerequisite 3]

---

## Quick Start

Get up and running in 5 minutes:

### Step 1: [Action]

[Brief instruction]

```bash
[command]
```

**Expected Result:** [What user should see]

### Step 2: [Action]

[Brief instruction]

### Step 3: [Verify]

[How to confirm it worked]

---

## Features

### [Feature 1 Name]

**What it does:** [One sentence]

**How to use it:**

1. [Step 1]
2. [Step 2]
3. [Step 3]

**Example:**

```
[Example usage]
```

**Tips:**
- [Tip 1]
- [Tip 2]

---

### [Feature 2 Name]

[Same structure...]

---

## Configuration

### [Configuration Option 1]

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `option_name` | string | `"default"` | What it controls |

**Example:**

```yaml
option_name: "custom_value"
```

---

## Troubleshooting

### Problem: [Description]

**Symptoms:** [What user sees]

**Cause:** [Why it happens]

**Solution:**

1. [Step 1]
2. [Step 2]

---

### Problem: [Description]

[Same structure...]

---

## FAQ

### Q: [Common question]?

A: [Answer]

### Q: [Common question]?

A: [Answer]

---

## Getting Help

- **Documentation:** [Link]
- **Issues:** [Link to issue tracker]
- **Community:** [Link to forum/Discord/etc.]

---

## See Also

- [Related Guide 1]
- [Related Guide 2]
```

### 9.4 Acceptance Criteria

Before documentation is considered complete:
- [ ] New user can install and run following docs only
- [ ] New developer can contribute following docs only
- [ ] All public APIs are documented
- [ ] All configuration options are documented
- [ ] No broken links
- [ ] Docs build without errors
- [ ] Someone other than author has followed the docs

---

## 10. Phase 8: Deployment & Release

**Purpose:** Deliver the system to users safely and repeatably.  
**Duration:** Hours to days depending on complexity  
**Output:** Deployed, accessible, monitored system  
**Human-in-the-Loop:** REQUIRED (release is a critical decision)

### 10.1 Pre-Release Checklist

**Code Readiness:**
- [ ] All quality gates pass
- [ ] Feature branch merged to main/release
- [ ] No known critical bugs
- [ ] Performance benchmarks acceptable

**Documentation Readiness:**
- [ ] Version number updated in code
- [ ] CHANGELOG updated with all changes
- [ ] Release notes written
- [ ] Migration guide (if breaking changes)
- [ ] User documentation current

**Operational Readiness:**
- [ ] Deployment scripts tested
- [ ] Rollback procedure documented and tested
- [ ] Monitoring dashboards ready
- [ ] Alerting configured
- [ ] On-call schedule confirmed

**Communication:**
- [ ] Stakeholders notified of release timeline
- [ ] Support team briefed on changes
- [ ] Status page ready for updates

### 10.2 Deployment Checklist

**Pre-Deployment:**
- [ ] Environment verified (correct target)
- [ ] Configuration reviewed
- [ ] Secrets/credentials in place
- [ ] Database backups taken
- [ ] Traffic drain initiated (if applicable)

**Deployment Execution:**
- [ ] Deployment command executed
- [ ] Deployment logs monitored
- [ ] Health checks passing
- [ ] Smoke tests passing

**Post-Deployment:**
- [ ] Full functionality verified
- [ ] Metrics baseline compared
- [ ] Error rates normal
- [ ] Performance acceptable
- [ ] User-facing tests pass

### 10.3 Post-Release Checklist

**Immediate (within 1 hour):**
- [ ] Monitoring confirms health
- [ ] No elevated error rates
- [ ] User reports triaged

**Short-term (within 24 hours):**
- [ ] Release notes published
- [ ] Documentation site updated
- [ ] Stakeholders notified of success
- [ ] Retrospective scheduled (if significant release)

**Follow-up (within 1 week):**
- [ ] User feedback collected
- [ ] Metrics reviewed
- [ ] Issues triaged
- [ ] Next release planning begun

### 10.4 Template: Release Notes

```markdown
# Release Notes: v[X.Y.Z]

**Release Date:** [YYYY-MM-DD]  
**Release Type:** [Major | Minor | Patch | Hotfix]

---

## Highlights

[1-3 sentence summary of the most important changes in this release]

---

## New Features

### [Feature Name]

[2-3 sentences describing the feature and its value]

**How to use:**
```
[Brief example]
```

**Documentation:** [Link to full docs]

---

## Improvements

- **[Area]:** [Description of improvement] ([#issue])
- **[Area]:** [Description of improvement] ([#issue])

---

## Bug Fixes

- **[Bug title]:** [Brief description of what was fixed] ([#issue])
- **[Bug title]:** [Brief description of what was fixed] ([#issue])

---

## Breaking Changes

### [Change Description]

**What changed:** [Description]

**Migration steps:**

1. [Step 1]
2. [Step 2]

**Before:**
```
[Old way]
```

**After:**
```
[New way]
```

---

## Deprecations

- `[deprecated_thing]` - Will be removed in v[X+1].0.0. Use `[new_thing]` instead.

---

## Known Issues

- [Issue description] - Workaround: [workaround] ([#issue])

---

## Upgrade Instructions

### From v[Previous Version]

```bash
# Step 1: Backup
[backup command]

# Step 2: Upgrade
[upgrade command]

# Step 3: Verify
[verification command]
```

---

## Contributors

Thanks to everyone who contributed to this release:

- @[username]
- @[username]

---

## Full Changelog

[Link to compare view or full changelog]
```

### 10.5 Acceptance Criteria

Before release is considered complete:
- [ ] All pre-release checklist items verified
- [ ] Deployment successful
- [ ] Health checks passing
- [ ] No elevated error rates
- [ ] Release notes published
- [ ] Stakeholders notified

---

## 11. Phase 9: Maintenance & Evolution

**Purpose:** Keep the system healthy and evolving.  
**Duration:** Ongoing  
**Output:** Maintained, improving system  
**Pattern:** Continuous improvement cycles

### 11.1 Ongoing Maintenance Schedule

**Daily:**
- [ ] Review error logs for anomalies
- [ ] Check monitoring dashboards
- [ ] Triage new issues

**Weekly:**
- [ ] Review and merge dependabot PRs (patch versions)
- [ ] Check security advisories
- [ ] Review performance metrics
- [ ] Address high-priority bug fixes

**Monthly:**
- [ ] Update dependencies (minor versions)
- [ ] Review and address technical debt
- [ ] Update documentation as needed
- [ ] Review and close stale issues
- [ ] Security scan and address findings

**Quarterly:**
- [ ] Major dependency updates
- [ ] Architecture review
- [ ] Performance audit
- [ ] Documentation comprehensive review
- [ ] Disaster recovery test

**Annually:**
- [ ] Full security audit
- [ ] Architecture modernization review
- [ ] Team retrospective on development practices
- [ ] Update project charter and roadmap

### 11.2 Evolution Workflow

When adding new features after initial release:

1. **Return to Phase 1**: Update Architecture document with new requirements
2. **Phase 2 if needed**: Update Tech Stack for new technologies
3. **Skip Phase 3**: Repository already exists
4. **Phase 4 if needed**: Extend Walking Skeleton for new integrations
5. **Phase 5**: Implement following atomic step pattern
6. **Phase 6-8**: Standard quality and release process

### 11.3 Technical Debt Management

```markdown
# Technical Debt Register

## Priority: High (Address within 1 sprint)

| ID | Description | Impact | Effort | Owner | Target Date |
|----|-------------|--------|--------|-------|-------------|
| TD-001 | | | | | |

## Priority: Medium (Address within 1 quarter)

| ID | Description | Impact | Effort | Owner | Target Date |
|----|-------------|--------|--------|-------|-------------|
| TD-010 | | | | | |

## Priority: Low (Address when convenient)

| ID | Description | Impact | Effort | Owner | Target Date |
|----|-------------|--------|--------|-------|-------------|
| TD-100 | | | | | |

## Resolved

| ID | Description | Resolution Date | Resolution |
|----|-------------|-----------------|------------|
| | | | |
```

---

## Appendix A: Templates

### A.1 Quick Reference: All Templates

| Phase | Template | Purpose | Location |
|-------|----------|---------|----------|
| 0 | Project Charter | Define project existence | Section 2.3 |
| 1 | Architecture Requirements (arc42) | Define system structure | Section 3.4 |
| 2 | MADR Decision Record | Document technology choices | Section 4.3 |
| 3 | Repository Structure | Organize codebase | Section 5.3 |
| 3 | Makefile | Build automation | Section 5.4 |
| 3 | README.md | Project overview | Section 5.5 |
| 4 | Walking Skeleton Verification | Prove architecture works | Section 6.4 |
| 5 | Feature Implementation Plan | Plan feature work | Section 7.4 |
| 5 | AI/Vibe Coding Prompt | Structure AI requests | Section 7.5 |
| 6 | Quality Gates | Define quality thresholds | Section 8.4 |
| 6 | Test Coverage Report | Track test health | Section 8.5 |
| 7 | User Guide Structure | Document for users | Section 9.3 |
| 8 | Release Notes | Communicate changes | Section 10.4 |

### A.2 Commit Message Convention

```
<type>(<scope>): <subject>

<body>

<footer>

────────────────────────────────────────

TYPES:
  feat     New feature
  fix      Bug fix
  docs     Documentation only
  style    Formatting (no code change)
  refactor Code change (no feature/fix)
  test     Adding tests
  chore    Maintenance tasks

SCOPE: Component or module affected

SUBJECT: Imperative mood, no period, max 50 chars

BODY: Explain what and why (not how), wrap at 72 chars

FOOTER: Reference issues, note breaking changes

────────────────────────────────────────

EXAMPLES:

feat(auth): add OAuth2 login flow

Implement OAuth2 authorization code flow with PKCE.
Users can now sign in with Google and GitHub.

Closes #123

────────────────────────────────────────

fix(api): handle null response from external service

The payment service occasionally returns null instead of
an error object. This caused unhandled exceptions.

Now we check for null and return appropriate error.

Fixes #456

────────────────────────────────────────

BREAKING CHANGE: change API response format

The /users endpoint now returns { data: [...] } instead
of the raw array. Update clients accordingly.
```

### A.3 Pull Request Template

```markdown
## Description

[Brief description of changes]

## Type of Change

- [ ] 🐛 Bug fix (non-breaking change fixing an issue)
- [ ] ✨ New feature (non-breaking change adding functionality)
- [ ] 💥 Breaking change (fix or feature causing existing functionality to change)
- [ ] 📝 Documentation update
- [ ] 🔧 Configuration change
- [ ] ♻️ Refactor (no functional change)

## Related Issues

Closes #[issue number]
Related to #[issue number]

## Changes Made

- [Change 1]
- [Change 2]
- [Change 3]

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

**Test instructions:**
1. [Step 1]
2. [Step 2]

## Checklist

- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings introduced
- [ ] Tests pass locally
- [ ] Changes are backward compatible (or breaking changes documented)

## Screenshots (if applicable)

[Add screenshots for UI changes]

## Notes for Reviewers

[Any context reviewers should know]
```

---

## Appendix B: Atomic Coding Standards Quick Reference

### B.1 Function-Level Limits

| Metric | Atomic Limit | Tool | Rationale |
|--------|--------------|------|-----------|
| **Cyclomatic Complexity** | ≤ 5 | ESLint, SonarQube, PMD | Fits in working memory; each path testable |
| **Lines of Code (per function)** | ≤ 15 | Checkstyle, RuboCop | Visual atomicity; no scrolling needed |
| **Indentation Depth** | ≤ 1 | ESLint max-depth | Forces method extraction; SLAP compliance |
| **Parameters** | ≤ 3 | Checkstyle, ESLint | Encourages value objects; reduces coupling |
| **Class Instance Variables** | ≤ 2-3 | PMD TooManyFields | Prevents God classes; enforces cohesion |

### B.2 Function Purity Requirements

**Pure Functions (REQUIRED for business logic):**
- Depend ONLY on inputs
- Produce ONLY outputs (return value)
- NO side effects (no I/O, no mutation, no globals)
- Same input → Same output (deterministic)

**Pattern: Functional Core / Imperative Shell**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FUNCTIONAL CORE / IMPERATIVE SHELL                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   IMPERATIVE SHELL (Thin)                                               │
│   ─────────────────────                                                 │
│   │ 1. Fetch data (impure)     │                                       │
│   │    - Read from database    │                                       │
│   │    - Call external API     │                                       │
│   │    - Read configuration    │                                       │
│   │                            │                                       │
│   │ 2. Call Functional Core ───┼──────────────┐                        │
│   │                            │              │                        │
│   │ 3. Save results (impure)   │              ▼                        │
│   │    - Write to database     │   ┌─────────────────────┐             │
│   │    - Send notifications    │   │  FUNCTIONAL CORE    │             │
│   │    - Update cache          │   │  (Pure Functions)   │             │
│   │                            │   │                     │             │
│   └────────────────────────────┘   │  • Calculate        │             │
│                                    │  • Transform        │             │
│                                    │  • Validate         │             │
│                                    │  • Decide           │             │
│                                    │                     │             │
│                                    │  Input → Output     │             │
│                                    │  (No side effects)  │             │
│                                    └─────────────────────┘             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### B.3 Code Smells to Avoid

| Smell | Symptom | Atomic Fix |
|-------|---------|------------|
| **Long Method** | >15 lines | Extract methods |
| **Long Parameter List** | >3 params | Introduce Parameter Object |
| **Nested Conditionals** | >1 indent | Extract methods, use guard clauses |
| **Feature Envy** | Method uses another class's data | Move method to that class |
| **Data Clumps** | Same params appear together | Create value object |
| **Primitive Obsession** | Primitives instead of objects | Wrap in value types |
| **God Class** | Class does too much | Split by responsibility |
| **Shotgun Surgery** | One change = many files | Consolidate related code |

### B.4 Enforcement Tools by Language

| Language | Complexity | Size | Purity | Architecture |
|----------|------------|------|--------|--------------|
| **JavaScript/TypeScript** | ESLint complexity | ESLint max-lines-per-function | eslint-plugin-functional | dependency-cruiser |
| **Python** | Ruff/Pylint | Ruff/Pylint | Custom rules | import-linter |
| **Rust** | Clippy | Clippy | Language enforced | cargo-modules |
| **Go** | golangci-lint | golangci-lint | go-critic | go-arch-lint |
| **Java** | PMD, SonarQube | Checkstyle | ArchUnit | ArchUnit |
| **Elm** | elm-review | elm-review | Language enforced | elm-review |

---

## Appendix C: AI/Vibe Coding Best Practices

### C.1 Context Engineering Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Just-in-Time Loading** | Load only context needed for current step | Progressive disclosure; don't pre-load everything |
| **Phase Isolation** | Each phase gets its own clean context | Separate Claude Projects per phase |
| **Artifact Hand-Off** | Completed artifacts become input for next phase | Explicit file passing between phases |
| **Explicit References** | Name files explicitly in prompts | "Refer to ArchSpec.md Section 3.2" |
| **Constraint Repetition** | Repeat key constraints in each prompt | Include coding standards in every implementation prompt |

### C.2 Prompt Patterns for Each Phase

**Architecture Brainstorm (Phase 1):**
```
Based on the requirements in [Requirements.md], brainstorm at least 
two architecture approaches for [system].

For each approach:
1. Describe the key components
2. List pros (minimum 3)
3. List cons (minimum 2)
4. Identify risks

Conclude with a recommendation and justification.
Format as markdown.
```

**Tech Stack Decision (Phase 2):**
```
I need to decide on [technology category] for [project].

Context:
- Constraints: [list from architecture doc]
- Team: [expertise]
- Timeline: [duration]

Evaluate these options:
1. [Option A]
2. [Option B]
3. [Option C]

For each, provide:
- Brief description
- Pros/cons
- Fit with constraints (table)

Recommend one with clear reasoning.
```

**Code Implementation (Phase 5):**
```
Context:
- Project: [name]
- Architecture: [key points or file reference]
- Current file: [path]
- Related code: [file paths]

Task: Implement [specific function/component]

Requirements:
- [Requirement 1]
- [Requirement 2]

Constraints:
- Max 15 lines per function
- Max complexity 5
- Pure functions for business logic
- Follow patterns in [existing file]

Provide:
1. Implementation code
2. Unit tests
3. Brief explanation of approach
```

**Documentation (Phase 7):**
```
Generate documentation for [component/feature].

Source materials:
- Code: [file paths]
- Architecture: [file reference]

Audience: [users / developers / operators]

Required sections:
- Overview (what it does)
- Prerequisites
- Usage with examples
- Configuration options
- Common errors and solutions

Style: Clear, concise, actionable. Use code examples.
```

### C.3 Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| **Context Overload** | AI loses focus, quality degrades | Use progressive disclosure; only load what's needed |
| **Vague Prompts** | Unpredictable, inconsistent output | Be specific; provide examples; define constraints |
| **Skipping Validation** | Errors compound across steps | Human review after each atomic step |
| **One Giant Prompt** | Can't debug failures; no checkpoints | Break into atomic steps; commit after each |
| **Ignoring Errors** | Technical debt accumulates | Fix errors immediately before proceeding |
| **No Context Reset** | Degraded performance in long sessions | Start fresh context for each phase/feature |
| **Trusting Blindly** | AI makes mistakes, especially integration | Always review; always test; always verify |

### C.4 Human-in-the-Loop Checkpoints

**Always require human review:**

| Checkpoint | Why | What to Check |
|------------|-----|---------------|
| **After Architecture Decisions** | Long-term impact | Feasibility, constraints, alternatives |
| **After Tech Stack Decisions** | Long-term impact | Team fit, maintenance burden |
| **Before Merging to Main** | Quality gate | Tests pass, code quality, no regressions |
| **After Security-Related Changes** | Safety critical | No vulnerabilities introduced |
| **When AI Expresses Uncertainty** | AI knows its limits | Verify approach manually |
| **At Phase Boundaries** | Major transitions | Artifacts complete, ready for next phase |
| **Before External Integrations** | External dependencies | API contracts, error handling |

### C.5 Continue.dev / MCP Integration Patterns

**Recommended .continue/ Structure:**

```
.continue/
├── config.yaml                 # Main configuration
├── rules/                      # Governance rules (loaded in order)
│   ├── 00-project-context.md   # Project overview, always loaded
│   ├── 01-coding-standards.md  # Atomic coding rules
│   ├── 02-architecture.md      # Architecture constraints
│   ├── 03-testing.md           # Testing requirements
│   └── 04-security.md          # Security guidelines
└── mcpServers/                 # MCP tool configurations
    ├── filesystem.yaml         # File access
    ├── github.yaml             # GitHub integration
    └── custom.yaml             # Project-specific tools
```

**Rule File Pattern:**
```markdown
---
name: coding-standards
description: Enforce atomic coding standards
globs: ["**/*.py", "**/*.ts", "**/*.go"]
---

# Coding Standards

## Function Rules
- Maximum 15 lines per function
- Maximum cyclomatic complexity: 5
- Maximum 3 parameters
- All business logic must be pure functions

## Naming Conventions
- Functions: verb_noun (snake_case for Python, camelCase for TS)
- Classes: PascalCase
- Constants: UPPER_SNAKE_CASE

## Architecture Rules
- No importing from internal/ outside the module
- All public APIs must have types
- All errors must include context

[Continue with specific rules...]
```

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2025-12-29 | Claude AI + Human | Initial creation |
| 2.0.0 | 2025-12-30 | Claude AI + Human | Comprehensive expansion with project integration |

---

## Sources & Acknowledgments

This document synthesizes best practices from:

- **arc42** - Software Architecture Documentation Template
- **MADR** - Markdown Any Decision Records
- **Walking Skeleton** - Alistair Cockburn
- **Multi-Phase AI-Assisted Development Framework** - Project Knowledge Base
- **DDMR Governance Framework** - Project Knowledge Base
- **Atomic Coding Standards** - Object Calisthenics, Clean Code
- **Continue.dev Documentation** - MCP Integration Patterns
- **C4 Model** - Simon Brown (Software Architecture Diagrams)

---

*This is a living document. Update it as your project evolves and as you learn what works best for your team.*
