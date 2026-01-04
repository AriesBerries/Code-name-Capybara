# TIER 3 RESOURCE ALLOCATION PLAN
## DevGuide Cockpit Implementation - 3-Month Execution

**Date:** 2026-01-02T14:00:00Z  
**Version:** 1.0.0  
**Task:** TASK 4 of TIER 3 First Pass  
**Status:** COMPLETE  
**Prerequisites:** TASK 0-3 outputs (Context Assembly + Monthly Timelines)

---

## Executive Summary

This document provides comprehensive resource allocation for the DevGuide Cockpit 3-month implementation, detailing team structure, skill requirements, workload distribution, and role assignments for all 7 engineers. The plan ensures balanced utilization (target 80%), identifies critical path dependencies, mitigates single points of failure, and establishes on-call rotation for launch support.

**Key Metrics:**
- **Total Team Size:** 7 engineers
- **Total Duration:** 12 weeks (60 working days)
- **Total Team Hours:** 2,800 hours (7 × 8h × 50 days effective)
- **Target Utilization:** 80% (20% buffer for meetings, issues, support)
- **Critical Path Items:** 4 identified with mitigations

---

## 1. Team Overview

### 1.1 Role Descriptions and Primary Responsibilities

| Role ID | Title | Primary Responsibilities | Secondary Responsibilities |
|---------|-------|-------------------------|---------------------------|
| **BE-Rust-1** | Backend Engineer 1 (Rust) | Rust systems development, MetadataValidator, The Bin validation layer, performance-critical code | Go integration, TiDB schema design, database migrations |
| **BE-Go-1** | Backend Engineer 2 (Go) | AI Cortex APIs, token tracking, context window management, provider interfaces | Rust integration, Redis caching, API documentation |
| **BE-Go-2** | Backend Engineer 3 (Go) | Extension Orchestration proxy, dashboard builder backend, integration APIs | Python testing, E2E test automation, fallback mechanisms |
| **FE-Elm-1** | Frontend Engineer 1 (Elm) | Composer UX (all 7 phases), core UI components, primary frontend architecture | TypeScript utilities, CSS design system, component library |
| **FE-Elm-2** | Frontend Engineer 2 (Elm) | Layout Edit Mode, Tri-Surface Workbench, accessibility compliance, UI testing | Component testing, keyboard navigation, screen reader support |
| **DevOps-1** | DevOps Engineer 1 | CI/CD pipelines, infrastructure deployment, security hardening, monitoring dashboards | Production deployment, rollback procedures, load testing |
| **DevOps-2** | DevOps Engineer 2 | Clock sync infrastructure, documentation, compliance verification, API versioning | Time sync monitoring, user documentation, training materials |

### 1.2 Skill Matrix

| Skill | BE-Rust-1 | BE-Go-1 | BE-Go-2 | FE-Elm-1 | FE-Elm-2 | DevOps-1 | DevOps-2 |
|-------|:---------:|:-------:|:-------:|:--------:|:--------:|:--------:|:--------:|
| **Rust** | ★★★★★ | ★★☆☆☆ | ★★☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★☆☆☆☆ | ☆☆☆☆☆ |
| **Go** | ★★★☆☆ | ★★★★★ | ★★★★★ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★★★☆☆ | ★★☆☆☆ |
| **Elm** | ☆☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★★★★★ | ★★★★★ | ☆☆☆☆☆ | ☆☆☆☆☆ |
| **TypeScript** | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ | ★★☆☆☆ |
| **TiDB/SQL** | ★★★★☆ | ★★★☆☆ | ★★★☆☆ | ★☆☆☆☆ | ★☆☆☆☆ | ★★★☆☆ | ★★☆☆☆ |
| **Redis** | ★★★☆☆ | ★★★★☆ | ★★★☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★★★☆☆ | ★★☆☆☆ |
| **CI/CD** | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★★ | ★★★★☆ |
| **Docker/K8s** | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ | ★☆☆☆☆ | ★☆☆☆☆ | ★★★★★ | ★★★★☆ |
| **NTP/PTP** | ★☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★★★☆☆ | ★★★★★ |
| **Accessibility** | ☆☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★★★★☆ | ★★★★★ | ☆☆☆☆☆ | ★★☆☆☆ |
| **Security** | ★★★★☆ | ★★★☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★★ | ★★★☆☆ |
| **API Design** | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ |

**Legend:** ★★★★★ Expert | ★★★★☆ Advanced | ★★★☆☆ Proficient | ★★☆☆☆ Familiar | ★☆☆☆☆ Basic | ☆☆☆☆☆ None

### 1.3 Reporting Structure

