# ShipLoop

**Your AI-coded app works 80% of the time. ShipLoop gets it to shipped.**

A Claude Code skill that acts like a senior engineer joining your project. It listens to what you need, explores your codebase, makes a plan, then works autonomously — fixing, verifying, and improving in a loop until the product is actually done.

Not "code compiles" done. *"A stranger can use this"* done.

---

## The Problem

You built something with AI. It mostly works. But "mostly" isn't shippable. There are edge cases that crash, empty states that show nothing, error messages that say "something went wrong," buttons that look clickable but do nothing, and flows that make sense to you but confuse everyone else.

You know the gap exists. You just don't have the patience to find every issue, fix it, verify the fix didn't break something else, and repeat until everything actually works.

ShipLoop does that for you.

---

## How It Works

```
You: /shiploop

ShipLoop: "Tell me about this project."

You: "It's a task app. It mostly works but the
      UI is rough and new users get confused."

ShipLoop: "What does 'done' look like for you?"

You: "Someone can sign up, create a task, and
      complete it without getting stuck."

ShipLoop: "Got it. Let me look at the code..."
          "I found 28 issues. Here's my plan.
           Want me to start?"

You: "Yes."

ShipLoop works autonomously.
Fixes issues one at a time.
Verifies each fix — not by reading code,
but by testing like a user.
Catches its own regressions.
Tracks a health score.

Health: 72 -> 81 -> 88 -> 93 -> 96

ShipLoop: "Done. 23 of 28 issues fixed.
           Health went from 72 to 96.
           Report in .shiploop/report.md."
```

You're only needed for the first 5-10 minutes (the conversation + plan approval). Everything after that is autonomous.

---

## Install

```bash
git clone https://github.com/VatsalEnpal/Ship-loop ~/.claude/skills/shiploop
```

No config files. No environment variables. No setup scripts.

## Use

```bash
cd ~/your-project
claude
> /shiploop
```

---

## The Five Phases

### Phase 1: Listen

ShipLoop has a real conversation with you. Not a form — a conversation. It asks about the project, what matters, what's broken, what "done" looks like. It asks follow-ups. It challenges vague answers. It mirrors back what it understood and confirms before moving on.

### Phase 2: Explore

ShipLoop looks at the project independently. It auto-detects the tech stack, runs the app, reads the code, and forms its own opinions. It finds things you didn't mention.

Supports: React, Next.js, Vue, Svelte, Angular, Python, Go, Rust, Ruby, CLIs, APIs, libraries, documentation projects, and anything with source files.

### Phase 3: Plan

ShipLoop combines what you said with what it found. Proposes a prioritized plan. Gets your approval. This is the last time it needs you.

### Phase 4: Work (autonomous)

The core loop. Each cycle:

1. Pick the next task from the plan
2. Do the work (as a focused subagent with its own context)
3. Verify it worked (as a **separate** subagent that cannot see the source code — only the result)
4. If verification fails, fix and re-verify
5. Track health score, log everything
6. Loop until quality converges

The builder and verifier are deliberately separated. The builder sees code but not results. The verifier sees results but not code. This catches the bugs that code review misses — the ones users actually hit.

### Phase 5: Deliver

ShipLoop writes a report, commits everything, and shows you the results.

---

## What Makes This Different

**It starts with listening.** Every other tool starts with "here's a prompt, go." ShipLoop starts with understanding what you actually need. The quality of the autonomous work depends entirely on the quality of this conversation.

**The builder never judges its own work.** The verification subagent cannot access source code. It can only interact with the output — opening the app in a browser, running CLI commands, making API requests, running tests. This separation catches regressions that self-review misses.

**The checklist gets smarter.** Starts with 60+ universal quality checks. During each run, ShipLoop discovers new things to check and adds them. When it finds a bug the checklist didn't predict, it categorizes why and adds a new check. After 5+ runs, it prunes checks that never trigger. The checklist converges on complete coverage for your specific project.

**Quality is a number.** The health score tracks product quality over time. Critical bugs count 4x, high bugs 2x. The convergence curve proves the system is working. If it flattens, the circuit breaker stops the loop.

---

## What Gets Created

ShipLoop creates a `.shiploop/` directory in your project:

```
.shiploop/
  context.md              What you told ShipLoop (Phase 1)
  findings.md             What ShipLoop found on its own (Phase 2)
  plan.md                 The approved plan (Phase 3)
  log.txt                 Every cycle, logged (Phase 4)
  health.json             Health score over time (Phase 4)
  report.md               Final report (Phase 5)
  learnings.md            What ShipLoop learned (persists across runs)
  checklist-additions.md  Project-specific checks (grows over time)
```

Run `/shiploop` again and it picks up where it left off — reading previous context, learnings, and checklist additions. Each run starts smarter than the last.

---

## Limitations

- **Requires Claude Code** with subagent support
- **Browser testing requires [agent-browser](https://github.com/vercel-labs/agent-browser)** — if not installed, browser-based checks are skipped
- **Large monorepos may exceed context** — works best on focused projects
- **Health score is a heuristic** — use it to track trends, not as an absolute measure
- **Not a replacement for human review** — ShipLoop finds and fixes a lot, but taste still requires a human eye

---

## License

MIT

---

## Standing On The Shoulders Of

This project wouldn't exist without the ideas, tools, and research from:

- **[Anthropic](https://anthropic.com)** — Claude Code, subagents, the tool-use architecture that makes autonomous work possible
- **[Andrej Karpathy](https://github.com/karpathy/autoresearch)** — The autoresearch pattern: try, measure, keep or discard. Applied here to product quality instead of ML training
- **[Vercel](https://github.com/vercel-labs/agent-browser)** — agent-browser CLI and the "dogfood" skill that tests apps like a real user
- **[Garry Tan](https://github.com/garrytan/gstack)** — gstack's QA patterns and the idea that slash commands can encode entire engineering workflows
- **[Baymard Institute](https://baymard.com)** — The narrow classification methodology: specific yes/no questions achieve 95% accuracy vs 50-75% for open-ended evaluation
- **[Jesse Vincent / obra](https://github.com/obra/superpowers)** — Superpowers framework for structured AI development workflows
- **[Anthropic Research](https://www.anthropic.com/engineering)** — The generator/evaluator separation architecture that prevents self-generous grading
- **[frankbria](https://github.com/frankbria/ralph-claude-code)** — Ralph Loop and the circuit breaker pattern for autonomous agent safety
- **[celesteanders](https://github.com/celesteanders/harness)** — The harness pattern for file-based state machines between agent sessions
