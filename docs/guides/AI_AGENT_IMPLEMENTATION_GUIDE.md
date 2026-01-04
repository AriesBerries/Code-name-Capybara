# AI Agent Implementation Guide: Structured Formatting

## YAML Front Matter • XML Tags • Progressive Disclosure

---

## QUICK DECISION MATRIX

```
┌─────────────────────────────────────────────────────────────────────┐
│ WHAT ARE YOU DOING?                                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Defining metadata about a document?  ──► YAML FRONT MATTER          │
│                                                                      │
│  Structuring sections of a prompt?    ──► XML TAGS                   │
│                                                                      │
│  Writing human-readable content?      ──► MARKDOWN                   │
│                                                                      │
│  Managing long context windows?       ──► PROGRESSIVE DISCLOSURE     │
│                                                                      │
│  All of the above?                    ──► USE ALL THREE TOGETHER     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## PART 1: YAML FRONT MATTER

### What It Is

A metadata block at the **top** of a file, enclosed by `---` delimiters.

### When to Use

✅ Page-level metadata (title, description, author)  
✅ Discovery triggers (when should an agent use this?)  
✅ Configuration flags (layout, version, platform)  
✅ Navigation control (weight, order, sidebar placement)  
✅ Skill identification for AI agents  

❌ NOT for narrative content  
❌ NOT for global settings (use config files instead)  
❌ NOT for content that belongs in the body  

### Syntax Rules

```yaml
---
# Line 1 MUST be --- with no blank lines before it
name: example-skill
description: What this does and when to use it
version: 1.0
# Use spaces only, NEVER tabs
# Close with --- before content begins
---

Content starts here after the closing delimiter.
```

### Example: Anthropic Skill Front Matter

```yaml
---
name: processing-pdfs
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

**CRITICAL REQUIREMENTS for Anthropic Skills:**

| Field | Constraints |
|-------|-------------|
| `name` | Max 64 chars, lowercase + numbers + hyphens only, NO "anthropic" or "claude" |
| `description` | Max 1024 chars, third person, must answer WHAT and WHEN |

### Example: GitHub-Style Documentation

```yaml
---
title: Setting up authentication
description: Learn how to configure authentication for your application
versions:
  fpt: '*'
  ghes: '>=3.11'
type: tutorial
topics:
  - authentication
  - security
redirect_from:
  - /old-path/auth-setup
layout: default
defaultPlatform: linux
---
```

### Good vs Bad Examples

**GOOD description:**
```yaml
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```
↳ Answers WHAT (generate commit messages) and WHEN (user asks for commit help)

**BAD description:**
```yaml
description: Helps with git stuff
```
↳ Too vague. Doesn't specify trigger conditions.

**GOOD name:**
```yaml
name: analyzing-spreadsheets
```
↳ Gerund form, specific, descriptive

**BAD name:**
```yaml
name: utils
```
↳ Too generic, no information conveyed

---

## PART 2: XML TAGS

### What They Are

Semantic markers that help Claude parse prompt structure. Claude is fine-tuned to recognize XML tags as section boundaries.

### When to Use

✅ Separating distinct sections of a prompt  
✅ Marking instructions, examples, context  
✅ Specifying output format  
✅ Enabling chain-of-thought reasoning  
✅ Wrapping user input or variable content  

❌ NOT for metadata (use YAML front matter)  
❌ NOT when the prompt is simple and flat  
❌ NOT as decoration—use purposefully  

### Key Principle: Consistency Over Canonicity

**There are no "correct" tag names.** Claude has not been trained on specific canonical tags. Choose tags that:
- Make semantic sense for your use case
- Are used consistently throughout your prompts
- Clearly communicate section purpose

### Standard Pattern: Multi-Component Prompts

```xml
<background_information>
You are helping a developer debug a Python application.
The codebase uses Flask for web services and SQLAlchemy for database access.
</background_information>

<instructions>
Analyze the error traceback provided and:
1. Identify the root cause
2. Suggest a fix
3. Explain why the fix works
</instructions>

<user_input>
{{TRACEBACK}}
</user_input>

<output_format>
Structure your response as:
- Root Cause: [one sentence]
- Fix: [code snippet]
- Explanation: [2-3 sentences]
</output_format>
```

### Chain-of-Thought Pattern

```xml
<instructions>
Solve this math problem step by step.
</instructions>

<problem>
{{PROBLEM}}
</problem>

<thinking>
Work through your reasoning here before giving the final answer.
</thinking>

<answer>
Provide only the final answer here.
</answer>
```

### Few-Shot Examples Pattern

```xml
<task>
Classify the sentiment of product reviews as positive, negative, or neutral.
</task>

<examples>
<example>
<input>This product exceeded my expectations! Highly recommend.</input>
<output>positive</output>
</example>

<example>
<input>Arrived broken. Customer service was unhelpful.</input>
<output>negative</output>
</example>

<example>
<input>It works as described. Nothing special.</input>
<output>neutral</output>
</example>
</examples>

<review_to_classify>
{{USER_REVIEW}}
</review_to_classify>
```