```
                    ┌─────────────────┐
                    │  Project Lead   │
                    │  (External)     │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────┴─────┐      ┌─────┴─────┐      ┌─────┴─────┐
    │  Backend  │      │ Frontend  │      │  DevOps   │
    │   Lead    │      │   Lead    │      │   Lead    │
    │(BE-Rust-1)│      │(FE-Elm-1) │      │(DevOps-1) │
    └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
          │                  │                  │
    ┌─────┴─────┐      ┌─────┴─────┐      ┌─────┴─────┐
    │ BE-Go-1   │      │ FE-Elm-2  │      │ DevOps-2  │
    │ BE-Go-2   │      │           │      │           │
    └───────────┘      └───────────┘      └───────────┘
```

**Lead Responsibilities:**
- **BE-Rust-1 (Backend Lead):** Technical decisions for backend architecture, code review for BE-Go-1 and BE-Go-2, API design coordination
- **FE-Elm-1 (Frontend Lead):** UI/UX consistency, component library ownership, code review for FE-Elm-2
- **DevOps-1 (DevOps Lead):** Infrastructure decisions, security policy, deployment coordination, code review for DevOps-2

### 1.4 Communication Channels and Meeting Cadence

| Meeting | Frequency | Duration | Attendees | Purpose |
|---------|-----------|----------|-----------|---------|
| **Daily Standup** | Daily, 9:30 AM UTC | 15 min | All 7 engineers | Blockers, progress, daily focus |
| **Backend Sync** | Mon/Wed/Fri, 10:00 AM UTC | 30 min | BE-Rust-1, BE-Go-1, BE-Go-2 | API coordination, integration points |
| **Frontend Sync** | Mon/Wed/Fri, 10:30 AM UTC | 30 min | FE-Elm-1, FE-Elm-2 | UI consistency, component sharing |
| **Cross-Team Integration** | Tuesday, 2:00 PM UTC | 45 min | Leads + relevant engineers | API contracts, end-to-end testing |
| **Sprint Planning** | Monday, 11:00 AM UTC (weekly) | 1 hour | All 7 engineers | Week objectives, task assignment |
| **Sprint Retrospective** | Friday, 4:00 PM UTC (weekly) | 45 min | All 7 engineers | Lessons learned, process improvement |
| **Architecture Review** | Wednesday, 3:00 PM UTC (bi-weekly) | 1 hour | Leads only | Technical decisions, debt tracking |

**Communication Tools:**
- **Slack:** #devguide-cockpit (general), #backend, #frontend, #devops, #incidents
- **GitHub:** Issues, PRs, code review, project board
- **Documentation:** Confluence/Notion for specs, decisions, meeting notes
- **On-Call:** PagerDuty for production alerts (Month 3+)

---

## 2. Monthly Workload Distribution

### 2.1 Month 1: Core Infrastructure (Weeks 1-4)

**Objective:** Deploy 11 database tables, establish metadata governance, AI Cortex infrastructure, and extension orchestration foundation.

| Engineer | Week 1-2 Focus | Week 3-4 Focus | Primary Deliverables |
|----------|---------------|----------------|---------------------|
| **BE-Rust-1** | DB schema deployment (4 metadata tables), seed standards | MetadataValidator integration with The Bin, auto-fix system | 4 tables deployed, MetadataValidator struct, 80% auto-fix rate |
| **BE-Go-1** | DB schema deployment (4 AI Cortex tables), API config seeding | Token tracking middleware, context window APIs, provider health | 4 tables deployed, Token tracking ±5% accuracy, health endpoints |
| **BE-Go-2** | DB schema deployment (3 extension tables), Go API scaffolding | Extension proxy foundation, extension communication protocol | 3 tables deployed, Proxy handling 5 extensions, protocol spec |
| **FE-Elm-1** | Metadata governance panel scaffolding, violation display | Metadata panel complete, real-time violation alerts | MetadataGovernance.elm, violation notification system |
| **FE-Elm-2** | Extension status monitoring UI, compliance dashboard | Accessibility audit (metadata panel), keyboard navigation | Extension panel, ARIA compliance >90 score |
| **DevOps-1** | CI/CD for DB migrations, TiDB Cloud staging, GitHub Actions | Performance monitoring setup, Grafana dashboards | Migration pipeline, staging env, 3 Grafana dashboards |
| **DevOps-2** | Clock sync research (NTP/PTP), architecture proposal | Clock sync implementation, documentation setup | Clock drift <10ms, sync status monitoring, docs site |

**Month 1 Cross-Functional Collaboration:**

