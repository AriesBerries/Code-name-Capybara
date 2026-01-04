# DevGuide Cockpit: Dashboard Template Catalog

**Document Type:** Mechanics (Tier 3)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** ARTIFACT_10_TEMPLATE_SYSTEM_SPECIFICATION.md, ARTIFACT_02_TERMINOLOGY_GLOSSARY.md

---

## Executive Summary

This artifact provides **complete specifications for the 3 MVP dashboard templates** included with DevGuide Cockpit. Each template is a pre-configured workflow optimized for specific use cases:

1. **Vibe Coding** (6 dashboards) - Web app development for non-technical users
2. **Data Science** (7 dashboards) - Research and experimentation workflows  
3. **Note Taking** (4 dashboards) - Personal knowledge management

---

## Template Comparison Matrix

| Feature | Vibe Coding | Data Science | Note Taking |
|---------|-------------|--------------|-------------|
| **Dashboard Count** | 6 (locked) | 7 (locked) | 4 (locked) |
| **Target Users** | Non-technical builders | Data scientists, researchers | Writers, knowledge workers |
| **Complexity** | Simple apps | Experiments & models | Knowledge graphs |
| **AI Intensity** | High (code generation) | High (model suggestions) | Medium (link suggestions) |
| **Propagation Default** | Gated (user review) | Auto (fast iteration) | Auto (interconnected) |
| **Code Generation** | Yes (Elm/Go/Rust) | Yes (Python notebooks) | Minimal (markdown only) |
| **Guardrails** | Restrictive (simplicity) | Flexible (experimentation) | Lightweight (minimal friction) |
| **Version Control** | Git | Git + DVC (data versioning) | Git |
| **Collaboration** | Post-MVP | Individual-focused | Individual-focused |

---

## 1. Vibe Coding Template

### Overview

**Philosophy:** Emulate Lovable/Cursor for rapid web app prototyping  
**Target Audience:** Non-technical "vibe coders" with high AI reliance  
**Key Value:** Minimal decisions, maximum guidance, fast iteration to deployment

### Dashboards (6 total, 4 required + 2 optional)

#### Dashboard 1: Project Inception (Required)

**Purpose:** Define project vision, success criteria, and initial requirements

**Workflow Modules:**
- Vision Statement Editor (AI-assisted)
- User Stories Builder (drag-and-drop cards)
- Success Metrics Tracker (quantified goals with progress)
- Stakeholder Map (visual diagram with D3.js)

**Default Configuration:**
```yaml
max_features: 5
require_success_metrics: true
ai_vision_generation: enabled
template_presets:
  - "SaaS Platform"
  - "E-commerce Store"
  - "Content Management System"
  - "Social Network"
```

**Generated Outputs:**
- `project.yaml` (master config)
- `README.md` (generated from template)
- `USER_STORIES.md`
- `STAKEHOLDERS.md`

**Guardrails:**
- Maximum 5 core features (enforce MVP focus)
- Success criteria must be measurable (AI validates)
- At least 2 user personas required

**AI Prompts:**
```yaml
vision_generator:
  prompt: |
    User wants to build: {user_input}
    Generate a concise vision statement (2-3 sentences) that:
    1. Identifies the problem being solved
    2. Describes the target audience
    3. States the unique value proposition
  max_tokens: 500
  temperature: 0.7

user_story_suggester:
  prompt: |
    Given vision: {vision_statement}
    Suggest 5-10 user stories in the format:
    "As a [persona], I want to [action] so that [benefit]"
  max_tokens: 1000
  temperature: 0.8
```

**Completion Criteria:**
- All required inputs filled
- Vision statement approved by user
- Generated README reviewed
- Dashboard status: Locked (cannot modify without unlocking project)

---

#### Dashboard 2: Architecture Canvas (Required)

**Purpose:** Visual system design with component diagram

**Workflow Modules:**
- Component Diagram Editor (D3.js force-directed graph)
- Data Flow Mapper (shows API calls between services)
- Dependency Tracker (identifies circular dependencies)
- Architecture Decision Logger (ADRs embedded)

