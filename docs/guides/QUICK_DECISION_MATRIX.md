---
title: GitHub Migration - Quick Decision Matrix
version: 1.0.0
date: 2026-01-03T00:00:00Z
priority: URGENT
---

# GitHub Migration - Quick Decision Matrix

## URGENT DECISIONS REQUIRED BEFORE MIGRATION

### Decision #1: Draft Specifications Disposition (BLOCKING)
**Timeline:** Must decide TODAY  
**Impact:** Affects artifact numbering, ARTIFACT_19 timeline, database schema

| Specification | Size | DB Tables | Option A | Option B | Option C |
|--------------|------|-----------|----------|----------|----------|
| AI_CORTEX_MANAGEMENT_DASHBOARD.md | 56KB | 4 tables | → ARTIFACT_20 | → /future-enhancements/ | → Merge into ARTIFACT_04 |
| VSCODE_EXTENSION_ORCHESTRATION_SYSTEM.md | 45KB | 3 tables | → ARTIFACT_21 | → /future-enhancements/ | → Merge into ARTIFACT_06 |
| ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md | 26KB | 4 tables | → ARTIFACT_22 or keep as 20 | → /future-enhancements/ | → Merge into ARTIFACT_06 |

**RECOMMENDATION: Option A**

**Rationale:**
1. All 3 specs are referenced in ARTIFACT_19 v2.1.0 implementation timeline
2. AI Cortex: Week 4 implementation (Month 1)
3. VS Code Extension: Week 5 implementation (Month 2)
4. Metadata Governance: Week 1-3 implementation (Month 1)
5. All 11 database tables are in ARTIFACT_05 v1.1.0 as official schema
6. Moving to future-enhancements breaks ARTIFACT_19 timeline

