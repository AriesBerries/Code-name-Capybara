# Workflow Patterns
**Last Updated:** December 27, 2025  
**Purpose:** Reusable patterns for common development workflows

---

## Pattern Index

| Pattern | Use Case | Complexity |
|---------|----------|------------|
| [Feature Implementation](#pattern-1-feature-implementation) | Building new features | Medium |
| [Bug Fix](#pattern-2-bug-fix) | Fixing defects | Low |
| [Code Refactoring](#pattern-3-code-refactoring) | Improving existing code | Medium |
| [Documentation Update](#pattern-4-documentation-update) | Updating docs | Low |
| [Configuration Change](#pattern-5-configuration-change) | Changing configs | Low |
| [Research & Spike](#pattern-6-research--spike) | Investigating options | Low |
| [Release Preparation](#pattern-7-release-preparation) | Preparing deployment | High |
| [Onboarding New Project](#pattern-8-onboarding-new-project) | Setting up new project | Medium |

---

## Pattern 1: Feature Implementation

**Use When:** Building a new capability

### Workflow
```
┌────────────────────────────────────────────────────────────┐
│  PHASE 1: PLAN (Advisory Layer / Desktop)                   │
│  ├── Review requirements                                    │
│  ├── Identify affected files                                │
│  ├── Select tools                                           │
│  └── Output: Implementation plan with tool annotations      │
├────────────────────────────────────────────────────────────┤
│  PHASE 2: PREPARE (CLI)                                     │
│  ├── Create feature branch                                  │
│  ├── Search existing patterns (Grep)                        │
│  └── Read related files                                     │
├────────────────────────────────────────────────────────────┤
│  PHASE 3: IMPLEMENT (CLI)                                   │
│  ├── Create new files (Write)                               │
│  ├── Modify existing files (Edit)                           │
│  ├── Add tests                                              │
│  └── Run tests (Bash)                                       │
├────────────────────────────────────────────────────────────┤
│  PHASE 4: VERIFY (CLI)                                      │
│  ├── Run full test suite                                    │
│  ├── Check for lint errors                                  │
│  └── Review changes (git diff)                              │
├────────────────────────────────────────────────────────────┤
│  PHASE 5: COMMIT (CLI)                                      │
│  ├── Stage changes (git add)                                │
│  ├── Commit with message (git commit)                       │
│  ├── Push branch (git push)                                 │
│  └── Create PR (github MCP or gh CLI)                       │
└────────────────────────────────────────────────────────────┘
```

### Tool Selection
| Phase | Tools |
|-------|-------|
| Plan | Desktop conversation, sequential-thinking MCP |
| Prepare | CLI: Bash (git), Grep, Read |
| Implement | CLI: Write, Edit, Bash |
| Verify | CLI: Bash (test runner) |
| Commit | CLI: Bash (git), github MCP |

---

## Pattern 2: Bug Fix

**Use When:** Fixing a defect

### Workflow
```
1. REPRODUCE
   └─► [CLI] Run failing test or reproduce steps

2. LOCATE
   ├─► [CLI: Grep] Search for error message
   ├─► [CLI: Read] Examine suspicious files
   └─► [CLI: Grep] Find related code

3. UNDERSTAND
   └─► [Desktop] Discuss root cause if complex

4. FIX
   ├─► [CLI: Edit] Modify code
   └─► [CLI: Write] Add regression test

5. VERIFY
   ├─► [CLI: Bash] Run tests
   └─► [CLI] Confirm fix

6. COMMIT
   └─► [CLI] git commit with "fix: " prefix
```

### Tool Selection
| Step | Primary Tool | Fallback |
|------|--------------|----------|
| Reproduce | CLI: Bash | Manual steps |
| Locate | CLI: Grep | CLI: Read multiple files |
| Fix | CLI: Edit | CLI: Write (if new file) |
| Verify | CLI: Bash | Manual testing |
| Commit | CLI: Bash (git) | - |

---

## Pattern 3: Code Refactoring

**Use When:** Improving code without changing behavior

### Workflow
```
1. SCOPE
   ├─► [Advisory] Define refactoring boundaries
   └─► [CLI: Grep] Find all instances to change

2. TEST BASELINE
   └─► [CLI: Bash] Run tests, confirm passing

3. REFACTOR INCREMENTALLY
   ├─► [CLI: Edit] Make single change
   ├─► [CLI: Bash] Run tests
   ├─► [CLI: Bash] git commit -m "refactor: specific change"
   └─► Repeat for each change

4. FINAL VERIFICATION
   ├─► [CLI: Bash] Run full test suite
   └─► [CLI: Bash] git diff --stat (review total changes)
```

### Key Principle
**Small commits, frequent tests.** Never refactor more than you can safely roll back.

---

## Pattern 4: Documentation Update

**Use When:** Updating docs to match reality

### Workflow
```
1. IDENTIFY OUTDATED
   ├─► [CLI: Glob] Find all .md files
   └─► [CLI: Grep] Search for outdated terms

2. UPDATE
   ├─► [CLI: Edit] Modify documentation
   └─► [CLI: Write] Create new docs if needed

3. VERIFY
   ├─► [CLI: Read] Review changes
   └─► [Desktop] For complex docs, discuss clarity

4. COMMIT
   └─► [CLI] git commit -m "docs: description"
```

### Tool Selection
| Task | Tool |
|------|------|
| Find docs | CLI: Glob *.md |
| Search content | CLI: Grep |
| Update | CLI: Edit |
| Review | CLI: Read or Desktop |

---

## Pattern 5: Configuration Change

**Use When:** Modifying config files

### Workflow
```
1. BACKUP
   └─► [CLI: Bash] Copy current config

2. UNDERSTAND
   └─► [CLI: Read] Review current configuration

3. MODIFY
   └─► [CLI: Edit] Make change

4. VALIDATE
   ├─► [CLI: Bash] Run validation command (if available)
   └─► [CLI: Bash] Test affected functionality

5. COMMIT
   └─► [CLI] git commit -m "config: description"
```

### Risk Mitigation
- Always backup before changing
- Test in isolation if possible
- Document the change reason

---

## Pattern 6: Research & Spike

**Use When:** Investigating options before committing

### Workflow
```
1. DEFINE QUESTION
   └─► [Desktop] Clarify what we're researching

2. GATHER INFORMATION
   ├─► [fetch MCP] Read documentation
   ├─► [CLI: Grep] Search existing codebase
   └─► [Desktop] Discuss findings

3. PROTOTYPE (if needed)
   ├─► [CLI] Create spike branch
   ├─► [CLI: Write] Create prototype
   └─► [CLI: Bash] Test prototype

4. DOCUMENT FINDINGS
   ├─► [CLI: Write] Create findings.md
   └─► [Desktop] Discuss implications

5. DECIDE
   └─► [Advisory] Tool recommendations for implementation
```

### Output
Spike should produce a decision document, not production code.

---

## Pattern 7: Release Preparation

**Use When:** Preparing for deployment

### Workflow
```
1. VERIFY STATUS
   ├─► [CLI: Bash] git status (clean working tree)
   ├─► [CLI: Bash] npm test (all tests pass)
   └─► [CLI: Bash] npm run lint (no warnings)

2. UPDATE VERSION
   ├─► [CLI: Edit] Update version in package.json
   └─► [CLI: Edit] Update CHANGELOG.md

3. BUILD
   └─► [CLI: Bash] npm run build

4. TAG
   ├─► [CLI: Bash] git add .
   ├─► [CLI: Bash] git commit -m "release: v1.2.3"
   └─► [CLI: Bash] git tag v1.2.3

5. PUSH
   ├─► [CLI: Bash] git push
   └─► [CLI: Bash] git push --tags

6. CREATE RELEASE
   └─► [github MCP] Create GitHub release
```

---

## Pattern 8: Onboarding New Project

**Use When:** Setting up Claude for a new project

### Workflow
```
1. INITIALIZE
   ├─► [CLI] Navigate to project
   ├─► [CLI: /init] Create CLAUDE.md
   └─► [CLI] Create .claude/commands/ folder

2. CONFIGURE
   ├─► [CLI: Edit] Customize CLAUDE.md
   ├─► [CLI: Write] Create custom commands
   └─► [Desktop] Update MCP server paths if needed

3. DOCUMENT IN ADVISORY
   ├─► [Advisory] Create projects/<n>/ folder
   ├─► [CLI: Write] Create PROJECT_BRIEF.md
   ├─► [CLI: Write] Create CONSTRAINTS.md
   └─► [CLI: Write] Create CURRENT_PHASE.md

4. TEST
   ├─► [CLI: /doctor] Verify setup
   └─► [CLI] Test a simple task

5. SYNC
   ├─► [CLI] Commit project setup
   └─► [Advisory] Commit project brief
```

---

## Pattern Selection Guide

| Situation | Pattern |
|-----------|---------|
| "Build feature X" | Feature Implementation |
| "Fix bug Y" | Bug Fix |
| "Clean up code" | Code Refactoring |
| "Update README" | Documentation Update |
| "Change setting" | Configuration Change |
| "Should we use X?" | Research & Spike |
| "Deploy version" | Release Preparation |
| "New project" | Onboarding New Project |

---

*Reference these patterns when advising on implementation approach.*