### Best Practices Table

| Practice | Example | Rationale |
|----------|---------|-----------|
| Mark sections clearly | `<instructions>...</instructions>` | Improves parsing accuracy |
| Nest examples | `<examples><example>...</example></examples>` | Groups related content |
| Prefill responses | `<answer>The sentiment is ` | Steers format, reduces preamble |
| Separate variables | `<user_input>{{VAR}}</user_input>` | Clear boundary for dynamic content |
| Allow escape routes | "If unsure, respond with 'uncertain'" | Reduces hallucination |

---

## PART 3: PROGRESSIVE DISCLOSURE

### What It Is

Loading information incrementally—providing only what's needed at each stage rather than pre-loading everything.

### Why It Matters

```
TOKEN EFFICIENCY CURVE
                    ▲ Quality
                    │
                    │    ╭────────╮
                    │   ╱          ╲
                    │  ╱            ╲ ← Diminishing returns
                    │ ╱              ╲
                    │╱                ╲
                    └───────────────────► Tokens
                    
Transformers create n² relationships for n tokens.
More context ≠ better results after a threshold.
```

### The Three-Level Pattern (Anthropic Skills)

```
┌─────────────────────────────────────────────────────────────────────┐
│ LEVEL 1: METADATA ONLY (Always Loaded at Startup)                   │
│ ───────────────────────────────────────────────────────────────────│
│ • name: processing-pdfs                                             │
│ • description: Extract text and tables from PDFs...                 │
│                                                                      │
│ Token cost: ~50-100 tokens per skill                                │
│ Purpose: Enables discovery without loading full content             │
├─────────────────────────────────────────────────────────────────────┤
│ LEVEL 2: SKILL BODY (Loaded When Triggered)                         │
│ ───────────────────────────────────────────────────────────────────│
│ • Full SKILL.md content                                             │
│ • Core instructions and workflows                                   │
│ • Primary examples                                                  │
│                                                                      │
│ Token cost: ~500-2000 tokens                                        │
│ Purpose: Provides working instructions for the task                 │
├─────────────────────────────────────────────────────────────────────┤
│ LEVEL 3+: REFERENCE FILES (Loaded On Demand)                        │
│ ───────────────────────────────────────────────────────────────────│
│ • forms.md - detailed form specifications                           │
│ • reference.md - API documentation                                  │
│ • examples.md - extended examples library                           │
│                                                                      │
│ Token cost: Variable (can be very large)                            │
│ Purpose: Deep detail only when specifically needed                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Metadata as Behavioral Signal

File system metadata provides signals WITHOUT consuming tokens:

| Signal | What It Tells the Agent |
|--------|------------------------|
| `/tests/test_utils.py` | This is test code, not production |
| `/docs/api/v2/endpoints.md` | API documentation, version 2 |
| `modified: 2 hours ago` | Recently changed, likely relevant |
| `size: 50KB` | Large file, may need selective reading |

**BEST PRACTICE:** Read directory structure FIRST, then selectively dive into files.

### Long-Horizon Techniques

#### Technique 1: Compaction

```
BEFORE COMPACTION:
┌──────────────────────────────────────────────────────────────────┐
│ [User message 1] [Assistant response 1] [Tool call 1]           │
│ [Tool result 1 - 5000 tokens] [User message 2] [Response 2]     │
│ [Tool call 2] [Tool result 2 - 3000 tokens] ...                 │
│ ... approaching context limit ...                                │
└──────────────────────────────────────────────────────────────────┘

AFTER COMPACTION:
┌──────────────────────────────────────────────────────────────────┐
│ [SUMMARY: Key decisions made, current state, unresolved issues] │
│ [5 most recently accessed files]                                 │
│ [Continuing from here...]                                        │
└──────────────────────────────────────────────────────────────────┘
```

**Strategy:** Maximize recall first (capture everything relevant), then iterate for precision.

#### Technique 2: Structured Note-Taking

```markdown
# AGENT_NOTES.md (Persistent Memory)

## Current Task
Implementing authentication module for Flask app

## Progress
- [x] Set up database models
- [x] Created user registration endpoint
- [ ] Implement login with JWT
- [ ] Add password reset flow

## Key Decisions
- Using bcrypt for password hashing (chosen over argon2 for compatibility)
- JWT tokens expire after 24 hours