**Default Configuration:**
```yaml
max_services: 3
enforce_microservices: false
visualization_library: "d3.js"
auto_layout: true
show_dependencies: true
```

**Generated Outputs:**
- `ARCHITECTURE.md` (arc42 format)
- `diagrams/component-diagram.svg`
- `diagrams/data-flow.svg`
- `adrs/ADR-001-architecture-style.md`

**Guardrails:**
- Maximum 3 microservices (Vibe Coding focuses on simplicity)
- No circular dependencies allowed (detected by Rust WASM)
- Each service must have defined API contract

**Capabilities Required:**
- `RenderCanvas` (for D3.js visualization)
- `ExecuteJavaScript` (for interactive diagram)
- `WriteViolations` (report architecture violations to The Bin)

---

#### Dashboard 3: Tech Stack Matrix (Required)

**Purpose:** Technology selection with AI-powered recommendations

**Workflow Modules:**
- Decision Matrix (weighted criteria)
- Technology Comparison (side-by-side feature table)
- ADR Generator (automatic documentation)
- Dependency Graph (visualize tech stack dependencies)

**Default Configuration:**
```yaml
required_languages: ["elm", "go", "rust"]
allow_language_mixing: false
ai_recommendations: enabled
auto_adr_generation: true
```

**Generated Outputs:**
- `TECH_STACK.md`
- `adrs/ADR-002-database-selection.md`
- `adrs/ADR-003-frontend-framework.md`
- `package.json` / `go.mod` / `Cargo.toml` (initialized)

**Guardrails:**
- Must use Elm, Go, and Rust (no other languages)
- Database selection limited to PostgreSQL or TiDB
- Frontend framework must be Elm (no React/Vue/Angular)

**AI Validation:**
```yaml
tech_stack_validator:
  prompt: |
    Tech stack selected: {tech_stack}
    Project vision: {vision}
    
    Does this stack align with the project goals?
    Are there any compatibility issues?
    Suggest alternatives if needed.
  max_tokens: 1500
```

---

#### Dashboard 4: Security Checklist (Optional)

**Purpose:** Security best practices and vulnerability scanning

**Workflow Modules:**
- Security Checklist (OWASP Top 10)
- Dependency Scanner (detect vulnerable packages)
- Secrets Detector (prevent hardcoded keys)
- Threat Model Builder (STRIDE framework)

**Default Configuration:**
```yaml
enable_dependency_scan: true
enable_secrets_scan: true
owasp_checklist: true
threat_modeling_required: false
```

**Guardrails:**
- No plaintext secrets allowed (hard block)
- All API endpoints require authentication (soft warning)
- HTTPS required for production deployment

---

#### Dashboard 5: Quality Gates (Optional)

**Purpose:** Test strategy and code quality enforcement

**Workflow Modules:**
- Test Strategy Planner (unit, integration, E2E)
- Coverage Tracker (real-time from CI/CD)
- Quality Metrics (cyclomatic complexity, duplication)
- Test Generator (AI-suggested test cases)

**Default Configuration:**
```yaml
min_test_coverage: 80
enable_mutation_testing: false
auto_generate_tests: true
quality_gate_enforcement: "soft"
```

---

#### Dashboard 6: Deployment Orchestrator (Required)

**Purpose:** CI/CD pipeline configuration and deployment management

**Workflow Modules:**
- Pipeline Builder (visual workflow editor)
- Environment Manager (dev, staging, prod)
- Deployment Trigger (manual or automatic)
- Rollback Manager (30-day rollback window)

**Default Configuration:**
```yaml
ci_cd_provider: "github-actions"
deployment_target: "docker"
auto_deploy_on_merge: false
health_check_required: true
```

**Generated Outputs:**
- `.github/workflows/ci.yml`
- `.github/workflows/deploy.yml`
- `Dockerfile`
- `docker-compose.yml`
- `DEPLOYMENT.md`

---

### Dashboard Workflow Sequence

```
Project Inception → Architecture Canvas → Tech Stack Matrix
                                              ↓
                                  (Optional: Security Checklist)
                                              ↓
                                  (Optional: Quality Gates)
                                              ↓
                                  Deployment Orchestrator
```