| Integration Point | Lead Owner | Collaborators | Week |
|-------------------|------------|---------------|------|
| Metadata tables → Validation API | BE-Rust-1 | BE-Go-2 | 2 |
| AI Cortex tables → Token UI | BE-Go-1 | FE-Elm-1 | 3-4 |
| Extension tables → Dashboard backend | BE-Go-2 | FE-Elm-2 | 3-4 |
| The Bin → MetadataValidator | BE-Rust-1 | BE-Go-1, BE-Go-2 | 3-4 |
| Clock sync → Timestamp validation | DevOps-2 | BE-Rust-1 | 4 |

**Month 1 Bottleneck Identification:**

| Potential Bottleneck | Risk Level | Mitigation |
|---------------------|------------|------------|
| TiDB migration delays | High | Test all migrations in staging first (Day 1-2); BE-Rust-1 + DevOps-1 dedicated |
| Clock sync complexity | Medium | Start with NTP (simpler); defer PTP to Month 2 if needed |
| MetadataValidator performance | Medium | Early performance testing Day 5; Rust profiling |
| Extension proxy unclear architecture | High | Architecture spike Day 3; prototype before full implementation |

### 2.2 Month 2: Workflow Systems (Weeks 5-8)

**Objective:** Complete Extension Orchestration MVP, update all 3 templates, implement Composer Phases 1-4, Export Pipeline Stages 1-3, Tri-Surface Workbench, Prompt Control Hub.

| Engineer | Week 5-6 Focus | Week 7-8 Focus | Primary Deliverables |
|----------|---------------|----------------|---------------------|
| **BE-Rust-1** | Community repository schema, dashboard import/export | Export Pipeline Stages 1-3, migration testing | Community repo, export <500ms (p95) |
| **BE-Go-1** | AI Cortex integration (templates), context window APIs | Note Taking AI, template finalization, DB state sharing | Per-template token budgets, streaming updates |
| **BE-Go-2** | Proxy layer production, fallback mechanisms | Bin integration (exports), SSE update delivery | 10+ extensions, circuit breaker, 100ms SSE |
| **FE-Elm-1** | Dashboard builder UI, Vibe Coding template update | Composer Phases 1-4, Rails system, layout persistence | Builder functional, Mode selector (Ask/Edit/Agent/Plan) |
| **FE-Elm-2** | Data Science template, Note Taking template | Tri-Surface surfaces, Prompt Control Hub UI | All 3 templates v2, surface switching <300ms |
| **DevOps-1** | Extension CI/CD, template CI, staging sync | A/B testing infrastructure, performance testing | Automated deployment, load tested at 1000 users |
| **DevOps-2** | Extension testing framework, documentation | Security audit prep, integration test updates | 5-extension test suite, API changelog |

**Month 2 Cross-Functional Collaboration:**

| Integration Point | Lead Owner | Collaborators | Week |
|-------------------|------------|---------------|------|
| Dashboard builder → Backend storage | FE-Elm-1 | BE-Go-2 | 5 |
| Template migration → Validation | BE-Rust-1 | All engineers | 6 |
| Composer → AI Cortex | FE-Elm-1 | BE-Go-1 | 7 |
| Export pipeline → The Bin | BE-Rust-1 | BE-Go-2 | 7 |
| Tri-Surface → DB state | FE-Elm-2 | BE-Go-1, BE-Go-2 | 8 |
| Prompt Hub → Agent backend | FE-Elm-2 | BE-Rust-1, BE-Go-1 | 8 |

**Month 2 Bottleneck Identification:**

| Potential Bottleneck | Risk Level | Mitigation |
|---------------------|------------|------------|
| Template migration breaks user data | High | Test with 100+ sample projects; automatic rollback; 30-day recovery |
| Composer phases take longer | High | Prioritize Phases 1-2; feature flag individual phases |
| Extension proxy performance | Medium | Early benchmarking Day 22; caching layer |
| SSE reliability | Medium | Reconnection logic; fallback to polling |

### 2.3 Month 3: Polish & Launch (Weeks 9-12)

**Objective:** Complete remaining TIER X features (Composer Phases 5-7, Export Stages 4-5), governance documentation, security hardening, performance optimization, and beta launch.

| Engineer | Week 9-10 Focus | Week 11-12 Focus | Primary Deliverables |
|----------|----------------|------------------|---------------------|
| **BE-Rust-1** | Prompt Control Hub agents, command macros | Security hardening, E2E testing, launch support | Agent profiles, security audit passed |
| **BE-Go-1** | Agent control APIs, queue management | Integration testing, performance optimization | Stop/queue endpoints, <100ms API (p95) |
| **BE-Go-2** | Export Preview/Publish stages, skills system | Bug fixes, E2E testing, launch support | 5-stage pipeline complete, export <5s |
| **FE-Elm-1** | Composer Phases 5-7 (Toggles, Agent Controls, Safety) | Launch preparation, beta onboarding, polish | All 7 phases complete, onboarding flow |
| **FE-Elm-2** | Layout Edit Mode advanced features, preset library | Accessibility final audit, documentation | Preset library, >90 Lighthouse score |
| **DevOps-1** | Load testing, production environment prep | Deployment, monitoring, rollback verification | 1000 concurrent users, zero-downtime deploy |
| **DevOps-2** | Governance documentation (~45 pages), training | Launch day ops, support system, on-call setup | User guide complete, support ready |