**If Option A chosen:**
- [ ] Rename `AI_CORTEX_MANAGEMENT_DASHBOARD.md` → `ARTIFACT_20_AI_CORTEX_MANAGEMENT_DASHBOARD.md`
- [ ] Rename `VSCODE_EXTENSION_ORCHESTRATION_SYSTEM.md` → `ARTIFACT_21_VSCODE_EXTENSION_ORCHESTRATION.md`
- [ ] Keep or rename `ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md` → `ARTIFACT_22_METADATA_GOVERNANCE_DASHBOARD.md`
- [ ] Update artifact count: 19 → 22 (or 20-21 if metadata is already #20)
- [ ] Update PROJECT_COMPLETION_SUMMARY.md
- [ ] Update README.md
- [ ] Update all cross-references in other artifacts

**Time Required:** 2-3 hours

---

### Decision #2: TIER X Specifications Status (HIGH PRIORITY)
**Timeline:** Must decide within 24 hours  
**Impact:** Affects repository structure, MVP scope

| Specification | Size | In ARTIFACT_19? | MVP or Post-MVP? |
|--------------|------|-----------------|------------------|
| COMPOSER_UX_COMPLETE_SPECIFICATION.md | 99KB | ? | ? |
| LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md | 77KB | ? | ? |
| TRI_SURFACE_WORKBENCH_COMPLETE_SPECIFICATION.md | 112KB | ? | ? |
| EXPORT_PIPELINE_COMPLETE_SPECIFICATION.md | 113KB | ? | ? |
| PROMPT_CONTROL_HUB_COMPLETE_SPECIFICATION.md | 164KB | ? | ? |

**ACTION REQUIRED:**
1. Search ARTIFACT_19 for references to these specs
2. Determine if they're part of 3-month MVP timeline
3. If MVP: keep in `/docs/specifications/`
4. If post-MVP: move to `/docs/specifications/future-enhancements/`

**Time Required:** 1-2 hours review

---

### Decision #3: Artifact Versioning (MEDIUM PRIORITY)
**Timeline:** Can be done during migration  
**Impact:** Professional appearance, changelog tracking

**Missing Version Headers:**
- [ ] ARTIFACT_03_THE_BIN_SPECIFICATION.md → add v1.1.0
- [ ] ARTIFACT_04_UNIFIED_AI_LAYER.md → add v1.1.0
- [ ] ARTIFACT_06_MASTER_CONTROL_DASHBOARD.md → add v1.1.0
- [ ] ARTIFACT_13_SECURITY_CAPABILITY_MODEL.md → add v1.1.0
- [ ] ARTIFACT_16_TECH_STACK_UPDATE.md → determine version (v1.0.0 or v1.1.0?)

**Template for version header:**
```yaml
---
title: [Artifact Title]
version: 1.1.0
date: 2026-01-02T00:00:00Z
status: Approved
supersedes: [previous version if any]
---

# [Artifact Title]

## Changelog

### v1.1.0 (2026-01-02)
- Added integration with ARTIFACT_20 (Metadata Governance)
- Updated database schema references
- Added new sections for AI Cortex integration

### v1.0.0 (2026-01-01)
- Initial version
```

**Time Required:** 1 hour

---

## PRIORITY ACTION SEQUENCE

### Step 1: Make Decision #1 (TODAY - 2-3 hours)
**Owner:** Project Lead  
**Output:** Clear decision on Option A/B/C

**If Option A (Promote to Official Artifacts):**
```bash
# Execute these renames
mv AI_CORTEX_MANAGEMENT_DASHBOARD.md ARTIFACT_20_AI_CORTEX_MANAGEMENT_DASHBOARD.md
mv VSCODE_EXTENSION_ORCHESTRATION_SYSTEM.md ARTIFACT_21_VSCODE_EXTENSION_ORCHESTRATION.md

# Determine if ARTIFACT_20_METADATA needs renaming
# If there are now 3 new artifacts, metadata might be ARTIFACT_22
# OR if ARTIFACT_20 was never official, keep it as is

# Update artifact count in all referencing documents
```

**If Option B (Move to Future Enhancements):**
```bash
mkdir -p future-enhancements/
mv AI_CORTEX_MANAGEMENT_DASHBOARD.md future-enhancements/
mv VSCODE_EXTENSION_ORCHESTRATION_SYSTEM.md future-enhancements/
mv ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md future-enhancements/

# WARNING: This will require updating ARTIFACT_19 v2.1.0
# to remove Weeks 1-5 references to these systems
# This is a MAJOR change to the implementation plan
```

**If Option C (Merge into Existing Artifacts):**
```bash
# MANUAL WORK REQUIRED
# 1. Merge AI Cortex content into ARTIFACT_04_UNIFIED_AI_LAYER.md
# 2. Merge VS Code Extension into ARTIFACT_06_MASTER_CONTROL_DASHBOARD.md
# 3. Determine disposition of Metadata Governance
# 4. Update version numbers of affected artifacts to v1.2.0
# 5. Update cross-references
```

---

### Step 2: Make Decision #2 (WITHIN 24 HOURS - 1-2 hours)
**Owner:** Architecture Lead  

**Search Command:**
```bash
# Search ARTIFACT_19 for TIER X spec references
grep -i "COMPOSER_UX\|LAYOUT_EDIT_MODE\|TRI_SURFACE\|EXPORT_PIPELINE\|PROMPT_CONTROL" \
  ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md

# If no matches → these are post-MVP → move to future-enhancements
# If matches → these are MVP → keep in /docs/specifications/
```

**Expected Outcome:**
- Clear determination of MVP vs post-MVP for each spec
- Appropriate directory placement decision

---

### Step 3: Add Version Headers (DURING MIGRATION - 1 hour)
**Owner:** Documentation Lead  

**Batch update script:**
```bash
# Create a script to add YAML front matter to artifacts
for file in ARTIFACT_03 ARTIFACT_04 ARTIFACT_06 ARTIFACT_13 ARTIFACT_16; do
  # Add version header to each file
  echo "Processing ${file}.md..."
  # Manual edit or sed script to prepend YAML front matter
done
```

---

## AUTOMATED CHECKS BEFORE MIGRATION

### Pre-Flight Checklist Script
```bash
#!/bin/bash
# pre-flight-check.sh

echo "DevGuide Cockpit Migration Pre-Flight Check"
echo "==========================================="

# Check 1: Artifact count
ARTIFACT_COUNT=$(ls ARTIFACT_*.md | wc -l)
echo "✓ Artifact count: $ARTIFACT_COUNT"
if [ $ARTIFACT_COUNT -lt 19 ]; then
  echo "⚠️  WARNING: Expected at least 19 artifacts, found $ARTIFACT_COUNT"
fi

# Check 2: TIER X specs
TIERX_COUNT=$(ls *_COMPLETE_SPECIFICATION.md 2>/dev/null | wc -l)
echo "✓ TIER X specs: $TIERX_COUNT"

# Check 3: Draft specs
if [ -f "AI_CORTEX_MANAGEMENT_DASHBOARD.md" ]; then
  echo "⚠️  WARNING: AI_CORTEX_MANAGEMENT_DASHBOARD.md still in draft status"
fi
if [ -f "VSCODE_EXTENSION_ORCHESTRATION_SYSTEM.md" ]; then
  echo "⚠️  WARNING: VSCODE_EXTENSION_ORCHESTRATION_SYSTEM.md still in draft status"
fi

# Check 4: Duplicate files
DOCX_COUNT=$(ls *.docx 2>/dev/null | wc -l)
if [ $DOCX_COUNT -gt 0 ]; then
  echo "⚠️  WARNING: Found $DOCX_COUNT .docx files - should be deleted"
fi

# Check 5: Superseded files
if [ -f "DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN.md" ]; then
  echo "⚠️  WARNING: Old implementation plan v1.0 still present"
fi

echo ""
echo "Pre-flight check complete. Review warnings before migration."
```

---

## QUICK REFERENCE: RECOMMENDED ACTIONS

### ✅ DO BEFORE MIGRATION
1. Decide on draft specifications (Option A recommended)
2. Add version headers to all artifacts
3. Review TIER X specifications status
4. Delete all .docx duplicates
5. Delete superseded implementation plans
6. Create backup of entire project directory

### ✅ DO DURING MIGRATION
1. Follow exact directory structure from report
2. Copy files in order: artifacts → specs → guides → archive
3. Verify each step before proceeding
4. Run automated link checker
5. Test all cross-references

### ✅ DO AFTER MIGRATION
1. Initialize Git repository
2. Create initial commit
3. Push to GitHub
4. Verify all files present
5. Test fresh clone
6. Update team on new structure

### ❌ DON'T DO
1. Don't migrate without resolving Issue #1 (draft specs)
2. Don't skip backup step
3. Don't copy files without checking for duplicates
4. Don't push to GitHub without local verification
5. Don't delete working directory until migration verified

---

## QUICK DECISION TEMPLATE

**For Project Lead:**

> Based on review of the GitHub Migration Status Report:
> 
> **Issue #1 (Draft Specifications) - I choose:** Option [A/B/C]
> - AI Cortex: [promote to ARTIFACT_20 / move to future / merge into ARTIFACT_04]
> - VS Code Extension: [promote to ARTIFACT_21 / move to future / merge into ARTIFACT_06]
> - Metadata Governance: [keep as ARTIFACT_20 / renumber to ARTIFACT_22 / move to future]
> 
> **Issue #3 (TIER X Specs) - I determine:**
> - COMPOSER_UX: [MVP / post-MVP]
> - LAYOUT_EDIT_MODE: [MVP / post-MVP]
> - TRI_SURFACE_WORKBENCH: [MVP / post-MVP]
> - EXPORT_PIPELINE: [MVP / post-MVP]
> - PROMPT_CONTROL_HUB: [MVP / post-MVP]
> 
> **Timeline for migration:** [date]
> **Migration lead:** [name]
> **Verification lead:** [name]
> 
> Approved: [signature/date]

---

## ESTIMATED TIMELINE WITH DECISIONS

| Decision Path | Time to Migration | Total Migration Time |
|--------------|-------------------|----------------------|
| **Option A (Promote all 3)** | 2-3 hours decisions | 10-12 hours total |
| **Option B (Future enhancements)** | 1-2 hours decisions + 4-6 hours ARTIFACT_19 updates | 14-16 hours total |
| **Option C (Merge into existing)** | 4-6 hours decisions + merging | 16-20 hours total |

**RECOMMENDATION:** Option A is fastest path to clean migration.

---

**NEXT STEP:** Make Decision #1 and execute renames immediately.
