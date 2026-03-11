# LLM CLI Command Reference
### Claude Code · Gemini CLI — Senior engineer quick-scan format — March 2026

> **Legend:** `CC` = Claude Code · `GEM` = Gemini CLI · `BOTH` = identical or near-identical

---

## §1 — Commands Common to Both Tools

Both tools converge on the same conceptual primitives. Syntax differs; intent is identical.

---

### 1.1 Session Lifecycle

| Concept | Claude Code | Gemini CLI |
|---|---|---|
| Start interactive REPL | `claude` | `gemini` |
| Start with initial prompt | `claude "query"` | `gemini -p "query"` |
| Continue last session | `claude -c` | `gemini` (auto-resumes) |
| Resume by session ID / tag | `claude -r <id> "query"` | `gemini /resume <tag>` |
| Exit REPL | `/exit` or `Ctrl-D` | `/quit` or `Ctrl-D` |
| Clear context window | `/clear` | `/clear` then `/compress` |
| Compact / summarise context | `/compact [instructions]` | `/compress` |

---

### 1.2 File & Directory Context

| Concept | Claude Code | Gemini CLI |
|---|---|---|
| Reference file inline | `@./path/to/file.ts` | `@./path/to/file.ts` |
| Add extra workspace dir | `claude --add-dir ../lib` | `/directory add ../lib` |
| List active context dirs | `/context` | `/directory list` |
| Pipe content in | `cat foo.py \| claude -p "explain"` | `cat foo.py \| gemini -p "explain"` |

---

### 1.3 Shell Passthrough

```
# Both tools use ! prefix to execute shell commands directly
!npm test
!git diff HEAD~1
!grep -r "pattern" src/ --include="*.ts" -l
```

Shell output is injected into the AI's context for the current turn.

---

### 1.4 Configuration & Health

| Concept | Claude Code | Gemini CLI |
|---|---|---|
| Show help | `/help` | `/help` |
| Show version | `claude --version` | `gemini -v` or `/version` |
| Open settings editor | `/config` | `/settings` |
| Health / diagnostics | `/doctor` | `/stats`, `/version` |
| Token / cost stats | `/cos` | `/stats` |
| List available tools | `/tools` | `/tools list` |
| Configure MCP server | Edit `.claude/settings.json` | `gemini mcp add` |
| Reference MCP tool inline | `@server_name query` | `@server_name query` |

---

### 1.5 Output & Scripting

| Concept | Claude Code | Gemini CLI |
|---|---|---|
| Print mode (non-interactive) | `claude -p "query"` | `gemini -p "query"` |
| JSON structured output | `claude -p --output-format json "query"` | `gemini -p --output-format json "query"` |
| Save output to file | `/save output.md` | `/save output.md` |
| Copy last output to clipboard | (OS clipboard) | `/copy` |

```bash
# Automation / CI pattern — both tools
git log --oneline | claude -p "summarise these commits" --output-format json
cat error.log  | gemini -p "find the root cause" --output-format json
```

---

### 1.6 Context Files (Project Memory)

Both tools auto-load a context file from the repo root on session start.

```
CLAUDE.md    # Claude Code reads this
GEMINI.md    # Gemini CLI reads this
```

**Recommended structure:**

```markdown
## Architecture
Hexagonal / Ports and Adapters. Split repo (api/ + ui/).

## Stack
Python 3.12 + FastAPI + Google ADK. TypeScript 5 + React.

## Conventions
All new code in src/. Tests mirror src/ under tests/.
Branch naming: feat/<ticket>-<slug>

## Before writing any code
1. Read relevant files.
2. State your plan.
3. Wait for confirmation.
```

> These files are the single highest-leverage thing you can add to either tool. Commit them.

---

### 1.7 Custom Slash Commands

| Concept | Claude Code | Gemini CLI |
|---|---|---|
| Project-scoped storage | `.claude/commands/<name>.md` | `.gemini/commands/<name>.toml` |
| Global (user-scoped) | `~/.claude/commands/` | `~/.gemini/commands/` |
| Invoke | `/name` or `/name arg` | `/name` or `/name arg` |
| Format | Markdown (freeform prompt) | TOML (`prompt = "..."`) |
| Namespacing | Flat by default | Sub-dirs → `/ns:cmd` |
| MCP prompts as commands | Via `/mcp` | Natively, auto-exposed as `/cmd` |
| Skills (CC) / Extensions (GEM) | `.claude/skills/<name>/SKILL.md` | `.gemini/extensions/<name>/` |

---

## §2 — Claude Code Only

---

### 2.1 Session Flags

| Flag | Example | Notes |
|---|---|---|
| `--model` / `-m` | `claude --model claude-opus-4-6 "query"` | Override default model |
| `--max-turns` | `claude -p --max-turns 5 "query"` | Limit agentic loop depth |
| `--verbose` | `claude --verbose "query"` | Show tool calls + reasoning |
| `--output-format` | `claude -p --output-format json\|text\|stream-json` | Control output type |
| `--add-dir` | `claude --add-dir ../shared ../lib` | Add extra workspace dirs |
| `--dangerously-skip-permissions` | `claude --dangerously-skip-permissions` | ⚠ Headless / CI only |

