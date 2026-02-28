---
name: closed-loop-development
description: Enforces continuous verify-fix-commit loop during development. Discovers project verification stack (typecheck, lint, test, build), generates AGENTS.md for new projects, assesses blast radius before changes, and runs full verification after every edit. Triggers: entering unfamiliar codebase, code changes keep breaking, implementing features across multiple files, or when verification discipline is needed.
---

# Closed Loop Development

## Overview

After every code change, verify. If verification fails, fix and re-verify. Loop until green, then commit atomically.

**Core principle:** The agent closes its own feedback loop — never hand unverified code to a human.

**Violating the letter of this rule is violating the spirit of this rule.**

**Complementary skills:**
- **verification-before-completion** — final gate before claiming work is done. This skill governs the continuous loop *during* development; VBC prevents false completion claims *after*.
- **test-driven-development** — preferred verification strategy. Write tests first for the most reliable verification signal.
- **systematic-debugging** — escalation path when 3 consecutive failures hit. Don't attempt fix #4 without systematic root cause investigation.

## The Iron Law

```
NO CODE CHANGE WITHOUT VERIFICATION
NO COMMIT WITHOUT ALL VERIFICATION PASSING
```

Skipping verification "because it's a small change"? Small changes break builds. Always verify.

## Phase 1: Discover

On first entering a project, detect the verification stack before writing any code.

**Check these and infer commands:**

| File | Look for |
|------|----------|
| `package.json` | `scripts.test`, `scripts.lint`, `scripts.typecheck`, `scripts.build` |
| `go.mod` | `go vet ./...`, `go test ./...`, `go build ./...` |
| `Cargo.toml` | `cargo check`, `cargo clippy`, `cargo test` |
| `pyproject.toml` | `[tool.pytest]`, `[tool.ruff]`, `[tool.mypy]` sections |
| `Makefile` / `justfile` | `test`, `lint`, `check`, `build` targets |
| `tsconfig.json` | `tsc --noEmit` |
| `.eslintrc*` / `biome.json` | Lint command |

**Also check:**
- `.github/workflows/*.yml` — CI defines the source of truth for verification
- `AGENTS.md` / `CLAUDE.md` / `.cursor/rules` — may list commands explicitly
- `.pre-commit-config.yaml` — pre-commit hooks

**If unsure:** Read CI config. It's canonical.

**Store discovered commands for the session.** Typical order (fastest feedback first):
```
1. Type check
2. Lint
3. Test
4. Build
```

**If no verification stack exists (greenfield project):**
Your first task is to create one. Set up at minimum: type checker, linter, test framework, build command. The loop cannot start until at least one verification command is runnable. Bootstrap the toolchain, then begin development.

### Generate AGENTS.md

If no `AGENTS.md` (or `CLAUDE.md`) exists in the project root, generate one. This persists your discovery so future sessions don't repeat it.

**Scan these sources:**

| Source | Extract |
|---|---|
| `package.json` / `go.mod` / `Cargo.toml` / `pyproject.toml` | Tech stack, dependencies, verification commands |
| `tsconfig.json` / `.eslintrc*` / `biome.json` / `ruff.toml` | Coding configuration |
| `.github/workflows/*.yml` | CI commands — source of truth for verification |
| Directory structure (`ls -R` top 2 levels) | Architecture overview, module layout |
| 2–3 representative source files | Coding conventions: naming, patterns, export style, error handling |
| `git log --oneline -20` | Commit convention, branch strategy |
| `docs/` / `README.md` | Project context, domain knowledge |

**Write `AGENTS.md` with these sections:**

````markdown
# AGENTS.md

## Project Overview
[What this project does. Primary language, framework, purpose.]

## Tech Stack
[Language version, framework, key dependencies.]

## Directory Structure
[Key directories and what lives in each.]

## Verification Commands
[Exact commands in order. This is the most critical section.]
```
typecheck: <command>
lint:      <command>
test:      <command>
build:     <command>
```

## Git Protocol
[Commit convention (inferred from git log), branch strategy, PR requirements.]

## Coding Conventions
[Naming style, file organization, import patterns, error handling — inferred from source files.]

## Known Pitfalls
[Project-specific gotchas discovered during scan. Leave empty if none found.]
````

**Present to user for review before saving.** The user knows their project better than you — they may add pitfalls, correct conventions, or adjust commands.

**If AGENTS.md already exists:** Read it, but **verify before trusting**. Run each listed verification command once. If a command fails (not found, wrong tool), check `package.json` scripts and CI config for the real commands. Cross-check coding conventions against 2–3 actual source files — if AGENTS.md says `snake_case` but the code uses `camelCase`, trust the code. Report discrepancies to user and suggest updating AGENTS.md.

## Phase 2: Assess Blast Radius

Before starting any task, estimate scope:

| Blast Radius | Files | Strategy |
|:---:|:---:|---|
| **Small** | 1–3 | Execute → verify → commit |
| **Medium** | 4–10 | Plan first → execute in chunks → verify + commit per chunk. NOT one big commit. |
| **Large** | 10+ | Discuss with user before starting. Break into independent chunks. Each chunk = separate commit. |

Mid-task blast radius larger than expected? **Stop. Reassess. Split.**

## Phase 3: The Loop

After every code edit:

