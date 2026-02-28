# Closed Loop Development

An agent skill that enforces a continuous **verify-fix-commit** loop during development.

Based on [Peter Steinberger's](https://steipete.me) AI-assisted development methodology — the same approach used to ship PSPDFKit and PDF Viewer.

## What it does

```
Enter project → Discover verification stack
Before task   → Assess blast radius
After edit    → Verify (typecheck → lint → test → build)
If fail       → Fix root cause → Re-verify (max 3×)
If green      → Atomic commit
If stuck      → Stop → Report to user
```

## Key features

- **Discovery** — Auto-detects project verification commands from `package.json`, `go.mod`, `Cargo.toml`, CI configs, etc.
- **AGENTS.md generation** — Creates project documentation for future agent sessions when none exists
- **Blast radius assessment** — Estimates change scope and picks commit strategy accordingly
- **The Loop** — Runs full verification (typecheck + lint + test + build) after every edit
- **Authority resistance** — Pushes back when told to skip verification, even under pressure
- **Toolchain bootstrap** — Installs missing verification tools instead of skipping verification

## Install

### Claude Code

```bash
# Copy SKILL.md to your skills directory
mkdir -p ~/.claude/skills/closed-loop-development
curl -o ~/.claude/skills/closed-loop-development/SKILL.md \
  https://raw.githubusercontent.com/addison-w/closed-loop-development/main/SKILL.md
```

### OpenCode / Other agents

```bash
mkdir -p ~/.agents/skills/closed-loop-development
curl -o ~/.agents/skills/closed-loop-development/SKILL.md \
  https://raw.githubusercontent.com/addison-w/closed-loop-development/main/SKILL.md
```

## Tested with TDD for skills

This skill was developed using RED-GREEN-REFACTOR methodology for agent skills:

- **RED**: 6 scenarios where agents failed without the skill (skipped tests under authority pressure, bypassed broken toolchains, forgot verification steps)
- **GREEN**: Same 6 scenarios with the skill — all failures corrected
- **REFACTOR**: Rationalization table covers 11 common excuses agents use to skip verification

## Language agnostic

Works with any language or project type: TypeScript, Go, Rust, Python, Java, Swift, and more. The skill discovers the verification stack from project config files — no hardcoded assumptions.

## License

MIT