---

### 2.2 Tool Permissions

```bash
# Whitelist specific tool patterns
claude --allowedTools "Bash(git:*)" "Write" "Read"

# Blacklist destructive operations
claude --disallowedTools "Bash(rm:*)" "Bash(sudo:*)"

# Custom permission gating via MCP
claude -p --permission-prompt-tool mcp_auth_tool "query"

# Multi-directory with restricted tools
claude --add-dir ../frontend ../backend \
       --allowedTools "Bash(git:*)" "Write" "Read" \
       --disallowedTools "Bash(rm:*)"
```

---

### 2.3 System Prompt Control

| Flag | Behaviour | When to use |
|---|---|---|
| `--append-system-prompt "…"` | Adds to defaults | **Preferred** — keeps built-in capabilities |
| `--append-system-prompt-file ./p.md` | File-based append | Team consistency, version-controlled prompts |
| `--system-prompt "…"` | Replaces ALL defaults | Full control needed |
| `--system-prompt-file ./p.md` | Full replacement from file | Complete custom persona |

> `--system-prompt` and `--system-prompt-file` are mutually exclusive.

---

### 2.4 REPL Slash Commands

| Command | What it does |
|---|---|
| `/init` | Generates `CLAUDE.md` by scanning the current repo |
| `/plan` | Read-only analysis mode — no code written until you confirm |
| `/review` | Code review of current changes via review subagent |
| `/pr-comments` | Reads open GitHub PR review comments and addresses them |
| `/compact [instructions]` | Summarise context; optionally specify what to preserve |
| `/ide` | Manage VS Code / JetBrains integrations |
| `/mcp` | List and manage MCP server connections |
| `/cos` | Show token cost and duration of current session |
| `/insights` | Analyses usage history, generates HTML report |
| `/checkpoint save <tag>` | Save named conversation state |
| `/checkpoint resume <tag>` | Restore named state |

---

### 2.5 Subagents

Defined in `.claude/agents/<name>.md` or inline via `--agents`. Claude invokes them automatically based on description matching.

```markdown
# .claude/agents/reviewer.md
---
name: reviewer
description: Use for thorough code reviews after any code change.
model: sonnet
tools: [Read, Grep, Glob, Bash]
---
You are a senior code reviewer. Focus on security,
performance, and architectural consistency.
```

```bash
# Inline via CLI flag
claude --agents '{
  "reviewer": {
    "description": "Expert code reviewer. Use after code changes.",
    "prompt": "You are a senior reviewer. Focus on security and performance.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

---

### 2.6 Git Worktree Parallel Sessions

```bash
# Create worktrees for parallel agent work — agents never touch each other's branches
git worktree add ../feat-auth     feat/auth-uuid-abc123
git worktree add ../feat-billing  feat/billing-uuid-def456

# Run independent Claude sessions in each
cd ../feat-auth    && claude "implement JWT auth" &
cd ../feat-billing && claude "implement Stripe integration" &
```

> Combined with UUID-suffixed branch isolation: each agent works its own branch and communicates via PR metadata — never directly overwrites another agent's branch.

---

### 2.7 Hooks (CI / Post-Write Automation)

Hooks fire shell commands on lifecycle events, regardless of model behaviour.

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write",
      "hooks": [{"type": "command", "command": "npm run lint -- --fix"}]
    }],
    "Stop": [{
      "hooks": [{"type": "command", "command": "git add -A && git status"}]
    }]
  }
}
```

---

## §3 — Gemini CLI Only

---

### 3.1 Launch Flags

| Flag | Example | Notes |
|---|---|---|
| `-m` / `--model` | `gemini -m gemini-2.5-flash "query"` | Flash = fast/cheap; Pro = default |
| `-p` / `--prompt` | `gemini -p "summarise this repo"` | Non-interactive, exits after |
| `--output-format` | `gemini -p "query" --output-format json` | `json` or `text` |
| `--include-directories` | `gemini --include-directories ../lib ../shared` | Extra dirs at launch |
| `--sandbox` | `gemini --sandbox` | Restrictive execution profile |
| `--debug` | `gemini --debug` | Verbose tool + request logging |
| `--checkpointing` | `gemini --checkpointing` | Auto-checkpoint before file writes |

---

### 3.2 Built-in REPL Commands

| Command | What it does |
|---|---|
| `/compress` | Summarise and replace context in one step |
| `/copy` | Copy last AI output to clipboard |
| `/chat save <tag>` | Named conversation checkpoint |
| `/chat resume <tag>` | Restore named checkpoint |
| `/checkpoint` | Auto-checkpoint management for file writes |
| `/directory add <path>` | Add directory mid-session |
| `/directory list` | Show active workspace directories |
| `/editor` | Switch editor (vim / emacs / nano etc.) |
| `/vim` | Toggle vim keybinding mode |
| `/theme` | Switch visual theme interactively |
| `/stats` | Token usage + cached token savings |
| `/tools list` | Show available tools |
| `/tools describe` | Full tool descriptions |
| `/commands reload` | Hot-reload `.toml` command files without restart |
| `/auth` | Switch authentication method |
| `/quit` | Exit session |

