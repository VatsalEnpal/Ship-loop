# Coordinator: Phase 4 Loop Driver

You drive the autonomous work loop in Phase 4. You dispatch work and verify subagents, track progress, and decide when to stop.

## Setup

Before the first cycle, initialize tracking:

1. Read `.shiploop/plan.md` to get the task list
2. Read `.shiploop/context.md` to know the project type and verification approach
3. Read `.shiploop/findings.md` for baseline issues
4. Create or update `.shiploop/log.txt` with a header:

```
========================================
ShipLoop Run: [timestamp]
Project Type: [from context.md]
Tasks Planned: [count]
========================================
```

5. Initialize health score tracking in `.shiploop/health.json`:

```json
{
  "current_score": 0,
  "history": [],
  "total_checks": 0,
  "critical": 0,
  "high": 0,
  "medium": 0,
  "low": 0,
  "cycle": 0,
  "no_progress_count": 0
}
```

## The Loop

Each cycle follows this exact sequence:

### Step 1: Pick the Next Task

Read `.shiploop/plan.md`. Find the next task that is NOT marked as `[DONE]` or `[SKIPPED]`.

If no tasks remain, check if health score >= 95 with zero critical/high issues:
- If yes: proceed to Phase 5 (DELIVER)
- If no: generate additional tasks based on remaining issues and continue

### Step 2: Dispatch Work Subagent

Launch a subagent with the prompt from `brains/work.md`. Pass it:
- The specific task to complete
- Any relevant context from previous cycles (what failed, what to watch for)

Wait for the work subagent to complete. Capture its output.

### Step 3: Dispatch Verify Subagent

Launch a SEPARATE subagent with the prompt from `brains/verify.md`. Pass it:
- What task was just completed (description only, not code)
- What to verify (expected behavior)
- Relevant checks from `checklist/universal.md`
- The project type and how to test it

**The verify subagent must NOT receive source code or diffs.** Only descriptions of expected behavior.

Wait for the verify subagent to complete. Capture its output.

### Step 4: Process Results

Parse the verification output:

**If PASS:**
1. Mark the task as `[DONE]` in `.shiploop/plan.md`
2. Update health score (see formula below)
3. Log success to `.shiploop/log.txt`
4. Reset no-progress counter

**If FAIL:**
1. Log the failure and issues to `.shiploop/log.txt`
2. Re-dispatch the work subagent with the specific issues to fix
3. Re-dispatch verify after the fix
4. If it fails AGAIN on the same task, mark as `[BLOCKED]` and move on
5. Increment no-progress counter if health score didn't change

**If PARTIAL:**
1. Log what passed and what failed
2. If critical/high issues: re-dispatch work to fix those, then re-verify
3. If only medium/low: mark task as `[DONE]` with notes, move on
4. Update health score

### Step 5: Check Side Effects

If the work subagent reported side effects or new issues:
1. Add them to `.shiploop/plan.md` as new tasks
2. Prioritize: critical issues go to the top, others append

### Step 6: Update Health Score

```
Health Score Formula:
  total = total_checks_run
  if total == 0: score = 0  # guard against division by zero
  else:
    penalty = (critical * 4) + (high * 2) + (medium * 1)
    score = ((total - penalty) / total) * 100
    score = max(0, min(100, score))  # clamp to 0-100
```

Update `.shiploop/health.json`:
```json
{
  "current_score": [new score],
  "history": [...previous, {"cycle": N, "score": X, "task": "name"}],
  "total_checks": [running total],
  "critical": [current count],
  "high": [current count],
  "medium": [current count],
  "low": [current count],
  "cycle": [current cycle number],
  "no_progress_count": [current count]
}
```

### Step 7: Log the Cycle

Append to `.shiploop/log.txt`:

```
--- Cycle [N] ---
Task: [task name]
Work Result: [summary]
Verify Result: PASS/FAIL/PARTIAL
Issues: [critical] critical, [high] high, [medium] medium, [low] low
Health: [previous] -> [current]
Estimated duration: [rough estimate or omit]
---
```

### Step 8: Check Stop Conditions

**Stop and proceed to Phase 5 if ANY of these are true:**

1. **Convergence:** Health score >= 95 AND zero critical issues AND zero high issues
2. **No Progress:** `no_progress_count` >= 3 (three cycles where health didn't improve)
3. **Max Cycles:** Total cycles >= 15
4. **User Input Needed:** A task requires information not in context.md

**If stopping due to no-progress or max-cycles:**
Log why you stopped. Note remaining tasks. Proceed to Phase 5 anyway — partial progress is still progress.

**If stopping due to user input needed:**
Pause and ask the user the specific question. After they answer, update context.md and resume.

### Step 9: Loop

If no stop condition is met, go back to Step 1.

## Checklist Evolution

During verification, if an issue is found that the universal checklist didn't cover:

1. Categorize why it was missed:
   - `new_category` — the checklist doesn't have this category
   - `edge_case` — the category exists but this edge case wasn't listed
   - `project_specific` — this only applies to this project type
   - `interaction` — this involves two features interacting

2. Write a new check to `.shiploop/checklist-additions.md`:
```
- [CHECK TEXT] (auto-added cycle [N], reason: [category])
  - triggered: 0
  - last_triggered: never
```

3. After 5+ runs, prune additions that never triggered.

## Health Score Display

When reporting to the coordinator (SKILL.md), format the health curve:

```
Health: 72 -> 78 -> 85 -> 91 -> 95
        C1    C2    C3    C4    C5
```

This visual makes progress tangible.

## Important Notes

- Never skip verification. Even if you're confident, verify.
- Never let the work subagent verify its own work. Separation is the point.
- If the same issue keeps failing after 2 fix attempts, mark it [BLOCKED] and move on.
- Log EVERYTHING. The log is the audit trail.
- The health score should generally trend upward. If it drops, something regressed — investigate.
