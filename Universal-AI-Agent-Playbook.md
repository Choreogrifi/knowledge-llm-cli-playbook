# Universal AI Coding Agent Playbook
### Command abstraction layer for generic workflows — Claude Code · Gemini CLI — March 2026

> Each workflow shows exact prompt patterns for both tools. Side-by-side where approaches differ; unified where identical. Copy-paste into your terminal.

---

## Universal Prompt Patterns (Tool-Agnostic)

These work identically on both tools. They are the abstraction layer.

| Pattern | Prompt |
|---|---|
| Explore before act | `Read @./src/services before suggesting any changes.` |
| Constrain scope | `Only touch files under src/payments/. Do not modify tests.` |
| Plan gate | `Give me a numbered plan only. Do not write any code yet.` |
| Iterative approval | `Implement step 1 only. Stop and wait for my review.` |
| Diff context | `!git diff HEAD~1 -- src/` then: `Explain what changed and why.` |
| Test before change | `Run !npm test first. If all pass, proceed. If any fail, stop.` |
| Rollback safe | `Before editing, show me the current content of @./src/foo.ts` |
| Targeted refactor | `Refactor @./src/old.ts to use dependency injection. No other files.` |
| Document as you go | `After each function you write, add a JSDoc comment block.` |
| Explain rationale | `For each change, include a one-line comment explaining why.` |

---

## Workflow 1 — Repository Exploration

**Goal:** Understand structure, entry points, and dependencies in under 10 minutes on an unfamiliar codebase.

---

### 1.1 Bootstrap project context

**Step 1 — Initialise project memory file**

```bash
# Claude Code
/init
# Claude scans the repo and writes CLAUDE.md

# Gemini CLI
gemini -p "Scan this repo. Write a GEMINI.md covering: tech stack,
entry points, key modules, test strategy, and conventions."
```

> Commit this file. Every future session starts with accurate codebase context for free.

**Step 2 — Validate the context file**

```
# Both tools — run after /init or GEMINI.md is written
@./CLAUDE.md Does this accurately represent the repo? What is missing?
```

---

### 1.2 Map the architecture

**Step 1 — High-level structure and entry points**

```
# Both tools
Describe the architecture of this codebase.
Identify: entry points, domain boundaries, external dependencies.
Do not suggest changes yet.
```

**Step 2 — Trace a key request lifecycle**

```
# Both tools
Trace the request lifecycle from the HTTP handler in @./src/api/routes/
through to the database call. Show me each layer and file involved.
```

**Step 3 — Identify complexity hotspots**

```bash
# Both tools
!find src -name "*.ts" | xargs wc -l | sort -rn | head -20
# Then:
The largest files are listed above. Which are the highest-risk to modify and why?
```

---

## Workflow 2 — Understanding Unknown Modules

**Goal:** Understand what a module does, what it depends on, and how to safely extend it.

---

### 2.1 Read before prompting

**Step 1 — Load the module and its dependencies**

```
# Both tools
Explain what @./src/payments/processor.ts does.
Then list every file it imports and what role each plays.
```

**Step 2 — Find all callers**

```bash
# Both tools
!grep -r "payments/processor" src/ --include="*.ts" -l
# Then:
Which files call into the payment processor? How does each use it?
```

**Step 3 — Identify the public contract**

```
# Both tools
Looking at @./src/payments/processor.ts and its callers,
identify the public interface that must remain stable.
What can I change without breaking callers?
```

---

### 2.2 Understand test coverage

```bash
# Both tools
!find tests -name "*processor*" -o -name "*payment*" 2>/dev/null
# Then:
Show me what aspects of the payment processor are tested.
What edge cases are missing?
```

---

## Workflow 3 — Fixing Production Bugs

**Goal:** Identify root cause, write a minimal targeted fix, verify, and leave a paper trail.

---

### 3.1 Triage: confirm root cause before writing a line

**Step 1 — Give the AI everything: stack trace + code**

```
# Both tools
Here is the error from production:

<paste stack trace here>

Read @./src/services/order.ts and identify the root cause.
Do not suggest fixes yet. Just confirm the root cause.
```

> The "do not fix yet" constraint forces thorough analysis before action.

**Step 2 — Check git history for recent changes**

```bash
# Both tools
!git log --oneline -10 -- src/services/order.ts
# Then:
Summarise these recent changes. Could any of them have introduced the bug?
```

---

### 3.2 Minimal targeted fix

**Step 1 — Implement with explicit scope constraint**

```
# Both tools
Fix the root cause in @./src/services/order.ts.
Change the minimum number of lines required.
Do not refactor. Do not change unrelated code.
Show me a diff before writing anything.
```

