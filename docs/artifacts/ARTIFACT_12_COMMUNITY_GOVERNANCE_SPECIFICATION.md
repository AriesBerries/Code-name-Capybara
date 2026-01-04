# DevGuide Cockpit: Community Governance Specification

**Document Type:** Governance & Security (Tier 4)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** TEMPLATE_SYSTEM_SPECIFICATION.md, SECURITY_CAPABILITY_MODEL.md

---

## Executive Summary

This artifact specifies the **community contribution and governance model** for DevGuide Cockpit templates. It defines:
- Template contribution workflow (fork → PR → review → approval)
- Quality gates and review criteria  
- Baseline vs community guardrails
- Featured template curation process
- Community moderation policies

**Key Principle:** Balance innovation (community contributions) with safety (certification process).

---

## Contribution Workflow

### Step-by-Step Process

```
[1] Developer forks base template
    ↓
[2] Customizes dashboards, modules, guardrails
    ↓
[3] Runs local validation:
    $ devguide-cli template validate ./my-template
    ↓
[4] Submits to community registry:
    $ devguide-cli template publish ./my-template
    ↓
[5] Automated checks:
    ✓ template.yaml syntax valid
    ✓ All Elm modules compile
    ✓ No malicious code patterns
    ✓ Capability declarations present
    ✓ Security scan passed
    ↓
[6] Community review (optional, for Tier 1)
    ↓
[7] Certification decision:
    - Tier 1 Certified (manual approval)
    - Tier 2 Community (auto-approved, use at own risk)
    - Tier 3 Experimental (flagged for issues)
    ↓
[8] Published to marketplace
```

---

## Certification Tiers

### Tier 1: Certified Templates

**Requirements:**
- Full code review by DevGuide team
- Security audit (penetration testing)
- Documentation completeness
- Test coverage ≥ 80%
- No critical vulnerabilities
- WCAG 2.1 Level AA accessibility

**Review Process:**
- Manual code review (2 team members)
- Security audit (external firm for high-impact templates)
- User testing (beta testers try template)
- Documentation review

**Trust Level:** High - safe for production use

**Examples:**
- Vibe Coding Template
- Data Science Template
- Note Taking Template

**Badge:**
```
┌────────────────────────┐
│  ✓ TIER 1 CERTIFIED    │
│  Reviewed by DevGuide  │
└────────────────────────┘
```

---

### Tier 2: Community Templates

**Requirements:**
- Automated checks pass (syntax, compilation, security scan)
- No critical security vulnerabilities
- Basic documentation present
- Creator has verified email

**Review Process:**
- Automated only (no manual review)
- Community upvotes/downvotes
- User reports handled reactively

**Trust Level:** Medium - use at own risk

**Examples:**
- E-commerce Template (community-built)
- DevOps Dashboard (community-built)

**Badge:**
```
┌────────────────────────┐
│  ⚠️  COMMUNITY         │
│  Use at own risk       │
└────────────────────────┘
```

---

### Tier 3: Experimental Templates

**Requirements:**
- Minimal validation (syntax only)
- May contain known issues
- Explicitly marked as experimental

**Review Process:**
- None (trust developer)

**Trust Level:** Low - expect bugs

**Usage:** Testing, education, proof-of-concept

**Badge:**
```
┌────────────────────────┐
│  ⚠️  EXPERIMENTAL      │
│  Not recommended       │
└────────────────────────┘
```

---

## Quality Gates

### Automated Checks (All Tiers)

**1. Template Structure:**
```bash
✓ template.yaml exists
✓ README.md exists
✓ dashboards/ directory exists
✓ All referenced Elm modules present
✓ No broken file references
```

**2. Code Compilation:**
```bash
✓ All Elm modules compile without errors
✓ All Go modules build successfully
✓ All Rust crates pass `cargo check`
```

**3. Security Scan:**
```bash
✓ No hardcoded secrets
✓ No SQL injection patterns
✓ No XSS vulnerabilities
✓ No dangerous JavaScript (eval, Function constructor)
✓ All external dependencies whitelisted
```

**4. Capability Declaration:**
```bash
✓ All capabilities declared in CAPABILITIES.yaml
✓ No undeclared network calls
✓ No undeclared file system access
```

---

### Manual Review Checklist (Tier 1 Only)

**Code Quality:**
- [ ] Follows Elm style guide
- [ ] No code duplication
- [ ] Meaningful variable names
- [ ] Comments for complex logic
- [ ] Test coverage ≥ 80%

**Security:**
- [ ] Input validation for all user inputs
- [ ] Output encoding to prevent XSS
- [ ] CSRF protection where applicable
- [ ] Rate limiting on API calls
- [ ] Audit log for sensitive operations

**Documentation:**
- [ ] README explains purpose and usage
- [ ] All dashboards documented
- [ ] Configuration options explained
- [ ] Examples provided
- [ ] Troubleshooting section

**Accessibility:**
- [ ] Keyboard navigation works
- [ ] Screen reader compatible
- [ ] Color contrast meets WCAG AA
- [ ] Focus indicators visible
- [ ] Error messages clear