**Lock Mechanism:**
- User must complete required dashboards in sequence
- Optional dashboards can be added at any time
- Cannot deploy until all required dashboards completed

---

## 2. Data Science Template

### Overview

**Philosophy:** Emulate Jupyter/Observable for reproducible research  
**Target Audience:** Data scientists, ML engineers, researchers  
**Key Value:** Experiment tracking, reproducibility, model versioning

### Dashboards (7 total, 5 required + 2 optional)

#### Dashboard 1: Research Question Definition (Required)

**Purpose:** Clearly define research question and hypothesis

**Workflow Modules:**
- Research Question Editor
- Hypothesis Builder (if/then format)
- Literature Review Tracker (citations + notes)
- Constraints Documenter (time, budget, data availability)

**Generated Outputs:**
- `RESEARCH.md`
- `hypothesis.yaml`
- `LITERATURE.bib`

---

#### Dashboard 2: Data Source Configuration (Required)

**Purpose:** Configure data pipelines and track lineage

**Workflow Modules:**
- Data Connector (SQL, APIs, CSV, Parquet)
- Schema Validator (automatic schema inference)
- Lineage Tracker (data provenance graph)
- Privacy Checker (PII detection)

**Guardrails:**
- PII detection and masking required
- Data versioning mandatory (DVC integration)
- Source provenance must be documented

**Generated Outputs:**
- `data/schemas/*.json`
- `data/lineage.yaml`
- `.dvc/config`

---

#### Dashboard 3: Experiment Notebook (Required)

**Purpose:** Interactive notebook for experiments

**Workflow Modules:**
- Code Cells (Python/R/Julia execution)
- Visualization Renderer (matplotlib, plotly, seaborn)
- Experiment Logger (MLflow integration)
- Checkpoint Manager (auto-save notebooks)

**Default Configuration:**
```yaml
kernel: "python3"
auto_save_interval: 300  # 5 minutes
mlflow_tracking: true
seed_randomness: true  # Reproducibility
```

**Generated Outputs:**
- `notebooks/*.ipynb`
- `mlruns/` (MLflow experiments)
- `checkpoints/`

---

#### Dashboard 4: Model Registry (Optional)

**Purpose:** Version and manage trained models

**Workflow Modules:**
- Model Uploader
- Metadata Tagger (hyperparameters, metrics)
- Performance Tracker (accuracy, loss over time)
- Deployment Promoter (dev → staging → prod)

---

#### Dashboard 5: Visualization Builder (Optional)

**Purpose:** Create publication-quality visualizations

**Workflow Modules:**
- Chart Designer (ggplot2, Vega-Lite)
- Interactive Dashboard (Observable, Streamlit)
- Export Manager (PNG, SVG, PDF, HTML)

---

#### Dashboard 6: Pipeline Orchestrator (Required)

**Purpose:** Automate data processing and model training

**Workflow Modules:**
- DAG Designer (Airflow/Prefect/Dagster visual editor)
- Schedule Configurator (cron, event-based)
- Dependency Manager (upstream/downstream tasks)
- Error Handler (retry policies, alerts)

**Generated Outputs:**
- `pipelines/*.py` (Airflow DAGs)
- `dags/config.yaml`

---

#### Dashboard 7: Results Publication (Required)

**Purpose:** Generate reports and papers from notebooks

**Workflow Modules:**
- Report Generator (Jupyter Book, Quarto)
- Citation Manager (BibTeX integration)
- LaTeX Exporter (for academic papers)
- Collaborative Review (comments + suggestions)

**Generated Outputs:**
- `reports/*.pdf`
- `reports/*.html`
- `paper.tex`

---

## 3. Note Taking Template

### Overview

**Philosophy:** Emulate Obsidian/Notion for personal knowledge management  
**Target Audience:** Writers, researchers, knowledge workers  
**Key Value:** Bidirectional links, graph view, personal knowledge base

### Dashboards (4 total, 2 required + 2 optional)

#### Dashboard 1: Note Capture (Required)

**Purpose:** Quick note entry with markdown support