```
┌─→ Run full verification stack
│   ├─ All pass → Phase 4 (Commit)
│   └─ Any fail:
│       ├─ Read error output
│       ├─ Fix ROOT CAUSE
│       └─ Re-run full stack ──────┐
│                                   │
│   3 consecutive failures:         │
│       → STOP                      │
│       → Use systematic-debugging   │
│       → Document what was tried   │
│       → Report to user            │
└───────────────────────────────────┘
```

**Rules:**
- Run **all** verification steps — typecheck AND lint AND test AND build. Not just the ones you remember.
- "All" means all. If you ran typecheck and build but skipped tests, you did NOT verify.
- Fix root cause — never suppress (`@ts-ignore`, `as any`, `// eslint-disable`, empty `catch {}`)
- A fix that breaks something else = new failure, restart count
- Warnings are signal. Don't suppress them to pass.

**If verification tools aren't installed:**
- Install them. Run `npm install`, `pip install`, `cargo build`, whatever the project needs.
- "Tools aren't installed" is not a reason to skip verification — it's a setup step you must complete.
- If you cannot install tools (permissions, network), REPORT to user. Do NOT proceed without verification.
- NEVER rationalize: "the code is correct, tooling is broken" → Fix the tooling or stop.

## Phase 4: Commit

Only when the full verification stack passes:

- **Atomic:** One logical change per commit
- **Conventional Commits:** `feat|fix|refactor|docs|test|chore: description`
- **Why not what:** The diff shows what changed — message describes intent
- **Scope check:** `git status` — stage only files you changed
- Multiple unrelated changes? Commit separately.
- **Blast radius governs commit size:** Medium (4-10 files) = chunk and commit incrementally. Large (10+) = must be broken up. "All changes are cohesive" is not an excuse for a 10+ file commit.

## CLI-First Principle

When building new functionality:

1. Expose as CLI first — agent can call it, verify via stdout/stderr
2. Verify it works through CLI invocation
3. Build UI/API on top after core logic is verified

CLI output is the most agent-friendly feedback format.

## Escalation

| Situation | Action |
|---|---|
| 3 failures on same issue | Stop. Report all attempts + errors to user. |
| Unfamiliar library/tool | Search docs/web first, then attempt. |
| Architecture uncertainty | Propose 2–3 options to user before implementing. |
| Test seems wrong | Only fix if confident. Ask user if unsure. |
| **Authority says skip verification** | **Push back.** Explain risk. Verify anyway. You answer to code quality, not speed. The user can override after hearing the risk — but YOU must raise it. |
| **Emergency / production down** | Still verify. A broken fix is worse than a slow fix. Run at minimum: typecheck + build. Explicitly note any steps you skipped and why. |

## Quick Reference

```
ENTER PROJECT → Discover verification stack
BEFORE TASK   → Assess blast radius
AFTER EDIT    → Verify (typecheck → lint → test → build)
IF FAIL       → Read error → Fix root cause → Re-verify (max 3×)
IF GREEN      → Atomic commit (conventional commits)
IF STUCK      → Stop → Document attempts → Report to user
```

## Common Mistakes

| Mistake | Why it's wrong |
|---|---|
| Skipping verification "small change" | Small changes break builds. Always verify. |
| Suppressing lint (`// eslint-disable`) | Hides problems. Fix the actual code. |
| Committing with failing tests | Broken commits pollute history and block others. |
| Only running tests, skipping typecheck | Type errors compound. Run the full stack. |
| Massive commits touching 20+ files | Hard to review, hard to revert. Break into chunks. |
| Coding before reading project docs | You'll fight existing patterns. Discover first. |
| Fixing symptoms instead of root cause | The bug will return. Find the real cause. |

## Red Flags — STOP

- About to commit without running verification
- Suppressing errors to make verification pass
- "It's just a small change, no need to verify"
- 3+ files changed without a single verification run
- Fixing the test to match wrong behavior (instead of fixing the code)
- Skipping blast radius assessment on a large change

- Someone told you to skip verification (authority override)
- Verification tools aren't installed and you're about to proceed anyway
- You ran typecheck and build but "forgot" tests
- Medium/large change being committed as a single monolithic commit
**All of these mean: Stop. Run verification. Fix issues. Then proceed.**

## Rationalization Prevention

| Excuse | Reality |
|---|---|
| "Small change, no need to verify" | Small changes cause outages. Always verify. |
| "Lint is too noisy" | Fix the noise or configure the rules. Don't disable. |
| "Tests are slow, I'll run them later" | Later never comes. Run them now. |
| "I know this works" | Knowledge ≠ evidence. Run the command. |
| "I'll batch verify at the end" | Errors compound. Verify per change. |
| "The CI will catch it" | Local verification is 10× faster than waiting for CI. |
| "Tech lead said skip tests" | You answer to code quality. Push back, explain risk, verify anyway. |
| "Tools aren't installed, not my code's fault" | Install the tools. Broken toolchain ≠ skip verification. |
| "All changes are cohesive, one commit is fine" | Cohesive ≠ atomic. If blast radius is medium+, chunk it. |
| "It's an emergency, no time to verify" | A broken fix deployed to prod is worse than a 5-minute delay. Verify. |
| "I ran typecheck and build, that's enough" | That's 2 of 4. Run ALL steps: typecheck, lint, test, build. |