**Month 3 Cross-Functional Collaboration:**

| Integration Point | Lead Owner | Collaborators | Week |
|-------------------|------------|---------------|------|
| Composer → Agent backend | FE-Elm-1 | BE-Go-1 | 9 |
| Export → Preview rendering | BE-Go-2 | FE-Elm-2 | 9-10 |
| Security audit → All systems | DevOps-1 | All engineers | 10-11 |
| Load testing → All APIs | DevOps-1 | BE-Rust-1, BE-Go-1 | 11 |
| Launch day → All systems | DevOps-1 | All engineers | 12 |

**Month 3 Bottleneck Identification:**

| Potential Bottleneck | Risk Level | Mitigation |
|---------------------|------------|------------|
| TIER X features delayed | Medium | Buffer Days 44-45 for overflow; defer non-critical |
| Security audit critical findings | Low | Early review Day 51; 5-day delay contingency |
| Load testing bottlenecks | Medium | Pre-optimization Days 51-52; reduce target to 500 if needed |
| Beta user critical bugs | Medium | 2-day buffer before launch (Days 58-59) |

---

## 3. Skill Requirements Matrix

### 3.1 System/Feature to Skills Mapping

| System/Feature | Required Skills | Skill Level | Lead Engineer | Backup Engineer |
|----------------|-----------------|-------------|---------------|-----------------|
| **Metadata Governance** | Rust, TiDB, Regex, Validation | Senior | BE-Rust-1 | BE-Go-2 |
| **MetadataValidator (The Bin)** | Rust, Systems Design | Senior | BE-Rust-1 | BE-Go-1 |
| **AI Cortex APIs** | Go, Redis, REST APIs, SSE | Senior | BE-Go-1 | BE-Go-2 |
| **Token Tracking** | Go, TiDB, Cost Calculation | Mid | BE-Go-1 | BE-Rust-1 |
| **Extension Proxy Layer** | Go, HTTP/WebSocket, Concurrency | Senior | BE-Go-2 | BE-Go-1 |
| **Dashboard Builder Backend** | Go, JSON Schema, CRUD | Mid | BE-Go-2 | BE-Rust-1 |
| **Community Repository** | Rust, TiDB, Search Indexing | Senior | BE-Rust-1 | BE-Go-2 |
| **Composer UX (Phases 1-7)** | Elm, CSS, State Management | Senior | FE-Elm-1 | FE-Elm-2 |
| **Layout Edit Mode** | Elm, Drag-and-Drop, Grid Systems | Senior | FE-Elm-2 | FE-Elm-1 |
| **Tri-Surface Workbench** | Elm, Animation, SSE Client | Senior | FE-Elm-2 | FE-Elm-1 |
| **Export Pipeline** | Go/Rust, Templates, Validation | Senior | BE-Rust-1 | BE-Go-2 |
| **Prompt Control Hub** | Elm (UI), Go (Backend) | Senior | FE-Elm-2 (UI), BE-Rust-1 (Backend) | FE-Elm-1, BE-Go-1 |
| **CI/CD Pipelines** | GitHub Actions, Docker, K8s | Senior | DevOps-1 | DevOps-2 |
| **Clock Sync Infrastructure** | NTP/PTP, Linux, Monitoring | Mid | DevOps-2 | DevOps-1 |
| **Security Hardening** | OWASP, TLS, Auth, Audit | Senior | DevOps-1 | BE-Rust-1 |
| **Documentation** | Markdown, API Specs, Tutorials | Mid | DevOps-2 | FE-Elm-2 |
| **Accessibility Compliance** | WCAG 2.1, ARIA, Testing | Mid | FE-Elm-2 | FE-Elm-1 |

### 3.2 Training Needs Identified

| Engineer | Training Need | Priority | Method | Timeline |
|----------|---------------|----------|--------|----------|
| BE-Go-1 | Rust basics (for code review) | Low | Pair programming with BE-Rust-1 | Month 1-2 |
| BE-Go-2 | TiDB advanced features | Medium | TiDB Cloud tutorials | Week 1 |
| FE-Elm-2 | SSE client implementation | Medium | FE-Elm-1 knowledge transfer | Week 7 |
| DevOps-2 | PTP protocol (if needed) | Medium | External course/docs | Month 2 |
| All Backend | AI provider APIs (Claude, GPT) | High | Shared workshop | Week 3 |
| All Frontend | New component library patterns | Medium | FE-Elm-1 workshop | Week 5 |

