# Template Library: Structured Formatting for AI Agents

## Ready-to-Use Templates for YAML, XML, and Progressive Disclosure

---

## TEMPLATE 1: ANTHROPIC SKILL (BASIC)

```yaml
---
name: {{skill-name-lowercase-hyphens}}
description: {{What this skill does}}. Use when {{trigger conditions}}.
---

# {{Skill Title}}

## Overview

{{Brief description of the skill's purpose}}

## When to Use This Skill

- {{Trigger phrase 1}}
- {{Trigger phrase 2}}
- {{Trigger phrase 3}}

## Core Workflow

1. {{Step 1}}
2. {{Step 2}}
3. {{Step 3}}

## Examples

### Example 1: {{Use Case Name}}

**Input:** {{User request}}

**Output:** {{Expected result}}

## Notes

- {{Important consideration 1}}
- {{Important consideration 2}}
```

**Usage:** Copy and replace `{{placeholders}}` with your content.

---

## TEMPLATE 2: ANTHROPIC SKILL (ADVANCED WITH REFERENCES)

```yaml
---
name: {{skill-name}}
description: {{Description answering WHAT and WHEN}}. Use when {{specific triggers}}.
---

# {{Skill Title}}

## Overview

{{2-3 sentence description}}

## When to Use This Skill

Use this skill when:
- {{Condition 1}}
- {{Condition 2}}
- {{Condition 3}}

Do NOT use this skill when:
- {{Anti-pattern 1}}
- {{Anti-pattern 2}}

## Core Principles

1. **{{Principle 1 Name}}**: {{Explanation}}
2. **{{Principle 2 Name}}**: {{Explanation}}
3. **{{Principle 3 Name}}**: {{Explanation}}

## Workflow

<workflow>
1. {{Step 1 with tool annotation if applicable}}
2. {{Step 2}}
3. {{Step 3}}
4. {{Verification step}}
</workflow>

## Output Format

<output_format>
{{Specify the expected output structure}}
</output_format>

## Examples

<examples>
<example>
<input>{{User request 1}}</input>
<o>{{Expected output 1}}</o>
</example>

<example>
<input>{{User request 2}}</input>
<o>{{Expected output 2}}</o>
</example>
</examples>

## Reference Files

For detailed specifications, read:
- `reference/{{file1}}.md` - {{Description}}
- `reference/{{file2}}.md` - {{Description}}

## Related Skills

- `{{related-skill-1}}` - {{When to use instead}}
- `{{related-skill-2}}` - {{When to use in combination}}
```

---

## TEMPLATE 3: DOCUMENTATION PAGE (GITHUB STYLE)

```yaml
---
title: {{Page Title}}
shortTitle: {{Breadcrumb Title}}
description: {{SEO description, 75-300 characters}}
versions:
  fpt: '*'
  ghes: '>=3.11'
type: {{overview|quickstart|tutorial|howto|reference}}
topics:
  - {{topic1}}
  - {{topic2}}
redirect_from:
  - {{/old/path/if/applicable}}
layout: default
defaultPlatform: {{linux|mac|windows}}
---

# {{Page Title}}

{{Introduction paragraph}}

## Prerequisites

- {{Prerequisite 1}}
- {{Prerequisite 2}}

## {{Section 1}}

{{Content}}

## {{Section 2}}

{{Content}}

## Next steps

- {{Link to related content 1}}
- {{Link to related content 2}}
```

---

## TEMPLATE 4: DOCUMENTATION PAGE (MICROSOFT STYLE)

```yaml
---
title: {{Browser Tab Title, max 70 chars}}
description: {{Search description, 75-300 chars for SEO}}
author: {{github-username}}
ms.author: {{microsoft-alias}}
ms.date: {{YYYY-MM-DD}}
ms.topic: {{conceptual|quickstart|tutorial|how-to|reference}}
ms.service: {{service-name}}
---

# {{Article Title}}

{{Introduction - what this article covers and who it's for}}

## Prerequisites

- {{Requirement 1}}
- {{Requirement 2}}

## {{Main Section 1}}

{{Content}}

### {{Subsection}}

{{Content}}

## {{Main Section 2}}

{{Content}}

## Related content

- [{{Related article 1}}]({{link}})
- [{{Related article 2}}]({{link}})
```