> "Show me a diff before writing" is the most important safety constraint for production fixes.

**Step 2 — Verify against existing tests**

```bash
# Both tools
!npm test -- --testPathPattern=order
# If failures:
Tests failed at: <paste failure>
Explain whether this failure is related to our change.
```

**Step 3 — Write a regression test**

```
# Both tools
Write a test that would have caught this bug before it reached production.
Add it to @./tests/services/order.test.ts
The test should fail before the fix and pass after.
```

---

## Workflow 4 — Code Review

**Goal:** Consistent, thorough review that catches what human reviewers miss — security holes, edge cases, architectural drift.

---

### 4.1 Self-review before submitting

**Step 1 — Review staged changes**

```bash
# Both tools
!git diff --cached
# Then:
Review these staged changes. Check for:
1. Logic errors or off-by-one bugs
2. Unhandled error paths
3. Security issues (injection, auth bypass, data exposure)
4. Violation of the architecture in @./docs/architecture.md
5. Missing tests for new code paths
```

**Step 2 — Use built-in review tooling**

```bash
# Claude Code
/review
# Invokes review subagent against current changes

# Gemini CLI — no built-in; create .gemini/commands/review.toml:
# prompt = "Review @./src changes: logic, security, architecture, test coverage"
/review
```

---

### 4.2 Address open PR review comments

```bash
# Claude Code
/pr-comments
# Reads GitHub PR context automatically and addresses each comment

# Gemini CLI — load data manually
!gh pr diff 123 > /tmp/pr.diff
!gh pr view 123 --comments > /tmp/pr-comments.txt
# Then:
@/tmp/pr.diff @/tmp/pr-comments.txt
Address each reviewer comment. For each: state the change, then implement it.
```

> `/pr-comments` is a Claude Code built-in that reads GitHub context automatically. Gemini needs the data loaded explicitly.

---

## Workflow 5 — Documentation Generation

**Goal:** Accurate docs generated from code — not aspirational markdown that drifts from reality.

---

### 5.1 Generate API documentation

**Step 1 — OpenAPI spec from route files**

```
# Both tools
Read every file in @./src/api/routes/
Generate an OpenAPI 3.1 YAML spec covering all endpoints.
Include: path, method, request body schema, response schemas, error codes.
Write it to docs/openapi.yaml
```

**Step 2 — README for a module**

```
# Both tools
Generate a README.md for @./src/payments/
Include: purpose, public API surface, usage examples, configuration, error handling.
Draw examples from the existing tests in @./tests/payments/
```

---

### 5.2 Inline code documentation

```
# Both tools
Add JSDoc comments to every exported function in @./src/utils/validation.ts
Do not change any logic. Only add documentation.
For each function: describe what it does, its parameters, return type, and one usage example.
```

---

## Workflow 6 — Test Generation

**Goal:** Tests that cover behaviour, not just lines. Happy paths, error paths, and edge cases.

---

### 6.1 Unit tests for an untested module

**Step 1 — Generate tests from the public interface**

```
# Both tools
Generate unit tests for @./src/services/pricing.ts
Cover: happy path, invalid inputs, boundary values, error propagation.
Use Jest. Mirror the structure of existing tests in @./tests/services/
Do not test private methods. Only the public interface.
```

**Step 2 — Verify they run**

```bash
# Both tools
!npm test -- --testPathPattern=pricing
# If failures: explain what the test revealed about the implementation
```

---

### 6.2 TDD workflow — test first

**Step 1 — Write a failing test for a feature that does not exist**

```bash
# Both tools
Write a failing unit test for a function called validateOrderAmount.
Requirements: amount must be > 0, <= 10000, numeric, max 2 decimal places.
Do not implement the function. Just the test.
!npm test -- --testPathPattern=validateOrderAmount
# Confirm it fails before continuing.
```

> "Run it to confirm it fails" prevents generating tests that trivially pass.

**Step 2 — Implement to make the tests pass**

```bash
# Both tools
Now implement validateOrderAmount to make the tests pass.
Minimum code required. No over-engineering.
!npm test -- --testPathPattern=validateOrderAmount
```

---

## Workflow 7 — Refactoring Legacy Code

**Goal:** Improve structure without changing observable behaviour. Rule: green tests before, green tests after.

---

### 7.1 Safe refactor protocol

**Step 1 — Establish a test baseline**

```bash
# Both tools
!npm test -- --coverage --testPathPattern=src/legacy/
# Then:
Coverage report above. Before I refactor @./src/legacy/processor.ts,
what is the minimum set of tests I need to add to make the refactor safe?
```

