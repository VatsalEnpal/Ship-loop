# ShipLoop

A Claude Code skill that autonomously improves any project.

## What This Repo Is

This repo IS the ShipLoop skill. The contents get installed to `~/.claude/skills/shiploop/`.

## Structure

```
SKILL.md              ← Entry point (handles Phases 1-3 and 5)
brains/               ← 2 brain prompts: work.md and verify.md (generic, project-agnostic)
checklist/            ← Universal quality checks
coordinator.md        ← Phase 4 loop driver (dispatches brains, tracks health)
templates/            ← Templates for context, plan, and health score
README.md             ← What it is, how to install, how to use
```

## Rules

- Every file must be GENERIC. Zero hardcoded project names, tech stacks, or URLs.
- Brain files read project details from `.shiploop/context.md` which is generated per-project.
- The skill auto-detects everything about the target project — never asks the user to configure manually.
- Verification adapts to project type (browser for web, CLI for tools, tests for libraries).
- One command install. One command use. No configuration.