## Blockers
- Need to decide on refresh token strategy
```

The agent reads these notes after context resets and continues work seamlessly.

#### Technique 3: Sub-Agent Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         MAIN AGENT                                   │
│                                                                      │
│  Coordinates overall task, synthesizes results                      │
│  Context: Condensed summaries only (~2000 tokens each)              │
│                                                                      │
├─────────────────┬─────────────────┬─────────────────┬───────────────┤
│                 │                 │                 │               │
│  ┌──────────┐   │  ┌──────────┐   │  ┌──────────┐   │               │
│  │ SUB-     │   │  │ SUB-     │   │  │ SUB-     │   │               │
│  │ AGENT 1  │   │  │ AGENT 2  │   │  │ AGENT 3  │   │               │
│  │          │   │  │          │   │  │          │   │               │
│  │ Legal    │   │  │ Technical│   │  │ Research │   │               │
│  │ Analysis │   │  │ Review   │   │  │          │   │               │
│  │          │   │  │          │   │  │          │   │               │
│  │ Deep     │   │  │ Deep     │   │  │ Deep     │   │               │
│  │ context  │   │  │ context  │   │  │ context  │   │               │
│  │ (10K+)   │   │  │ (10K+)   │   │  │ (10K+)   │   │               │
│  └────┬─────┘   │  └────┬─────┘   │  └────┬─────┘   │               │
│       │         │       │         │       │         │               │
│       ▼         │       ▼         │       ▼         │               │
│  [Summary      │  [Summary      │  [Summary       │               │
│   1-2K tokens] │   1-2K tokens] │   1-2K tokens]  │               │
│                 │                 │                 │               │
└─────────────────┴─────────────────┴─────────────────┴───────────────┘
```

**When to use:** Complex research, parallel exploration, specialized domains.

---

## PART 4: PUTTING IT ALL TOGETHER

### Complete Skill File Example

```yaml
---
name: generating-api-documentation
description: Generate comprehensive API documentation from code files. Use when the user asks for API docs, endpoint documentation, or wants to document their REST/GraphQL APIs.
---
```

```markdown
# API Documentation Generator

## Overview

This skill generates professional API documentation from source code files.

## When to Use

- User mentions "API documentation" or "document my endpoints"
- User provides code files containing route definitions
- User asks about REST or GraphQL documentation

## Core Workflow

<instructions>
1. Analyze provided code files for route/endpoint definitions
2. Extract HTTP methods, paths, parameters, and response types
3. Generate documentation in the specified format
4. Include example requests and responses
</instructions>

## Output Formats

<output_options>
- Markdown (default)
- OpenAPI/Swagger YAML
- HTML with styling
</output_options>

## Examples

<examples>
<example>
<input>Document the endpoints in auth.py</input>
<output>
# Authentication API

## POST /auth/login
Authenticates a user and returns a JWT token.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "string"
}
```

**Response (200):**
```json
{
  "token": "eyJ...",
  "expires_in": 86400
}
```
</output>
</example>
</examples>

## Reference Files

For detailed format specifications, read:
- `formats/openapi.md` - OpenAPI/Swagger generation rules
- `formats/markdown.md` - Markdown documentation templates
- `examples/` - Extended examples for different frameworks
```

### Decision Framework

```
┌─────────────────────────────────────────────────────────────────────┐
│ SCENARIO → RECOMMENDED APPROACH                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│ Single document with metadata         → YAML front matter            │
│                                                                      │
│ Complex prompt with sections          → XML tags                     │
│                                                                      │
│ Multi-file skill with references      → 3-level progressive         │
│                                                                      │
│ Long conversation nearing limit       → Compaction                   │
│                                                                      │
│ Multi-hour task with milestones       → Structured note-taking       │
│                                                                      │
│ Parallel research/analysis            → Sub-agent architecture       │
│                                                                      │
│ Static reference content              → Pre-load (hybrid approach)   │
│                                                                      │
│ Dynamic, changing content             → JIT retrieval (high autonomy)│
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## QUICK REFERENCE CARD

### YAML Front Matter Checklist
- [ ] Starts with `---` on line 1
- [ ] Uses spaces, not tabs
- [ ] Ends with `---` before content
- [ ] Required fields present (name, description for skills)
- [ ] Description answers WHAT and WHEN
- [ ] Name follows constraints (lowercase, no reserved words)

### XML Tags Checklist
- [ ] Tags are meaningful and consistent
- [ ] Sections are clearly delineated
- [ ] Examples are nested properly
- [ ] Output format is specified
- [ ] Variable content is wrapped

### Progressive Disclosure Checklist
- [ ] Metadata only at Level 1
- [ ] Core content at Level 2
- [ ] Deep references at Level 3+
- [ ] Directory structure read before files
- [ ] Compaction strategy defined for long tasks
- [ ] Notes/memory system for multi-session work

---

## COMMON MISTAKES TO AVOID

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Blank line before `---` | YAML parser fails | Start file with `---` immediately |
| Using tabs in YAML | Indentation breaks | Convert to spaces |
| Generic skill names | Discovery fails | Use specific, gerund-form names |
| Vague descriptions | Trigger conditions unclear | Answer WHAT and WHEN explicitly |
| Pre-loading everything | Token waste, confusion | Use progressive disclosure |
| Inconsistent XML tags | Parsing ambiguity | Standardize tag vocabulary |
| No output format spec | Verbose, unpredictable responses | Always specify format |
| Ignoring file metadata | Miss navigation signals | Read directory structure first |

---

*This guide optimized for AI agent reference. Version 1.0 - December 2025*