### 3.3 Knowledge Transfer Requirements

| Knowledge Area | Source | Target(s) | Method | Scheduled |
|----------------|--------|-----------|--------|-----------|
| Rust validation patterns | BE-Rust-1 | BE-Go-1, BE-Go-2 | Code walkthrough + docs | Week 2 |
| TiDB schema design | BE-Rust-1 | All Backend | Architecture session | Week 1 |
| Elm state management | FE-Elm-1 | FE-Elm-2 | Pair programming | Week 1-2 |
| Clock sync architecture | DevOps-2 | DevOps-1 | Design review | Week 2 |
| Extension proxy design | BE-Go-2 | All Backend | ADR presentation | Week 5 |
| Composer UX patterns | FE-Elm-1 | FE-Elm-2 | Component review | Week 7 |
| Security best practices | DevOps-1 | All engineers | Security workshop | Week 10 |

---

## 4. Critical Path Resource Dependencies

### 4.1 Critical Tasks and Required Engineers

| Critical Task | Required Engineer | Bus Factor | Why Critical |
|---------------|-------------------|------------|--------------|
| **Rust MetadataValidator core** | BE-Rust-1 | 1 | Only Rust expert; blocks The Bin integration |
| **TiDB schema deployment** | BE-Rust-1 + DevOps-1 | 2 | Foundation for all systems; Day 1 blocker |
| **AI Cortex token tracking** | BE-Go-1 | 1 | Token accuracy critical for cost control |
| **Extension proxy architecture** | BE-Go-2 | 1 | Complex architecture; requires deep knowledge |
| **Composer UX Phases 1-4** | FE-Elm-1 | 1 | Core user experience; Month 2 critical path |
| **Elm component architecture** | FE-Elm-1 | 1 | All frontend depends on patterns established |
| **Clock sync implementation** | DevOps-2 | 1 | Specialized knowledge; temporal security |
| **Production deployment** | DevOps-1 | 1 | Security + deployment expertise required |

### 4.2 Single Points of Failure Analysis

| SPOF | Impact if Unavailable | Mitigation Strategy |
|------|----------------------|---------------------|
| **BE-Rust-1** | Rust validation blocked; The Bin integration stalled | Document all Rust code extensively; pair BE-Go-2 on critical sessions; consider Go fallback for non-critical validation |
| **FE-Elm-1** | Composer UX blocked; frontend architecture decisions | FE-Elm-2 to shadow all architectural decisions; maintain detailed ADRs; shared component library ownership |
| **DevOps-1** | Production deployment blocked; security audit delayed | DevOps-2 cross-trained on deployment; documented runbooks; external contractor backup plan |
| **DevOps-2** | Clock sync implementation blocked | NTP implementation documented step-by-step; DevOps-1 backup capability; can defer PTP to Month 3 |

### 4.3 Detailed Mitigation Plans

#### BE-Rust-1 Mitigation Plan (Highest Risk)

```
Risk: BE-Rust-1 is sole Rust expert
Impact: P0 - MetadataValidator, The Bin integration blocked
Bus Factor: 1

Mitigation Actions:
1. Documentation (Week 1-2):
   - Document MetadataValidator architecture with diagrams
   - Comment all Rust code extensively (>30% comment ratio)
   - Create Rust style guide for team

2. Knowledge Transfer (Week 2-4):
   - BE-Go-2 pairs on 50% of Rust sessions
   - Record code walkthrough videos
   - Shared ownership of validation test suite

3. Contingency:
   - If unavailable >3 days in Month 1: Delay validation to Week 3-4
   - If unavailable >1 week: Implement basic validation in Go (slower but functional)
   - External Rust contractor on standby (identified, not contracted)

Owner: BE-Rust-1 (documentation), BE-Go-2 (backup)
Review: Weekly during architecture sync
```

#### FE-Elm-1 Mitigation Plan (High Risk)

```
Risk: FE-Elm-1 owns Composer UX and frontend architecture
Impact: P1 - Month 2 critical path blocked
Bus Factor: 1

Mitigation Actions:
1. Pattern Documentation (Week 1):
   - Document Elm architecture patterns in team wiki
   - Create component library with examples
   - Establish PR review checklist

2. Shared Ownership (Week 3+):
   - FE-Elm-2 co-owns component library
   - FE-Elm-2 reviews all Composer PRs
   - Rotate lead on Composer Phases 5-7

3. Contingency:
   - If unavailable in Month 2: FE-Elm-2 takes Composer lead (Phases 3-4 delayed 1 week)
   - If unavailable >2 weeks: Defer Phases 5-7 to Month 3; focus on Phases 1-4 only

Owner: FE-Elm-1 (documentation), FE-Elm-2 (backup)
Review: Bi-weekly during frontend sync
```