**Workflow Modules:**
- Quick Capture (hotkey-accessible: Ctrl+N)
- Markdown Editor (live preview)
- Tag Suggester (AI-powered based on content)
- Link Autocomplete (suggest existing notes)

**Default Configuration:**
```yaml
auto_save: true
save_interval: 10  # seconds
enable_vim_mode: false
default_folder: "inbox"
```

**Generated Outputs:**
- `notes/*.md` (individual note files)
- `tags.yaml` (tag index)

---

#### Dashboard 2: Knowledge Graph (Required)

**Purpose:** Visualize connections between notes

**Workflow Modules:**
- Graph Renderer (force-directed layout, D3.js)
- Cluster Detector (identify topic groups via AI)
- Path Finder (shortest path between two notes)
- Orphan Detector (unlinked notes warning)

**Capabilities:**
- `RenderCanvas`
- `ExecuteJavaScript`

**Generated Outputs:**
- `graph/knowledge-graph.json`
- `graph/clusters.yaml`

---

#### Dashboard 3: Tag Taxonomy (Optional)

**Purpose:** Organize notes hierarchically

**Workflow Modules:**
- Tag Tree Builder (hierarchical tags)
- Synonym Manager (merge similar tags)
- Tag Merger (bulk tag operations)
- Tag Analytics (most used, trending)

**Generated Outputs:**
- `taxonomy.yaml`

---

#### Dashboard 4: Publication Engine (Optional)

**Purpose:** Export notes to blog/website

**Workflow Modules:**
- Static Site Generator (11ty, Hugo, Jekyll)
- Theme Selector
- SEO Optimizer (meta tags, sitemap)
- Deployment Manager (Netlify, Vercel)

**Generated Outputs:**
- `site/` (static site files)
- `site/config.toml`

---

## Dashboard-to-SDLC Phase Mapping

### Original 9-Phase Checklist → Vibe Coding Dashboards

| SDLC Phase | Dashboard | Notes |
|------------|-----------|-------|
| 1. Requirements Gathering | Project Inception | User stories, success metrics |
| 2. System Design | Architecture Canvas | Component diagram, data flow |
| 3. Detailed Design | Tech Stack Matrix | Concrete technology choices |
| 4. Implementation | (External - IDE) | Continue.dev alternative handles coding |
| 5. Testing | Quality Gates | Test strategy, coverage tracking |
| 6. Deployment | Deployment Orchestrator | CI/CD, infrastructure |
| 7. Monitoring | Deployment Orchestrator | Observability setup |
| 8. Security | Security Checklist | Auth, encryption, vulnerability scan |
| 9. Documentation | (Auto-generated) | Jinja2 templates from all dashboards |

**Key Insight:** Implementation (Phase 4) happens in IDEs (Kiro AI, AIDE, Void Editor). Dashboards orchestrate *around* coding, not replace it.

---

## Template Migration

**Can users switch templates mid-project?**

**Answer:** No, templates are locked at project initialization.

**Workaround:** Create new project from different template, manually migrate content.

**Future Feature (Post-MVP):** Template migration wizard
1. Analyze current project state
2. Map existing dashboards to new template
3. Show diff of required changes
4. User reviews and approves
5. Automated migration with rollback option

---

## Community Template Contribution

**Process:**
```
1. Developer creates custom template (extends base)
2. Submits to certification queue
3. Governance review (see COMMUNITY_GOVERNANCE_SPECIFICATION.md)
4. If approved → Tier 1 (Certified)
5. If not → Tier 2 (Community, use at own risk)
```

**Example Custom Template:** "E-commerce Template"
- Dashboards: Product Catalog, Payment Integration, Order Management, Shipping, Analytics
- Built by community member
- Reviewed and certified by DevGuide team
- Available in template library

---

## Open Questions

1. Should there be a "Blank Template" with zero dashboards? (Suggest: Yes, for advanced users)
2. Maximum custom templates per user? (Suggest: 10 templates)
3. Can dashboards be reordered after project lock? (Suggest: No, sequence is immutable)
4. Different pricing tiers for templates? (Suggest: Post-MVP)

---

**End of ARTIFACT_11_DASHBOARD_TEMPLATE_CATALOG.md**
