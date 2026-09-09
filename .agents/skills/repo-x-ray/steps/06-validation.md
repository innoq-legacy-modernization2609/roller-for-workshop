# Step: Validation

**Goal:** Validate a completed iteration — the implementation of the deep-dive's work packages — **by measurements, not judgment**. Every success criterion of the work item becomes a metric with a target value, a reproducible measurement procedure, and a measured value. The step document's claims are verified against the code, and the Step-4 baselines are re-measured to quantify the delta. This step closes the iteration loop and hands back to Step 3 with fresh metrics.

**Prerequisite:** The deep-dive step document (`04-deep-dive--<work-item-slug>.md`) exists with its implementation plan and validation decisions, and the implementation step document (`05-implementation--<work-item-slug>.md`) exists with its package log and guardrail results. The implementation of the planned work packages is claimed done.

**Artifact:** `modernization-knowledge/06-validation--<work-item-slug>.md`.

## Core rule: metrics only

The verdict rests exclusively on metrics. Each metric states:

1. **Target** — the value the work item's card, register row, or a recorded validation decision demands, with its source.
2. **Procedure** — how it is measured, described so anyone can re-run it (which report files, which check, which scope). A metric without a reproducible procedure is an opinion.
3. **Measured value** — the result of running the procedure now, during validation.
4. **Verdict** — exactly one of: **met**, **not met**, or **deferred** (only with the recorded decision and date that deferred it; a deferral without a decision is a "not met").

Qualitative impressions ("feels solid", "code looks clean") do not belong in the verdict. If they matter, record them in a clearly separated observations section that is explicitly not part of the verdict. Absence of a check is itself a finding — report explicit negative results ("no drift found", "metric not measurable because …", never silence).

## 1. Metric extraction

- Re-read the improvement card / register row, the deep dive's scope confirmation and validation decisions, and the implementation log (`05-implementation--<work-item-slug>.md`). Extract **every** success criterion — including deferred parts (e.g. umbrella targets postponed by a recorded decision) — and each risk mitigation that was promised as configuration or code.
- Turn each criterion into a measurable expression. If a criterion cannot be measured, that is a finding: either sharpen it now (with the user) or record it as unmeasurable and why.
- Separate the metric families: run metrics (tests, stability), coverage/delta metrics (baselines from the deep dive), size metrics (the new code itself), and claims verification (documentation vs. code).

## 2. Run metrics

- Execute fresh, consecutive build-and-test runs — not just read old reports. The number of runs follows what the deep dive promised (stability claims usually demand at least two consecutive runs; re-verify what was claimed, don't trust the log's history blindly).
- Record: test counts per class, pass rate, failures/errors/skips, per-class and total time. Read them from the build tool's report files when the console summary is ambiguous — but state the source.
- If the iteration introduced ordering variance (test classes that share seeded state), verify the promised order-independence mechanically, e.g. by forcing a different class order, and record both directions' results.

## 3. Claims verification — documentation vs. code, both directions

- Extract every file-and-line citation from both step documents (deep dive and implementation — body, package log, appendices). Verify each mechanically: read the cited lines and check that they still contain what the claim says.
- Report counts: citations **checked**, citations **drifted**, and each drift with **both** locations (document section + code location) and the direction of the drift (code moved, doc imprecise, claim stale).
- Distinguish state descriptions from baseline records: a claim like "0 test classes reference X" written as a pre-implementation baseline is not drift after implementation — it is the baseline; re-measure it as a delta instead (see §4). A claim about the current state that no longer holds is drift.
- Direction of correction: the code states reality, the document states intention. **Never change the code to match the document.** Correct the document (this validation step may do so, recording each correction) or open an issue where the code is wrong. List what implementation revealed that the documents did not yet say — that is the "both directions" part.
- Verify also the negative mechanical claims (a file that must not exist, a config key that must be disabled) — they are drift-prone because nothing breaks when they silently flip.

## 4. Baseline deltas

- Re-measure the deep dive's precise-coverage and baseline statements with the same procedure that produced them, and report before → after deltas: test classes referencing the change surface, per-class coverage where tooling allows, models/components under test.
- Distinguish "changed by design" (the net was the deliverable) from "unchanged by design" (adjacent surfaces explicitly out of the thin slice — name them, they are candidates for the next iteration's umbrella targets).
- Re-verify promised risk controls mechanically (config keys, disabled features, isolation directories) — they are part of what "done" means.

## 5. Size metrics of the delivered work

- Measure the delivered test/code surface itself: files, lines (a tool like cloc gives language-aware counts including comment ratio — recommended when available), and the ratio of shared infrastructure to per-flow test code. A characterization net that is mostly shared scaffolding for few pinned flows is a finding worth reporting.

## 6. Verdict and loop closure

- Assemble the verdict table: metric | target (source) | procedure | measured | verdict. No metric without a row; no verdict without a metric.
- On **not met** or drift: name the follow-up (fix, re-run, or issue) — the validation does not silently absorb failures.
- Close the iteration: re-evaluate the Step-3 improvement backlog with the fresh metrics (retire achieved improvements, update hotspot data), and state what the next iteration's work item should be. That re-entry point is where this step hands control back to Step 3.

## Language coverage

Tooling is ecosystem-dependent and follows the repo-x-ray defaults: build/test reports from Maven Surefire/Failsafe, Gradle Test, pytest, or jest; line-coverage from JaCoCo/coverage.py/nyc where present (absence of coverage tooling is a reportable finding, not a blocker); citation verification and baseline re-measurement need only a way to read specific file lines and count matches. cloc (already recommended by this skill) serves the size metrics.
