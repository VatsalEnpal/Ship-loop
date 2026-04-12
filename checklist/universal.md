# Universal Quality Checklist

Mark checks that don't apply to the project type as N/A (e.g., keyboard navigation checks for a CLI tool).

60+ checks that apply to ANY project. No project-specific checks here — those get generated during Phase 2 and stored in `.shiploop/checklist-additions.md`.

Severity guide:
- **CRITICAL** — Blocks the user entirely. App crashes, data loss, security hole.
- **HIGH** — Significant problem. Bad experience, confusing flow, wrong output.
- **MEDIUM** — Noticeable issue. Workaround exists but shouldn't be needed.
- **LOW** — Polish. Nice to have. Won't stop anyone.

---

## 1. First Impression (8 checks)

- [ ] **README exists and explains what this is** in the first 2 sentences (HIGH)
- [ ] **Install/setup instructions work** when followed exactly as written (CRITICAL)
- [ ] **The project runs** on first attempt after setup (CRITICAL)
- [ ] **No errors on first launch** — console, terminal, or UI (HIGH)
- [ ] **A new person can figure out what to do** within 30 seconds (HIGH)
- [ ] **The main action is obvious** — not buried in menus or flags (MEDIUM)
- [ ] **The project name/branding is consistent** across files (LOW)
- [ ] **License file exists** if the project is public (LOW)

## 2. Core Functionality (8 checks)

- [ ] **The primary feature works end-to-end** (CRITICAL)
- [ ] **Output is correct** — data, calculations, rendering match expectations (CRITICAL)
- [ ] **Changes persist** where expected — saves, state, database writes (HIGH)
- [ ] **Undo/back works** where applicable (MEDIUM)
- [ ] **Multiple sequential operations work** — not just the first one (HIGH)
- [ ] **Concurrent usage doesn't corrupt state** if applicable (HIGH)
- [ ] **The feature matches what was described** in the plan/spec (HIGH)
- [ ] **No placeholder or TODO content** visible to users (MEDIUM)

## 3. States & Edge Cases (10 checks)

- [ ] **Empty state is handled** — no data, no input, fresh install (HIGH)
- [ ] **Loading state is shown** during any operation > 200ms (MEDIUM)
- [ ] **Error state is helpful** — tells the user what happened AND what to do (HIGH)
- [ ] **Long content doesn't overflow** or break layout (MEDIUM)
- [ ] **Special characters handled** — quotes, ampersands, unicode, emoji (MEDIUM)
- [ ] **Extremely long input handled** — doesn't crash, truncates or warns (MEDIUM)
- [ ] **Rapid repeated actions don't break things** — double-click, double-submit (HIGH)
- [ ] **Network failure handled gracefully** if applicable (HIGH)
- [ ] **Missing/deleted resources handled** — 404s, missing files, removed items (HIGH)
- [ ] **Boundary values work** — zero, one, max, negative if applicable (MEDIUM)

## 4. Keyboard & Accessibility (7 checks)

- [ ] **All interactive elements reachable by Tab** (HIGH)
- [ ] **Focus is visible** — you can see where you are (HIGH)
- [ ] **Enter activates buttons and links** (MEDIUM)
- [ ] **Escape closes modals and dropdowns** (MEDIUM)
- [ ] **Tab order is logical** — follows visual layout (MEDIUM)
- [ ] **Text is readable** — sufficient contrast, reasonable size (HIGH)
- [ ] **Screen reader basics** — images have alt text, form fields have labels (MEDIUM)

## 5. Performance (5 checks)

- [ ] **Initial load is fast** — under 3 seconds on reasonable hardware (HIGH)
- [ ] **Operations feel responsive** — no unexplained delays (MEDIUM)
- [ ] **No memory leaks on repeated usage** — app doesn't slow down over time (MEDIUM)
- [ ] **Large datasets handled** — performance doesn't degrade badly with scale (MEDIUM)
- [ ] **No unnecessary network requests** or repeated computations (LOW)