**Performance:**
- [ ] Dashboard load time < 2s
- [ ] No memory leaks
- [ ] Efficient rendering (no jank)
- [ ] Token usage within budget

---

## Review SLA

| Tier | Target Review Time | Max Review Time |
|------|-------------------|-----------------|
| Tier 1 | 7 days | 14 days |
| Tier 2 | Immediate (automated) | N/A |
| Tier 3 | None | N/A |

**Reviewer Workload Cap:** Max 5 Tier 1 reviews per week per reviewer

---

## Baseline vs Community Guardrails

### Baseline Guardrails (System-Wide, Non-Negotiable)

**Enforced by The Bin, cannot be overridden:**

```yaml
baseline_guardrails:
  - id: "no_plaintext_secrets"
    type: "security"
    enforcement: "hard"
    rationale: "Secrets must be encrypted or environment variables"
    
  - id: "max_token_budget"
    type: "cost"
    enforcement: "hard"
    value: 100000  # tokens per hour
    rationale: "Prevent runaway AI costs"
    
  - id: "audit_log_required"
    type: "compliance"
    enforcement: "hard"
    rationale: "All changes must be logged for audit trail"
    
  - id: "wcag_aa_required"
    type: "accessibility"
    enforcement: "hard"
    rationale: "Accessibility is mandatory, not optional"
```

---

### Community Guardrails (Template-Specific, Configurable)

**Recommended by template author, user can override with rationale:**

```yaml
community_guardrails:
  - id: "test_coverage_min"
    type: "quality"
    enforcement: "soft"
    value: 80
    rationale: "High test coverage prevents regressions"
    override_allowed: true
    override_requires_rationale: true
    
  - id: "max_service_count"
    type: "architecture"
    enforcement: "soft"
    value: 3
    rationale: "Simplicity for Vibe Coding template"
    override_allowed: true
    override_requires_rationale: true
```

**Override Example:**
```elm
-- User overrides community guardrail
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        OverrideGuardrail guardrailId rationale ->
            if String.length rationale < 100 then
                ( model, showError "Rationale must be at least 100 characters" )
            else
                ( { model | overrides = Dict.insert guardrailId rationale model.overrides }
                , reportGuardrailOverride guardrailId rationale
                )
```

---

## Featured Template Curation

### Selection Criteria

**Metrics:**
- Downloads > 1000
- Rating ≥ 4.5/5
- Active maintenance (updated within 6 months)
- Community feedback positive (>80% upvotes)
- No critical issues reported

**Manual Review:**
- DevGuide team tests template
- Verifies quality and usefulness
- Checks documentation accuracy

**Featured Badge:**
```
┌────────────────────────┐
│  ⭐ FEATURED           │
│  Recommended by team   │
└────────────────────────┘
```

**Featured Templates Appear:**
- Top of marketplace search results
- Homepage carousel
- Email newsletters
- Social media highlights

---

## Community Moderation

### Reporting System

**Users can report templates for:**
- Security vulnerabilities
- Malicious code
- Copyright violations
- Misleading descriptions
- Poor quality

**Report Flow:**
```
User clicks "Report Template"
    ↓
Selects reason from dropdown
    ↓
Provides details (optional)
    ↓
Report submitted to moderation queue
    ↓
Moderator reviews within 48 hours
    ↓
Actions:
    - Dismiss (not a violation)
    - Request changes from author
    - Downgrade tier (e.g., Tier 1 → Tier 2)
    - Remove from marketplace (critical violations)
```

---

### Moderation Actions

| Violation | First Offense | Second Offense | Third Offense |
|-----------|---------------|----------------|---------------|
| **Minor Bug** | Flag for author | Downgrade tier | N/A |
| **Security Issue** | Immediate takedown | Ban author 30 days | Permanent ban |
| **Malicious Code** | Permanent ban + legal action | N/A | N/A |
| **Copyright** | Takedown + warning | Ban author | N/A |
| **Spam** | Warning | 30-day ban | Permanent ban |

---

## Version Control & Updates

### Template Versioning

**Semantic Versioning:** MAJOR.MINOR.PATCH

**Upgrade Notifications:**
- Email to all template users
- In-dashboard notification
- Changelog displayed

**Breaking Changes:**
- Must increment MAJOR version
- Requires migration wizard
- Users must explicitly opt-in

---

## Incentive System (Post-MVP)

**Recognition:**
- Top contributor badge
- Featured author profile
- Invitation to advisory board

**Potential Monetization (Future):**
- Template marketplace with paid templates
- Revenue sharing (70% author, 30% platform)
- Premium support subscriptions

---

## Open Questions

1. Should Tier 2 templates be time-limited (e.g., expire after 1 year)? (Suggest: No, but flag unmaintained)
2. Should template authors be required to respond to issues? (Suggest: Yes, for Tier 1)
3. Maximum number of templates per author? (Suggest: 10 for Tier 1, unlimited for Tier 2/3)

---

**End of COMMUNITY_GOVERNANCE_SPECIFICATION.md**