---

## TEMPLATE 5: DOCUMENTATION PAGE (GRAFANA STYLE)

```yaml
---
title: {{Page Title}}
menuTitle: {{Shorter Nav Title}}
description: {{SEO description, 150+ characters for search and social}}
aliases:
  - ./{{old-relative-path}}/ # {{Absolute URL this resolves to}}
canonical: {{https://full-canonical-url-if-multi-hosted}}
weight: {{100|200|300}} # Sidebar order, increments of 100
labels:
  products:
    - oss
    - cloud
    - enterprise
  stage: {{experimental|private-preview|public-preview|general-availability}}
review_date: {{YYYY-MM-DD}}
cascade:
  labels:
    products:
      - {{inherited-product-label}}
---

# {{Page Title}}

{{Introduction}}

## Before you begin

- {{Prerequisite 1}}
- {{Prerequisite 2}}

## {{Section 1}}

{{Content}}

## {{Section 2}}

{{Content}}

## Learn more

- [{{Related topic 1}}]({{link}})
- [{{Related topic 2}}]({{link}})
```

---

## TEMPLATE 6: XML STRUCTURED PROMPT (BASIC)

```xml
<system>
You are {{role description}}.
</system>

<instructions>
{{Clear, direct task instructions}}

Requirements:
1. {{Requirement 1}}
2. {{Requirement 2}}
3. {{Requirement 3}}
</instructions>

<context>
{{Background information the model needs}}
</context>

<input>
{{USER_INPUT_VARIABLE}}
</input>

<output_format>
{{Specify exact output structure}}
</output_format>
```

---

## TEMPLATE 7: XML STRUCTURED PROMPT (WITH EXAMPLES)

```xml
<system>
You are {{role}}. Your task is to {{primary function}}.
</system>

<instructions>
{{Task description}}

Follow these rules:
- {{Rule 1}}
- {{Rule 2}}
- {{Rule 3}}
</instructions>

<examples>
<example>
<input>{{Example input 1}}</input>
<reasoning>{{Optional: show thinking process}}</reasoning>
<output>{{Example output 1}}</output>
</example>

<example>
<input>{{Example input 2}}</input>
<reasoning>{{Optional: show thinking process}}</reasoning>
<output>{{Example output 2}}</output>
</example>

<example>
<input>{{Example input 3}}</input>
<reasoning>{{Optional: show thinking process}}</reasoning>
<output>{{Example output 3}}</output>
</example>
</examples>

<input>
{{USER_INPUT}}
</input>

<output_format>
Respond using this exact structure:
{{Format specification}}
</output_format>
```

---

## TEMPLATE 8: XML CHAIN-OF-THOUGHT PROMPT

```xml
<system>
You are a {{role}} that thinks through problems step by step.
</system>

<instructions>
{{Task description}}

Before providing your final answer:
1. Analyze the problem in your thinking section
2. Consider alternative approaches
3. Verify your reasoning
4. Then provide your final answer
</instructions>

<problem>
{{PROBLEM_DESCRIPTION}}
</problem>

<thinking>
Work through your reasoning here. Consider:
- What information is given?
- What is being asked?
- What approaches could work?
- Which approach is best and why?
</thinking>

<answer>
{{Provide final answer here}}
</answer>
```

---

## TEMPLATE 9: XML DATA EXTRACTION PROMPT

```xml
<instructions>
Extract the following information from the provided text.
If a field is not present, use "N/A".
</instructions>

<fields_to_extract>
- {{Field 1}}: {{description of what to look for}}
- {{Field 2}}: {{description of what to look for}}
- {{Field 3}}: {{description of what to look for}}
</fields_to_extract>

<source_text>
{{TEXT_TO_ANALYZE}}
</source_text>

<output_format>
Return results as JSON:
{
  "{{field1}}": "extracted value",
  "{{field2}}": "extracted value",
  "{{field3}}": "extracted value"
}
</output_format>
```

---

## TEMPLATE 10: PROGRESSIVE DISCLOSURE SKILL STRUCTURE