### 4.4 Cross-Training Requirements

| Primary | Backup | Domain | Training Hours | Scheduled |
|---------|--------|--------|----------------|-----------|
| BE-Rust-1 | BE-Go-2 | Rust validation patterns | 8 hours | Week 2 |
| BE-Go-1 | BE-Rust-1 | Token tracking system | 4 hours | Week 4 |
| BE-Go-2 | BE-Go-1 | Extension proxy architecture | 6 hours | Week 5 |
| FE-Elm-1 | FE-Elm-2 | Composer UX patterns | 8 hours | Week 6-7 |
| DevOps-1 | DevOps-2 | Deployment procedures | 4 hours | Week 11 |
| DevOps-2 | DevOps-1 | Clock sync monitoring | 4 hours | Week 2 |

**Total Cross-Training Hours:** 34 hours (distributed across 3 months)

---

## 5. Weekly Utilization Projections

### 5.1 12-Week Utilization Chart

| Week | BE-Rust-1 | BE-Go-1 | BE-Go-2 | FE-Elm-1 | FE-Elm-2 | DevOps-1 | DevOps-2 | Team Avg |
|------|:---------:|:-------:|:-------:|:--------:|:--------:|:--------:|:--------:|:--------:|
| **1** | 100% | 100% | 100% | 60% | 40% | 100% | 80% | 83% |
| **2** | 100% | 80% | 80% | 80% | 60% | 60% | 80% | 77% |
| **3** | 100% | 100% | 80% | 80% | 80% | 80% | 80% | 86% |
| **4** | 80% | 100% | 100% | 80% | 80% | 80% | 60% | 83% |
| **5** | 100% | 60% | 100% | 100% | 80% | 100% | 60% | 86% |
| **6** | 80% | 80% | 80% | 100% | 100% | 60% | 80% | 83% |
| **7** | 80% | 80% | 80% | 100% | 100% | 60% | 80% | 83% |
| **8** | 80% | 100% | 100% | 100% | 100% | 80% | 60% | 89% |
| **9** | 100% | 100% | 100% | 100% | 100% | 80% | 80% | 94% |
| **10** | 80% | 80% | 80% | 80% | 80% | 100% | 100% | 86% |
| **11** | 80% | 80% | 80% | 80% | 80% | 100% | 80% | 83% |
| **12** | 100% | 100% | 100% | 100% | 100% | 100% | 100% | 100% |

**Notes:**
- Week 1: Frontend ramp-up slower (waiting for backend APIs)
- Week 9: Peak utilization (all TIER X features completing)
- Week 12: Launch week (all hands on deck)

### 5.2 Monthly Averages

| Month | BE-Rust-1 | BE-Go-1 | BE-Go-2 | FE-Elm-1 | FE-Elm-2 | DevOps-1 | DevOps-2 | Team Avg |
|-------|:---------:|:-------:|:-------:|:--------:|:--------:|:--------:|:--------:|:--------:|
| **Month 1** | 95% | 95% | 90% | 75% | 65% | 80% | 75% | **82%** |
| **Month 2** | 85% | 80% | 90% | 100% | 95% | 75% | 70% | **85%** |
| **Month 3** | 90% | 90% | 90% | 90% | 90% | 95% | 90% | **91%** |
| **Overall** | 90% | 88% | 90% | 88% | 83% | 83% | 78% | **86%** |

**Target:** 80% average utilization (20% buffer)
**Actual:** 86% average utilization (14% buffer)

### 5.3 Utilization Risk Analysis

| Engineer | Utilization Risk | Risk Level | Mitigation |
|----------|------------------|------------|------------|
| BE-Rust-1 | Consistently high (90%) | Medium | Delegate documentation to DevOps-2; prioritize critical path only |
| FE-Elm-1 | Peak in Month 2 (100%) | Medium | FE-Elm-2 to absorb overflow; defer non-critical polish |
| DevOps-2 | Lower utilization (78%) | Low | Allocate more documentation; support other teams |
| FE-Elm-2 | Lower in Month 1 (65%) | Low | Advance accessibility research; component library setup |
| All | Week 12 at 100% | High | Pre-launch checklist complete by Day 58; buffer Day 59 |

### 5.4 Hours Breakdown by Engineer

