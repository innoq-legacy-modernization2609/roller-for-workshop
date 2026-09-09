# Step: Deep Dive

**Goal:** Targeted depth for the selected **work item** — an improvement from the Step-3 improvement backlog or a feature from the feature register. Where Step 2 measured the whole scope coarsely, the deep dive measures only the affected components precisely, maps the behavioral surface the change will touch, and turns both into a validated, actionable implementation plan guarded by **hard, testable guardrails** — the work item's characterization tests and its executable metric gates. Implementation follows in Step 5, gated by these guardrails; after validation (Step 6), return to Step 3 with fresh metrics.

**Prerequisite:** `03-issue-analysis.md` must exist and record the selected work item — an improvement marked „selected for this iteration" in the improvement backlog, or a selected feature in the feature register. The improvement card (or register row) defines scope, success metric, and blast radius.

## 1. Confirm scope and success criteria

- Re-read the improvement card / register row: scope, expected benefit, success metric, blast radius, and the characterization-test-effort estimate.
- Check the feature register for coupling (its rules 1–3): features scheduled on components in scope → combine or sequence explicitly; features blocked by the tech debt this improvement addresses → name them as beneficiaries.
- Record what „done" means for this iteration: the success metric, plus any coupled feature the work item must enable.
- Take stock of the existing guardrails (tests, checks, metric thresholds already present in the repo) and judge whether they — together with the characterization tests and metric gates this step defines — are sufficient to make the intended change mechanically verifiable. If they are not, extending the guardrails becomes part of the implementation plan, not an afterthought.

## 2. Deep metrics for the affected components only

Where Step 2 collected coarse distributions, collect precise per-component values now:

- **Complexity** — per class and per method in scope; name the top offenders with values.
- **Coupling / cohesion** — the import graph of the classes to be changed: fan-in/fan-out, hidden dependencies (reflection, config lookups, thread pools, static singletons).
- **Precise test coverage** — line and branch coverage for the affected classes where the toolchain allows; explicitly name the untested paths the plan will depend on.
- **State** — which runtime state each touched component reads/writes (database, caches, filesystem, search index, session); ownership and lifetime of each.
- **Maintenance signals refresh** — churn and bus-factor for the affected files, re-run for the current analysis window.
- **Guardrail baselines** — for every metric that will gate this work item (§5), record the exact, reproducible measurement command and its current value. These baselines are what the metric gates ratchet against; a metric without a reproducible command cannot gate anything.

## 3. Map the behavioral surface

- Enumerate the *observable* behavior the change will touch: request→response contracts, events, state transitions, output artifacts — not internal structure.
- **Define the characterization tests for this work item separately — never for a whole application.** Every work item gets its own test definition, scoped to exactly the change surface of that item (the flows, classes, and contracts it will touch). The improvement card's characterization-test-effort estimate becomes this concrete definition here.
- **Characterization tests are hard guardrails, not documentation.** Each test is specified so it can be implemented immediately: entry point, inputs, pinned expected outputs (captured from current behavior), determinism control, pass/fail semantics. They are implemented as the first work package (§5), must be green before any behavior change starts, run on every build of the iteration, and survive as regression tests afterwards — they are the net that makes unintended behavior change visible.
- For work items whose deliverable *is* the net (test-net improvements): list the exact flows to pin (entry points, inputs, relevant outputs) and the determinism controls needed (caches, asynchronous processes, clocks/locales). A flaky characterization test is worse than none — state how each control is achieved.
- Distinguish behavior that must be **preserved** from behavior that will **intentionally change** (coupled features): the former gets pinned and must survive; the latter gets pinned first, then deliberately broken, so the test diff documents the intended change.

## 4. Update the risk assessment

- Re-assess the improvement card's risks against the deep data: retire, confirm, or add risks, each with evidence.
- Confirm the blast radius using the import graph from §2.

## 5. Draft the implementation plan

- Work packages small enough to be verified independently, sequenced so the safety net comes first: **package 0 implements the work item's characterization tests (defined in §3) and is done only when they are green** — no behavior change starts before that gate passes.
- For each package: what changes, how it is verified (test, metric, command), and which part of the success metric it serves. Verification is a **hard gate**: the package counts as done only when all its guardrails pass — characterization tests green (failures allowed only where §3 marked behavior as intentionally changing) and metric gates within their thresholds.
- **Metric gates are executable, not narrative.** Each gate is an exact command with a pass/fail threshold, recorded with its output in the step logfile. Ratchet semantics against the §2 guardrail baselines: no guardrail metric may degrade (e.g. complexity of changed classes must not increase, coverage of the change surface must not decrease); an iteration may tighten a gate, never loosen it.
- Abort / rollback criteria per package, and the point of no return if one exists. A guardrail that fails and cannot be satisfied aborts the package — a guardrail is never adjusted to make a change pass; the failure returns to interactive validation (§6).

## 6. Interactive validation

- Present the deep-dive findings, the guardrails (characterization tests and metric gates), and the implementation plan to the user. The user validates findings, adjusts scope, sequence, and guardrails, and green-lights implementation.
- Record the decisions. Implementation follows in Step 5 (gated by the guardrails defined here) and validation in Step 6; when validation is finished, return to Step 3: increment the iteration, re-run the relevant Step-2 metrics, and re-evaluate the improvement backlog and the feature register together.

## Report format

Persist the outcome as `modernization-knowledge/04-deep-dive--<work-item-slug>.md` (e.g. `04-deep-dive--rendering-test-net.md`):

```
## Deep Dive — <target name>

**Input:** 03-issue-analysis.md (improvement card #<n>) [or feature register (F<n>)]
**Date:** <date>
**Iteration:** <n>

### Scope confirmation
- scope / success metric / coupled features

### Deep metrics
- complexity, coupling, coverage, state per affected component (tables)

### Behavioral surface
- work-item characterization test definition (scoped to this item's change surface)
- flows to preserve / flows to change deliberately
- determinism controls

### Guardrails
- characterization tests: entry point, inputs, pinned outputs, status (implemented / green)
- metric gates: metric, exact command, threshold, §2 baseline value, ratchet direction

### Updated risk assessment
- risks with evidence + mitigations, confirmed blast radius

### Implementation plan
- ordered work packages: package 0 (characterization tests green), then change, verification gate, abort criteria

### Validation decisions
- user feedback, adjustments, green light
```

The general rules from SKILL.md apply throughout — explicit negative results, evidence over assumption, glossary terms with their exact meanings, outcome persisted before implementation begins.
