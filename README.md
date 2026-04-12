# ShipLoop

**Your AI-coded app works 80% of the time. ShipLoop gets it to shipped.**

Not "code compiles" done. *"A stranger can use this"* done.

---

## The Problem

You built something with AI. It mostly works. But "mostly" isn't shippable. Edge cases crash, empty states show nothing, error messages say "something went wrong," and flows that make sense to you confuse everyone else.

You know the gap exists. Closing it means finding every issue, fixing it, verifying the fix didn't break something else, and repeating — until everything actually works.

ShipLoop does that for you. Autonomously. In a loop. Until it's done.

---

## How It Works

```
You: /shiploop

ShipLoop: "Tell me about this project."

You: "It's a task app. Works but the UI is
      rough and new users get confused."

ShipLoop: "I found 28 issues. Here's my plan."

You: "Go."

ShipLoop works autonomously.
Fixes → verifies → tracks health score → loops.

Health: 72 -> 81 -> 88 -> 93 -> 96

ShipLoop: "Done. 23 of 28 fixed.
           Report in .shiploop/report.md."
```

---

## The Architecture

### The Loop

```
  ┌─────────────┐
  │  WORK        │  Subagent with code access.
  │  Fix one     │  Reads the plan. Makes changes.
  │  issue.      │  Commits each fix.
  └──────┬───────┘
         │
         ▼
  ┌─────────────┐
  │  VERIFY      │  Separate subagent.
  │  Test like   │  CANNOT see source code.
  │  a user.     │  Only interacts with the output.
  └──────┬───────┘
         │
    ┌────┴────┐
    │ Pass?   │
    └────┬────┘
     no  │  yes
     │   │
     │   ▼
     │  Track health score
     │  Next task from plan
     │   │
     ▼   ▼
    Fix  ──────► Loop until health >= 95
```

Each subagent gets its own fresh context. The coordinator stays lightweight — just dispatching and tracking. No context bloat even after hours of autonomous work.

### Separated Evaluation

The builder sees code but not results. The verifier sees results but not code. This is the key insight — an agent grading its own work is generous. A separate agent testing only the output catches what self-review misses.

### Health Score Convergence

Quality is a number. Critical bugs count 4x, high bugs 2x. The score must trend upward or the circuit breaker fires (3 no-progress cycles = stop). The convergence curve proves the system is working:

```
Health: 72 ── 81 ── 88 ── 93 ── 96
        C1    C2    C3    C4    C5
```

### Self-Improving Checklist

Starts with 62 universal checks. Each run, ShipLoop discovers new things to check and adds them. When a bug slips through, it categorizes why the checklist missed it and adds a new check. After 5+ runs, it prunes checks that never trigger. The checklist converges on your project's specific needs.

---

## The Five Phases

**Phase 1: Listen** — A real conversation about your project. What matters, what's broken, what "done" means.

**Phase 2: Explore** — Auto-detects tech stack, runs the app, reads the code, forms its own opinions. Finds things you didn't mention.

**Phase 3: Plan** — Combines what you said with what it found. Proposes a plan. Gets your approval. Last time it needs you.

**Phase 4: Work** — The autonomous loop. Fix → verify → track → repeat. Each cycle uses fresh subagents. Commits every fix. Logs everything.

**Phase 5: Deliver** — Report of everything done, health score progression, remaining items.

You're present for Phases 1-3 (~5 minutes). Phases 4-5 are fully autonomous.

Supports: React, Next.js, Vue, Svelte, Angular, Python, Go, Rust, Ruby, CLIs, APIs, libraries, documentation — anything with source files.

---

## What Gets Created

```
.shiploop/
  context.md              What you told ShipLoop
  findings.md             What ShipLoop found independently
  plan.md                 The approved plan
  log.txt                 Every cycle, logged
  health.json             Health score over time
  report.md               Final report
  learnings.md            Persists across runs — each run starts smarter
  checklist-additions.md  Project-specific checks, growing over time
```

Run `/shiploop` again and it picks up where it left off.

---

## Limitations

- Requires Claude Code with subagent support
- Browser testing requires [agent-browser](https://github.com/vercel-labs/agent-browser) — skipped if not installed
- Large monorepos may exceed context — works best on focused projects
- Health score is a heuristic — track trends, not absolute numbers
- Not a replacement for human taste — ShipLoop fixes a lot, but design judgment still needs you

---

## Install

```bash
git clone https://github.com/VatsalEnpal/Ship-loop ~/.claude/skills/shiploop
```

No config. No env vars. No setup scripts.

## Use

```bash
cd ~/your-project
claude
> /shiploop
```

---

## License

MIT

---

## Standing On The Shoulders Of

- **[Anthropic](https://anthropic.com)** — Claude Code, subagents, tool-use architecture
- **[Andrej Karpathy](https://github.com/karpathy/autoresearch)** — Autoresearch: try, measure, keep or discard. Applied here to product quality
- **[Vercel](https://github.com/vercel-labs/agent-browser)** — agent-browser CLI
- **[Garry Tan](https://github.com/garrytan/gstack)** — gstack QA patterns
- **[Baymard Institute](https://baymard.com)** — Narrow classification: yes/no questions at 95% accuracy
- **[obra](https://github.com/obra/superpowers)** — Superpowers workflow framework
- **[Anthropic Research](https://www.anthropic.com/engineering)** — Generator/evaluator separation
- **[frankbria](https://github.com/frankbria/ralph-claude-code)** — Ralph Loop, circuit breaker pattern
- **[celesteanders](https://github.com/celesteanders/harness)** — Harness: file-based state machines