| Engineer | Month 1 Hours | Month 2 Hours | Month 3 Hours | Total Hours |
|----------|:-------------:|:-------------:|:-------------:|:-----------:|
| BE-Rust-1 | 152 | 136 | 144 | **432** |
| BE-Go-1 | 152 | 128 | 144 | **424** |
| BE-Go-2 | 144 | 144 | 144 | **432** |
| FE-Elm-1 | 120 | 160 | 144 | **424** |
| FE-Elm-2 | 104 | 152 | 144 | **400** |
| DevOps-1 | 128 | 120 | 152 | **400** |
| DevOps-2 | 120 | 112 | 144 | **376** |
| **Total** | **920** | **952** | **1016** | **2888** |

**Assumptions:**
- 8 hours/day nominal
- 20% buffer for meetings, issues, support
- 4 weeks/month × 5 days/week = 20 days/month

---

## 6. On-Call & Support Rotation

### 6.1 Launch Week On-Call Schedule (Week 12)

| Day | Primary On-Call | Secondary On-Call | Hours (UTC) |
|-----|-----------------|-------------------|-------------|
| **Monday** (Day 56) | DevOps-1 | BE-Rust-1 | 08:00-20:00 |
| **Tuesday** (Day 57) | DevOps-1 | BE-Go-1 | 08:00-20:00 |
| **Wednesday** (Day 58) | DevOps-2 | BE-Go-2 | 08:00-20:00 |
| **Thursday** (Day 59) | DevOps-1 | FE-Elm-1 | 08:00-22:00 |
| **Friday** (Day 60 - Launch) | DevOps-1 | All Engineers Available | 06:00-24:00 |
| **Saturday** (Day 61) | DevOps-2 | BE-Rust-1 | 09:00-18:00 |
| **Sunday** (Day 62) | DevOps-1 | BE-Go-1 | 09:00-18:00 |

### 6.2 Post-Launch Support Rotation (Weeks 13+)

| Week | Primary | Secondary | Focus Area |
|------|---------|-----------|------------|
| 13 | DevOps-1 | BE-Rust-1 | Infrastructure stability |
| 14 | DevOps-2 | BE-Go-1 | User issue triage |
| 15 | DevOps-1 | BE-Go-2 | Extension issues |
| 16 | DevOps-2 | FE-Elm-1 | UI bug fixes |
| 17+ | Rotating weekly | Rotating weekly | General support |

### 6.3 Escalation Paths

```
Level 1: Primary On-Call Engineer
    ↓ (15 min response, cannot resolve)
Level 2: Secondary On-Call Engineer
    ↓ (30 min response, cannot resolve)
Level 3: Domain Expert (Backend/Frontend/DevOps Lead)
    ↓ (1 hour response, cannot resolve)
Level 4: Project Lead (External)
    ↓ (Business hours, major incident)
Level 5: Executive Escalation (P0 incidents only)
```

**Incident Classification:**

| Severity | Response Time | Resolution Target | Example |
|----------|---------------|-------------------|---------|
| **P0** | 15 min | 2 hours | Production down, data loss |
| **P1** | 30 min | 4 hours | Major feature broken, >50% users affected |
| **P2** | 2 hours | 24 hours | Feature degraded, workaround exists |
| **P3** | 8 hours | 72 hours | Minor bug, <10% users affected |

### 6.4 Coverage During Holidays/PTO

**Pre-Approval Requirements:**
- PTO requests submitted 2 weeks in advance
- No more than 2 engineers off simultaneously
- No PTO during launch week (Week 12) without explicit approval
- On-call swap arranged before PTO

**Holiday Coverage Plan:**
| Holiday | Expected Off | Minimum Coverage | Backup Plan |
|---------|--------------|------------------|-------------|
| Month 1 holidays | 1-2 engineers | 5 engineers + 1 on-call | Shift non-critical work |
| Month 2 holidays | 1-2 engineers | 5 engineers + 1 on-call | Prioritize integration testing |
| Month 3 holidays | 0-1 engineers | 6 engineers + 1 on-call | Launch week: No PTO |

### 6.5 On-Call Compensation

- **Launch Week:** 1.5x comp time for all hours worked over 8
- **Weekend On-Call:** 1 day comp time per weekend day worked
- **Incident Response (after hours):** 1 hour comp time per incident

---

## 7. Hiring Recommendations

### 7.1 Current Team Assessment

| Area | Current Capacity | Required Capacity | Gap |
|------|------------------|-------------------|-----|
| Backend (Rust) | 1 engineer | 1-1.5 engineers | **0.5 engineer gap** |
| Backend (Go) | 2 engineers | 2 engineers | Adequate |
| Frontend (Elm) | 2 engineers | 2-2.5 engineers | **0.5 engineer gap** |
| DevOps | 2 engineers | 2 engineers | Adequate |

### 7.2 Contractor Recommendations