```
{{skill-name}}/
├── SKILL.md              # Level 2: Loaded when triggered
├── reference/
│   ├── api.md            # Level 3: Loaded on demand
│   ├── formats.md        # Level 3: Loaded on demand
│   └── advanced.md       # Level 3: Loaded on demand
└── examples/
    ├── basic.md          # Level 3: Loaded on demand
    └── advanced.md       # Level 3: Loaded on demand
```

### SKILL.md Structure

```yaml
---
name: {{skill-name}}
description: {{Level 1 content - always loaded for discovery}}
---

# {{Skill Name}}

## Overview
{{Essential information needed to use the skill}}

## When to Use
{{Trigger conditions}}

## Core Workflow
{{Primary workflow steps}}

## Quick Reference
{{Most common use cases}}

## Detailed References
For advanced usage, read:
- `reference/api.md` - Detailed API specifications
- `reference/formats.md` - Output format options
- `examples/advanced.md` - Complex use cases
```

---

## TEMPLATE 11: AGENT NOTES (PERSISTENT MEMORY)

```markdown
# Agent Working Notes

## Session: {{YYYY-MM-DD HH:MM}}

### Current Task
{{Brief description of what we're working on}}

### Progress
- [x] {{Completed step 1}}
- [x] {{Completed step 2}}
- [ ] {{Pending step 3}}
- [ ] {{Pending step 4}}

### Key Decisions Made
1. **{{Decision 1}}**: {{Rationale}}
2. **{{Decision 2}}**: {{Rationale}}

### Files Modified
- `{{path/to/file1}}` - {{What was changed}}
- `{{path/to/file2}}` - {{What was changed}}

### Open Questions
- {{Question 1}}?
- {{Question 2}}?

### Next Steps
1. {{Next action 1}}
2. {{Next action 2}}

### Context to Preserve
{{Critical information that must survive context resets}}
```

---

## TEMPLATE 12: COMPACTION SUMMARY

```markdown
# Context Summary

## Task Overview
{{Original user request and high-level goal}}

## Current State
{{Where we are in the task}}

## Key Decisions
{{Important decisions made and their rationale}}

## Architecture/Approach
{{Technical approach being used}}

## Unresolved Issues
{{Problems still to be solved}}

## Recent Files (Last 5 Accessed)
1. `{{file1}}` - {{brief description of content}}
2. `{{file2}}` - {{brief description of content}}
3. `{{file3}}` - {{brief description of content}}
4. `{{file4}}` - {{brief description of content}}
5. `{{file5}}` - {{brief description of content}}

## Immediate Next Steps
1. {{Next action}}
2. {{Following action}}
```

---

## TEMPLATE 13: SUB-AGENT TASK HANDOFF

```xml
<sub_agent_task>
<role>{{Specialized role for this sub-agent}}</role>

<objective>
{{Specific task to complete}}
</objective>

<context>
{{Relevant background information}}
</context>

<constraints>
- {{Constraint 1}}
- {{Constraint 2}}
- {{Token budget: approximately {{N}} tokens for response}}
</constraints>

<output_requirements>
Return a summary that includes:
1. {{Required output element 1}}
2. {{Required output element 2}}
3. Key findings or recommendations

Maximum length: {{N}} tokens
</output_requirements>
</sub_agent_task>
```

---

## USAGE NOTES

### Placeholder Legend
- `{{text}}` - Replace with your content
- `{{USER_INPUT}}` - Will be replaced at runtime with user's input
- `{{YYYY-MM-DD}}` - Date format
- `{{N}}` - Numeric value

### Template Selection Guide

| If you need... | Use Template |
|----------------|--------------|
| Basic skill with triggers | 1: Basic Skill |
| Complex skill with reference files | 2: Advanced Skill |
| GitHub-style documentation | 3: GitHub Style |
| Microsoft Learn documentation | 4: Microsoft Style |
| Grafana-style with versioning | 5: Grafana Style |
| Simple structured prompt | 6: Basic XML |
| Prompt with examples | 7: XML with Examples |
| Problem-solving prompt | 8: Chain-of-Thought |
| Data extraction | 9: Data Extraction |
| Multi-file skill organization | 10: Skill Structure |
| Persistent memory | 11: Agent Notes |
| Context compression | 12: Compaction |
| Sub-agent coordination | 13: Sub-Agent Handoff |

---

*Templates v1.0 - December 2025*
