# Step: Implementation

**Goal:** Execute the deep dive's implementation plan for the selected **work item** — package by package, in the planned order, with the guardrails as hard gates. The step succeeds when every planned package is done, every guardrail passes (characterization tests green except the intended diffs, metric gates within their thresholds), and the run is logged so completely that Step 6 (Validation) can re-verify everything by measurements instead of trusting this log.

**Prerequisite:** `04-deep-dive--<work-item-slug>.md` exists and records the green light: the user-validated implementation plan (package 0, gates, abort criteria) and the validation decisions. If scope or sequence need to change mid-implementation, stop — the change goes back through Step 4 §6 and is re-validated before work continues.

## 1. Package 0: build the safety net

- Implement the work item's characterization tests exactly as defined in the deep dive (§3): entry points, inputs, pinned expected outputs, determinism controls.
- Package 0 is done only when all of them are **green against the unchanged code** — they pin current behavior, they do not judge it.
- If a test cannot be implemented as specified or cannot be made deterministic, do not weaken it to proceed: record the problem, apply the abort criteria, and return to Step 4 (§6). A flaky characterization test is worse than none.

## 2. Execute the remaining packages in order

- One package at a time, in the planned sequence. Per package: make the change, then run the package's verification (test, metric, command).
- A package counts as done only when all its guardrails pass: characterization tests green — failures allowed only where the deep dive marked the behavior as intentionally changing — and metric gates within their thresholds.
- Record every gate run in the step logfile with the exact command and its full output, so Step 6 can re-run it independently instead of trusting this log's history.

## 3. Guardrail discipline

- A guardrail is never adjusted to make a change pass. If a guardrail fails and cannot be satisfied: abort the package, apply the abort / rollback criteria from the deep dive, and return to Step 4 (§6) with the failure as evidence.
- If the failed package has a point of no return, the failure goes to the user for an explicit decision before anything else changes.
- No scope creep: only the planned packages are implemented. A newly discovered issue is recorded as a candidate for the next Step-3 pass, not fixed in passing.

## 4. Implementation log

- Keep a running log with one entry per package: what changed (files, rough LOC), which gates ran (exact commands), gate results, deviations from the plan and the decision that authorized them, aborts / rollbacks if any.
- Explicit negative results belong here too (e.g. "no metric gate degraded", "no unplanned file touched").

## 5. Handoff to validation

- After the last package: summarize the gate results (all green, or the intended diffs listed), set the work item status accordingly, and hand over to Step 6 (Validation) — which re-measures everything independently.

## Report format

Persist the outcome as `modernization-knowledge/05-implementation--<work-item-slug>.md`:

```
## Implementation — <work item name>

**Input:** 04-deep-dive--<work-item-slug>.md (implementation plan, validation decisions)
**Date:** <date>
**Iteration:** <n>

### Package log
| # | Package | Change | Gates run (commands) | Gate results | Deviations |
|---|---------|--------|----------------------|--------------|------------|
| 0 | characterization tests | … | … | green | — |
| 1 | … | … | … | … | … |

### Guardrail results
- characterization tests: green / intended diffs (list them)
- metric gates: metric, command, threshold, baseline, measured, verdict

### Deviations from the plan
- what diverged, why, and the decision that authorized it

### Result
- packages done, work item status, handoff to Step 6
```

The general rules from SKILL.md apply throughout — explicit negative results, evidence over assumption, glossary terms with their exact meanings, every gate run logged with exact command and output.
