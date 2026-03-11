# 🤖 LLM CLI Playbook

**The missing operations manual for Claude Code and Gemini CLI.**

A senior-engineer-grade command reference and workflow playbook for developers who live in the terminal. Built for quick scanning during real work — not for reading once and forgetting.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
![Last Updated](https://img.shields.io/badge/updated-March%202026-blue)

---

## Why this exists

Both Claude Code and Gemini CLI are converging on the same set of primitives. The official docs are comprehensive but optimised for first-time readers. This repo is optimised for the engineer who already knows the tools and needs answers in 10 seconds while their terminal is open.

Two documents. One reference, one playbook.

---

## Contents

### 📋 [LLM CLI Command Reference](LLM-CLI-Command-Reference.md)

Covers the ~25 canonical commands every LLM developer tool converges on, organised for scanning not reading.

| Section | What's in it |
|---|---|
| §1 — Common to both | Session lifecycle, file context, shell passthrough, config, output/scripting, context files, custom slash commands |
| §2 — Claude Code only | Launch flags, tool permissions, system prompt control, REPL commands, subagents, git worktree parallel sessions, hooks |
| §3 — Gemini CLI only | Launch flags, built-in REPL commands, TOML custom command format, MCP management, extensions |
| §4 — The Canonical 25 | Side-by-side table mapping every core concept across both tools |

### 🎯 [Universal AI Agent Playbook](Universal-AI-Agent-Playbook.md)

Concrete workflows with copy-paste prompts shown side-by-side for both tools. Includes 10 universal prompt patterns that work as a tool-agnostic abstraction layer.

| Workflow | What it covers |
|---|---|
| Repository exploration | Bootstrap context, map architecture, find complexity hotspots |
| Understanding unknown modules | Read before prompting, find callers, identify public contracts |
| Fixing production bugs | Triage without guessing, minimal targeted fix, regression tests |
| Code review | Self-review before PR, address open review comments (`/pr-comments`) |
| Documentation generation | OpenAPI specs, module READMEs, inline JSDoc/docstrings |
| Test generation | Unit tests from public interface, full TDD cycle |
| Refactoring legacy code | Safe refactor protocol, parallel worktrees vs staged checkpoints |
| Onboarding engineers | Generate Day 1 guides and ADRs from code, configure AI as domain expert |

---

## Quick start

```bash
git clone https://github.com/choreogrifi/knowledge-llm-cli-playbook
```

Both documents are standalone Markdown. No toolchain required. Open in any editor, render in GitHub, or grep from the terminal.

```bash
# Find everything related to context management
grep -n "context" LLM-CLI-Command-Reference.md

# Find the production bug workflow
grep -n "production" Universal-AI-Agent-Playbook.md
```

---

## The Canonical 25 — preview

Every LLM developer tool eventually converges on these primitives. Master these and you are productive on any future tool.

| # | Concept | Claude Code | Gemini CLI |
|---|---|---|---|
| 1 | Start session | `claude` | `gemini` |
| 2 | Start with prompt | `claude "…"` | `gemini -p "…"` |
| 6 | Reference file | `@./src/api.ts` | `@./src/api.ts` |
| 7 | Run shell command | `!npm test` | `!npm test` |
| 10 | Compact context | `/compact` | `/compress` |
| 14 | Project memory | `CLAUDE.md` | `GEMINI.md` |
| 16 | Custom command | `.claude/commands/foo.md` | `.gemini/commands/foo.toml` |
| 17 | Plan before code | `/plan` | `GEMINI.md` instruction |
| 24 | Subagent | `.claude/agents/` | Via MCP / extensions |
| 25 | Hook / automation | `.claude/settings.json` hooks | `.gemini/settings` automation |

→ [Full table of 25 in the reference doc](LLM-CLI-Command-Reference.md#4--the-canonical-25)

---

## Universal prompt patterns — preview

These work identically on both tools.

```
# Plan gate — stops the AI from writing code before you've reviewed the approach
Give me a numbered plan only. Do not write any code yet.

# Scope constraint — prevents blast-radius surprises
Only touch files under src/payments/. Do not modify tests.

# Rollback safe — always know what you're changing
Before editing, show me the current content of @./src/foo.ts

# Test before change — catches existing breakage before your change does
Run !npm test first. If all pass, proceed. If any fail, stop.
```

→ [All 10 patterns in the playbook](Universal-AI-Agent-Playbook.md#universal-prompt-patterns-tool-agnostic)

---

## Who this is for

- Engineers using Claude Code or Gemini CLI daily who want a fast reference without re-reading docs
- Teams standardising on AI-assisted workflows across a codebase
- Anyone switching between tools and needing a cross-reference
- Engineers onboarding to a codebase with AI assistance

---

## Contributing

Corrections, additions, and new workflows welcome.

- **Wrong command / outdated flag** — open an issue or PR with the correct syntax and a source link
- **New workflow** — follow the structure in the playbook (goal → steps → side-by-side prompts → notes)
- **New tool** — if another LLM CLI reaches the same adoption level, open an issue to discuss adding a column

Both documents are intentionally opinionated about format. New content should match the existing scan-optimised style — no prose intros, no hedging, concrete prompts only.

---

## Versioning

Dated in the document headers. Tools update frequently; if you find a command that no longer works, check the date and open an issue.

| Tool | Version validated against |
|---|---|
| Claude Code | March 2026 |
| Gemini CLI | v0.23+ |

---

## License

MIT. Use freely, attribute if you share.