> If coverage < 80% on the target module, add characterisation tests first.

**Step 2 — Plan the refactor without implementing**

```bash
# Claude Code
/plan
# Then in the plan session:
Plan a refactor of @./src/legacy/processor.ts to hexagonal architecture.
Show me the target file structure, new interfaces, and migration steps.
Do not write any implementation code.

# Gemini CLI
Plan a refactor of @./src/legacy/processor.ts to hexagonal architecture.
Show me the target file structure and migration steps. No code yet.
```

**Step 3 — Implement in small verified increments**

```bash
# Both tools
Implement only Step 1 from the plan: extract the database interface.
!npm test -- --testPathPattern=processor
Stop if any test fails.
```

---

### 7.2 Large-scale refactoring across multiple files

**Step 1 — Scope the full impact**

```bash
# Both tools
!grep -r "legacyProcessor" src/ --include="*.ts" -l
# Then:
How many files reference legacyProcessor?
Generate a dependency graph and estimated refactor effort.
```

**Step 2 — Execute: parallel agents (CC) vs staged checkpoints (GEM)**

```bash
# Claude Code — parallel worktrees, agents never touch each other's branches
git worktree add ../refactor-core feat/refactor-core-uuid123
git worktree add ../refactor-api  feat/refactor-api-uuid456

# In ../refactor-core:
claude "Refactor src/core/processor.ts to hexagonal. Tests must stay green."

# In ../refactor-api (independent branch):
claude "Refactor src/api/handler.ts to use the new processor interface."
```

```bash
# Gemini CLI — sequential stages with named checkpoints
/chat save "pre-refactor-baseline"

# Stage 1: core only
Refactor src/core/processor.ts only. No other files.
!npm test
/chat save "post-core-refactor"

# Stage 2: API layer
Now update src/api/handler.ts to use the new interface.
!npm test
```

> Claude Code worktrees enable true parallel execution. Gemini's checkpoint-based staging is safer for codebases without comprehensive test coverage.

---

## Workflow 8 — Onboarding Engineers

**Goal:** Compress weeks of institutional knowledge into hours. Generate materials from the codebase, not from memory.

---

### 8.1 Generate onboarding documentation from code

**Step 1 — "Day 1 Setup" guide**

```
# Claude Code
Read @./CLAUDE.md and:
@./package.json @./docker-compose.yml @./Makefile @./README.md

Write a step-by-step "Day 1 Setup" guide for a new engineer.
Include: prerequisites, local setup, running the stack, running tests,
the development workflow, and 3 "good first tasks" based on code complexity.

# Gemini CLI — same prompt, replace CLAUDE.md with GEMINI.md
```

**Step 2 — Generate architectural decision records (ADRs)**

```
# Both tools
Analyse the codebase and identify 5 significant architectural decisions.
For each, write an ADR in this format:
  Title | Status | Context | Decision | Consequences
Base each decision on evidence you find in the code.
```

---

### 8.2 Configure the AI as a domain expert for Q&A sessions

Add to `CLAUDE.md` / `GEMINI.md`:

```markdown
## Onboarding mode
When answering questions from new engineers:
1. Answer the question directly.
2. Point to the specific file and line where it is implemented.
3. Explain the "why" — the reason this design choice was made.
4. Suggest a related thing to read next.
```

Then a new engineer can just ask:

```
Why do we have both a PaymentService and a PaymentProcessor?
What is the difference and when do I use each?
```

With the context file loaded, both tools answer from actual codebase knowledge.

---

## Prompt Anti-Patterns

| Anti-pattern | Why it fails | Better prompt |
|---|---|---|
| `Fix my code` | No context. AI guesses scope and intent. | `Fix @./src/order.ts so validateAmount rejects zero values. Run tests after.` |
| `Refactor everything` | Unbounded scope. AI touches too much. | `Refactor @./src/payments/ only. No other directories. Show a plan first.` |
| `Add tests` | Which module? Which framework? What coverage? | `Add Jest unit tests for exported functions in @./src/utils/format.ts. No mocks.` |
| `Why is this slow?` | AI cannot profile without data. | `!node --prof app.js` then `@profile.txt — find the hotspot.` |
| `Write documentation` | Too vague. Produces generic output. | `Write JSDoc for each exported function in @./src/api/client.ts. No logic changes.` |
| `Review my PR` | No diff loaded. No review criteria given. | `!git diff main...HEAD` then `Review for: security, error handling, test coverage, arch compliance.` |
| `Help me understand this` | No file referenced. AI guesses. | `Explain @./src/core/eventBus.ts. Then tell me every file that depends on it.` |

---

*Tools change. Prompting discipline does not.*