## 6. Voice & Tone (5 checks)

- [ ] **Error messages are human** — not stack traces, not "Error: ENOENT" (HIGH)
- [ ] **Language is consistent** — same term for the same thing everywhere (MEDIUM)
- [ ] **Tone matches the project's personality** — per context.md (MEDIUM)
- [ ] **No developer jargon** exposed to non-developer users (MEDIUM)
- [ ] **Spelling and grammar are correct** (LOW)

## 7. Code Quality (8 checks)

- [ ] **No TypeScript/type errors** if typed language is used (HIGH)
- [ ] **No lint errors** if linter is configured (MEDIUM)
- [ ] **No console.log/print debugging** left in code (LOW)
- [ ] **No commented-out code blocks** (LOW)
- [ ] **Functions are reasonably sized** — under ~50 lines (MEDIUM)
- [ ] **Naming is clear** — variables, functions, files describe what they do (MEDIUM)
- [ ] **No code duplication** of significant logic (MEDIUM)
- [ ] **Error handling exists** — try/catch, error boundaries, result types (HIGH)

## 8. Security Basics (6 checks)

- [ ] **No hardcoded secrets** — API keys, passwords, tokens in source (CRITICAL)
- [ ] **User input is validated** before use (HIGH)
- [ ] **No SQL injection** if database queries exist (CRITICAL)
- [ ] **No XSS vectors** if rendering user content (CRITICAL)
- [ ] **Authentication checked** on protected routes/endpoints (HIGH)
- [ ] **Sensitive data not logged** — passwords, tokens, PII (HIGH)

## 9. Documentation (5 checks)

- [ ] **README has setup instructions** that actually work (HIGH)
- [ ] **API endpoints are documented** if applicable (MEDIUM)
- [ ] **Configuration options are documented** (MEDIUM)
- [ ] **Common errors have documented solutions** (LOW)
- [ ] **Architecture is explained** for contributors (LOW)

## 10. Self-Generating Exploration (variable)

These checks are generated dynamically based on the project. During Phase 2, ask:

- "What would a first-time user try to do?"
- "What's the most complex workflow in this project?"
- "What data could a user have that might break this?"
- "What happens when external services are unavailable?"
- "What would someone with accessibility needs experience?"
- "What would happen if a user did things in an unexpected order?"
- "What assumptions does this code make that could be wrong?"

Generate 5-10 project-specific checks from these questions. Store them in `.shiploop/checklist-additions.md`.

---

## Checklist Evolution

This checklist improves over time. Here's how:

### When a bug is found that wasn't covered:

1. **Categorize the miss:**
   - `gap` — No check existed for this category of issue
   - `edge_case` — Category exists but this specific case wasn't listed
   - `project_specific` — Only applies to this type of project
   - `interaction` — Involves two or more features interacting
   - `environment` — Depends on OS, browser, config, or runtime

2. **Write a new check** in `.shiploop/checklist-additions.md`:
   ```
   - [ ] [CHECK DESCRIPTION] (severity)
     added: cycle [N], reason: [category]
     triggered: 0
     last_triggered: never
   ```

3. **On each run,** read `.shiploop/checklist-additions.md` and include those checks.

### Pruning

After 5+ complete ShipLoop runs on the same project:
- Checks that have NEVER triggered → remove them (they're noise)
- Checks that triggered 3+ times → consider promoting to universal.md
- Track hit rate per check to measure checklist effectiveness

### Metrics

Keep these stats in `.shiploop/checklist-stats.json`:
```json
{
  "total_runs": 0,
  "total_bugs_found": 0,
  "bugs_caught_by_checklist": 0,
  "bugs_missed_by_checklist": 0,
  "checks_added": 0,
  "checks_pruned": 0,
  "hit_rate": 0.0
}
```

The hit rate (bugs caught / total bugs) shows how good the checklist is getting. Over time, it should approach 1.0 for a given project.
