# 🔄 Closed Loop Development

**An agent skill that makes AI coding agents actually verify their work.**

> Edit → Verify → Fix → Verify again → Commit. No exceptions. No excuses.

Inspired by [Peter Steinberger's](https://steipete.me) battle-tested AI development workflow.

---

## 🤔 The Problem

AI agents love to write code and immediately declare victory. They skip tests, ignore linters, and commit broken builds. When called out, they rationalize: *"it's a small change"*, *"I'll run tests later"*, *"the CI will catch it."*

This skill fixes that.

## ✨ What It Does

```
🔍 Enter project  →  Discover verification stack
📐 Before task    →  Assess blast radius
✅ After edit     →  Verify (typecheck → lint → test → build)
🔧 If fail        →  Fix root cause → Re-verify (max 3×)
💾 If green       →  Atomic commit
🛑 If stuck       →  Stop → Report to user
```

## 🎯 Key Features

| | Feature | What it does |
|---|---|---|
| 🔍 | **Auto-Discovery** | Detects verification commands from `package.json`, `go.mod`, `Cargo.toml`, CI configs, etc. |
| 📝 | **AGENTS.md Generation** | Creates project docs for future agent sessions when none exist |
| 💥 | **Blast Radius Assessment** | Estimates change scope → picks commit strategy accordingly |
| 🔁 | **The Loop** | Runs full verification (typecheck + lint + test + build) after *every* edit |
| 🛡️ | **Authority Resistance** | Pushes back when told to skip verification — even under pressure |
| 🔧 | **Toolchain Bootstrap** | Installs missing tools instead of skipping verification |

## 🌍 Language Agnostic

Works with **any** language or project type:

TypeScript · Go · Rust · Python · Java · Swift · and more

The skill discovers the verification stack from project config files — zero hardcoded assumptions.

## 🧪 TDD-Tested (Yes, Really)

Built using RED-GREEN-REFACTOR methodology *for agent skills*:

- 🔴 **RED** — 9 scenarios where agents failed without the skill
  - Obeyed "skip tests" from authority without pushback
  - Bypassed broken toolchains entirely
  - Forgot to run tests after typecheck + build
  - Skipped AGENTS.md generation under task pressure
  - Saved AGENTS.md without user review
  - Blindly trusted incorrect AGENTS.md
- 🟢 **GREEN** — Same 9 scenarios with the skill → all corrected
- ♻️ **REFACTOR** — 11 common agent rationalizations catalogued with counters

## 📦 Install

### Using Skills CLI (Recommended)

```bash
# bunx
bunx skills add addison-w/closed-loop-development -g

# npx
npx skills add addison-w/closed-loop-development -g
```

### Claude Code (Manual)

```bash
mkdir -p ~/.claude/skills/closed-loop-development
curl -o ~/.claude/skills/closed-loop-development/SKILL.md \
  https://raw.githubusercontent.com/addison-w/closed-loop-development/main/SKILL.md
```

### OpenCode / Other Agents (Manual)

```bash
mkdir -p ~/.agents/skills/closed-loop-development
curl -o ~/.agents/skills/closed-loop-development/SKILL.md \
  https://raw.githubusercontent.com/addison-w/closed-loop-development/main/SKILL.md
```

## 📄 License

MIT
