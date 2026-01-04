# Tool Catalog
**Last Updated:** December 27, 2025  
**Purpose:** Master reference for all available tools across interfaces

---

## Quick Reference Matrix

| Capability | Claude Desktop | Claude Code CLI | Best Choice |
|------------|----------------|-----------------|-------------|
| **File Operations** |
| Read file | ✅ MCP | ✅ Native | CLI (faster) |
| Write file | ✅ MCP | ✅ Native | CLI (faster) |
| Edit in-place | ❌ | ✅ Native | CLI only |
| List directory | ✅ MCP | ✅ Native | Either |
| **Shell/Commands** |
| Execute bash | ❌ | ✅ Native | CLI only |
| Run scripts | ❌ | ✅ Native | CLI only |
| **Search** |
| Search file contents | ❌ | ✅ Grep | CLI only |
| Find files by pattern | ❌ | ✅ Glob | CLI only |
| **Version Control** |
| Git status/log/diff | ✅ MCP | ✅ Native+MCP | Either |
| Git add/commit | ✅ MCP | ✅ Native | Either |
| Git push | ❌ MCP | ✅ Native | CLI only |
| **GitHub API** |
| List repos | ✅ MCP | ✅ MCP | Either |
| Create issues/PRs | ✅ MCP | ✅ MCP | Either |
| **Web** |
| Fetch URL content | ✅ MCP | ✅ MCP | Either |
| **Memory** |
| Store/retrieve context | ✅ MCP | ✅ MCP | Either |
| **Reasoning** |
| Complex planning | ✅ MCP | ✅ MCP | Either |
| **Custom Commands** |
| Slash commands | ❌ | ✅ Native | CLI only |

---

## Tool Categories

### Category 1: File Operations

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| Read | CLI Native | Automatic | Read any file |
| Write | CLI Native | Automatic | Create new files |
| Edit | CLI Native | Automatic | Modify existing files |
| read_file | Desktop MCP | filesystem server | Read within allowed paths |
| write_file | Desktop MCP | filesystem server | Write within allowed paths |
| list_directory | Both | filesystem server | List folder contents |

**When to use:**
- **CLI Native** for speed and flexibility
- **Desktop MCP** when in conversation mode

---

### Category 2: Shell Execution

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| Bash | CLI Native | Automatic | Any shell command |

**Examples:**
```
npm install
git status
docker ps
winget list
```

**When to use:**
- Any command-line operation
- Build/test commands
- System queries

---

### Category 3: Search

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| Grep | CLI Native | Search file contents | Find patterns in files |
| Glob | CLI Native | Find files by pattern | Locate files (*.md, src/**/*.ts) |
| LS | CLI Native | List directory | Quick folder view |

**When to use:**
- Finding code patterns
- Locating files
- Codebase exploration

---

### Category 4: Version Control

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| git_status | MCP | git server | View working tree state |
| git_log | MCP | git server | View commit history |
| git_diff | MCP | git server | View changes |
| git_add | MCP | git server | Stage files |
| git_commit | MCP | git server | Create commit |
| Bash: git * | CLI Native | Direct commands | Full git access |

**When to use:**
- **MCP** for integrated git in Desktop conversations
- **CLI Native** for push/pull operations

---

### Category 5: GitHub API

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| list_repos | MCP | github server | See repositories |
| get_repo | MCP | github server | Repository details |
| list_issues | MCP | github server | View issues |
| create_issue | MCP | github server | Create new issue |
| list_prs | MCP | github server | View pull requests |
| create_pr | MCP | github server | Create pull request |
| gh CLI | CLI Native | Bash: gh * | Full GitHub CLI access |

**Requirements:**
- Docker Desktop running (for MCP)
- GITHUB_PERSONAL_ACCESS_TOKEN set
- gh auth status verified (for CLI)

---

### Category 6: Web Fetch

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| fetch_url | MCP | fetch server | Get web content |
| fetch_html | MCP | fetch server | Get raw HTML |

**When to use:**
- Reading documentation
- Checking API responses
- Web research

---

### Category 7: Memory

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| store | MCP | memory server | Save key-value pair |
| retrieve | MCP | memory server | Get stored value |
| list | MCP | memory server | List all memories |
| delete | MCP | memory server | Remove memory |

**When to use:**
- Persisting context across sessions
- Storing user preferences
- Tracking state

---

### Category 8: Reasoning

| Tool | Interface | Command/Method | Use Case |
|------|-----------|----------------|----------|
| think | MCP | sequential-thinking | Step-by-step reasoning |
| plan | MCP | sequential-thinking | Multi-step planning |

**When to use:**
- Complex problem decomposition
- Architecture decisions
- Multi-step task planning

---

### Category 9: Custom Commands

| Tool | Interface | Location | Use Case |
|------|-----------|----------|----------|
| /sync | CLI | .claude/commands/sync.md | Commit and push |
| /project-status | CLI | .claude/commands/project-status.md | Status report |
| /review | CLI | .claude/commands/review.md | Codebase review |
| /start | CLI | .claude/commands/start.md | Load context |
| /init | CLI | Built-in | Create CLAUDE.md |
| /doctor | CLI | Built-in | Health check |
| /ide | CLI | Built-in | IDE integration |

---

## Tool Selection Decision Tree

```
START: What do you need to do?

├─► Read/write files?
│   └─► Use Claude Code CLI (faster, more flexible)
│
├─► Execute shell commands?
│   └─► Use Claude Code CLI (only option)
│
├─► Search codebase?
│   └─► Use Claude Code CLI (Grep, Glob)
│
├─► Git operations?
│   ├─► Push to remote? → Claude Code CLI
│   └─► Status/log/commit? → Either interface
│
├─► GitHub API (issues, PRs)?
│   └─► Either interface (both have MCP)
│
├─► Have a conversation?
│   └─► Use Claude Desktop (better for dialogue)
│
├─► Complex reasoning/planning?
│   └─► Either interface (sequential-thinking MCP)
│
└─► Multi-file code changes?
    └─► Use Claude Code CLI (Edit tool)
```

---

## Prerequisites by Tool

| Tool | Requires |
|------|----------|
| filesystem MCP | Path in allowed list |
| github MCP | Docker running, PAT set |
| git MCP | uvx installed, valid repo |
| fetch MCP | uvx installed |
| memory MCP | npx available |
| sequential-thinking MCP | npx available |
| Bash | CLI mode |
| Grep/Glob | CLI mode |
| Custom commands | .claude/commands/*.md exists |

---

## Common Tool Combinations

### Implement Feature
```
1. Grep - Find related code
2. Read - Understand context
3. Edit - Make changes
4. Bash - Run tests
5. git_commit - Save work
```

### Research Task
```
1. fetch_url - Get documentation
2. think - Analyze information
3. Write - Create summary
```

### Deploy Change
```
1. Bash - Run build
2. Bash - Run tests
3. git_add + git_commit - Save
4. Bash - git push
5. create_pr - Open PR
```

---

*Reference this catalog when advising on tool selection.*