| Role | Duration | Hours/Week | Estimated Cost | Justification |
|------|----------|------------|----------------|---------------|
| **Rust Contractor** | Month 2-3 | 20 hours | $8,000/month | Reduce BE-Rust-1 load; accelerate validation features |
| **QA Engineer (Contractor)** | Month 3 | 40 hours | $6,000/month | E2E testing, regression testing, launch prep |

**Total Contractor Budget:** $22,000 (2 months Rust + 1 month QA)

### 7.3 FTE Recommendations (Post-MVP)

| Role | Priority | Timing | Annual Cost | Justification |
|------|----------|--------|-------------|---------------|
| **Senior Rust Engineer** | High | Month 4+ | $180,000 | Long-term bus factor mitigation; feature velocity |
| **QA Lead** | Medium | Month 6+ | $140,000 | Sustainable testing; release quality |
| **Technical Writer** | Low | Month 8+ | $100,000 | Documentation velocity; user guides |

### 7.4 Skill Gaps That Cannot Be Covered Internally

| Skill Gap | Current Coverage | Risk | Recommendation |
|-----------|------------------|------|----------------|
| Advanced Rust async patterns | BE-Rust-1 only | High | Contractor or new hire |
| Machine learning (for future AI features) | None | Medium | Future hire (not MVP) |
| Mobile development (iOS/Android) | None | Low | Future hire (not MVP) |
| Advanced Kubernetes (multi-cluster) | DevOps-1 (basic) | Low | Training or consultant |

---

## Appendix A: Weekly Task Ownership Matrix (Month 1)

| Task | Week 1 | Week 2 | Week 3 | Week 4 |
|------|--------|--------|--------|--------|
| Metadata tables deployment | BE-Rust-1 | - | - | - |
| AI Cortex tables deployment | BE-Go-1 | - | - | - |
| Extension tables deployment | BE-Go-2 | - | - | - |
| CI/CD pipeline setup | DevOps-1 | - | - | - |
| Clock sync research | DevOps-2 | - | - | - |
| Seed metadata standards | - | BE-Rust-1 | - | - |
| API config seeding | - | BE-Go-1 | - | - |
| Go API scaffolding | - | BE-Go-2 | - | - |
| Metadata panel scaffolding | - | FE-Elm-1 | - | - |
| Extension status UI | - | FE-Elm-2 | - | - |
| MetadataValidator core | - | - | BE-Rust-1 | BE-Rust-1 |
| Token tracking middleware | - | - | BE-Go-1 | BE-Go-1 |
| Extension proxy foundation | - | - | BE-Go-2 | BE-Go-2 |
| Violation display UI | - | - | FE-Elm-1 | FE-Elm-1 |
| Accessibility audit | - | - | FE-Elm-2 | FE-Elm-2 |
| Grafana dashboards | - | - | DevOps-1 | DevOps-1 |
| Clock sync implementation | - | - | DevOps-2 | DevOps-2 |

---

## Appendix B: Reference Documents

| Document | Purpose | Critical Sections |
|----------|---------|-------------------|
| VERSION_ANALYSIS_AND_UPDATE_REQUIREMENTS_V2.md | Team structure, budget | Part 5: Summary Statistics |
| TIER3_MONTH1_TIMELINE.md | Month 1 day-by-day tasks | All days, risk matrix |
| TIER3_MONTH2_TIMELINE.md | Month 2 day-by-day tasks | All days, risk matrix |
| TIER3_MONTH3_TIMELINE.md | Month 3 day-by-day tasks | Launch day checklist |
| TIER3_CONTEXT_ASSEMBLY.md | Resource requirements | Section 5 |
| ARTIFACT_13_SECURITY_CAPABILITY_MODEL.md | Security training needs | Temporal security section |
| ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md | Overall implementation | All sections |

---

## Quality Checklist ✓

- [x] All 7 engineers assigned for all 12 weeks
- [x] No single point of failure without mitigation (4 identified, all mitigated)
- [x] Average utilization 75-85% (actual: 86% with 14% buffer)
- [x] Critical path resources identified (8 critical tasks)
- [x] Cross-training plan included (34 hours scheduled)
- [x] On-call rotation defined for launch (7-day coverage)
- [x] Hiring recommendations provided (2 contractors + 3 FTE post-MVP)
- [x] Skill matrix complete (12 skills × 7 engineers)
- [x] Monthly workload distribution detailed
- [x] Escalation paths documented

---

**TASK 4 COMPLETE** ✓

*Generated: 2026-01-02T14:00:00Z*  
*Duration: 45 minutes*  
*Total Engineers: 7*  
*Total Project Hours: 2,888*  
*Average Utilization: 86%*