---

### 3.3 Custom Command TOML Format

```toml
# .gemini/commands/review.toml  →  invoked as /review
description = "Review code for a given concern"
prompt = """
You are a senior engineer. Review the following code for {{args}}.
Focus on: security, performance, maintainability.
Reference our standards: @{docs/engineering-standards.md}
"""
```

```toml
# .gemini/commands/git/commit.toml  →  invoked as /git:commit
description = "Generate a conventional commit message"
prompt = """
Analyse the staged diff and write a conventional commit message.
Format: <type>(<scope>): <short description>
Types: feat | fix | refactor | test | docs | chore
"""
```

> `{{args}}` receives everything typed after the command name.  
> `@{path}` injects file content at prompt time.  
> Sub-directories create namespaced commands: `git/commit.toml` → `/git:commit`.

---

### 3.4 MCP Server Management

```bash
# Manage via CLI
gemini mcp add
gemini mcp list
gemini mcp remove <name>
```

```json
// .gemini/settings.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "$GITHUB_TOKEN" }
    },
    "db": {
      "command": "node",
      "args": ["mcp/db-server.js"],
      "trust": false,
      "includeTools": ["query", "schema"],
      "excludeTools": ["drop_table"]
    }
  }
}
```

> MCP prompts are auto-exposed as slash commands. A prompt named `review` becomes `/review` in the session.

---

### 3.5 Extensions

```json
// .gemini/extensions/project-tools/gemini-extension.json
{
  "name": "project-tools",
  "version": "1.0.0",
  "mcpServers": {
    "db": { "command": "node mcp/db-server.js" }
  },
  "contextFileName": "GEMINI.md",
  "excludeTools": ["run_shell_command"]
}
```

---

## §4 — The Canonical 25

Every LLM developer tool eventually converges on these primitives.

| # | Concept | Claude Code | Gemini CLI | Notes |
|---|---|---|---|---|
| 1 | Start session | `claude` | `gemini` | Entry point. Both auto-detect cwd. |
| 2 | Start with prompt | `claude "…"` | `gemini -p "…"` | One-shot execution. |
| 3 | Continue session | `claude -c` | `gemini` / `/resume` | Restore prior context. |
| 4 | Print mode | `claude -p "…"` | `gemini -p "…"` | Non-interactive, pipe-friendly. |
| 5 | JSON output | `--output-format json` | `--output-format json` | Automation / CI. |
| 6 | Reference file | `@./src/api.ts` | `@./src/api.ts` | Inline file injection. |
| 7 | Run shell command | `!npm test` | `!npm test` | Passthrough to shell. |
| 8 | Add workspace dir | `--add-dir ../lib` | `/directory add ../lib` | Multi-repo context. |
| 9 | Pipe to AI | `cat log \| claude -p "…"` | `cat log \| gemini -p "…"` | stdin passthrough. |
| 10 | Compact context | `/compact` | `/compress` | Save tokens mid-session. |
| 11 | Save checkpoint | `/checkpoint save <tag>` | `/chat save <tag>` | Branch conversation state. |
| 12 | Resume checkpoint | `/checkpoint resume <tag>` | `/chat resume <tag>` | Restore named state. |
| 13 | Clear context | `/clear` | `/clear` + `/compress` | Fresh context window. |
| 14 | Project memory file | `CLAUDE.md` | `GEMINI.md` | Persistent codebase context. |
| 15 | Init context file | `/init` | Write `GEMINI.md` manually | Bootstrap project awareness. |
| 16 | Custom command | `.claude/commands/foo.md` | `.gemini/commands/foo.toml` | Reusable prompt macros. |
| 17 | Plan before code | `/plan` | `/plan` custom cmd or `GEMINI.md` | Read-only analysis first. |
| 18 | Model selection | `--model claude-opus-4-6` | `-m gemini-2.5-flash` | Balance quality vs cost. |
| 19 | Config / settings | `/config` | `/settings` | Interactive config editor. |
| 20 | Show help | `/help` | `/help` | List all commands. |
| 21 | Token / cost stats | `/cos` | `/stats` | Session usage summary. |
| 22 | MCP server | `--mcp` / `.claude/settings.json` | `gemini mcp add` / `settings.json` | Extend with external tools. |
| 23 | Tool permissions | `--allowedTools` `--disallowedTools` | `sandbox` / `settings.json` | Control blast radius. |
| 24 | Subagent / agent | `.claude/agents/` or `--agents` | Via MCP / extensions | Delegate to specialised AI. |
| 25 | Non-interactive hook | `.claude/settings.json` hooks | `.gemini/settings` automation | CI / post-write linting. |
