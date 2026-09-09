# Step: Issue Analysis

**Goal:** Turn the Notable Findings from Step 2 into a validated issue list and improvement backlog, then select **one** work item for the immediate deep dive. This step is **interactive and iterative**: the agent proposes, the user validates and selects. After the selected work item is implemented and validated (Steps 4–6), the user returns here to re-evaluate the backlog with fresh metrics — no long-term sequence is fixed upfront.

**Prerequisite:** `02-code-analysis--<scope>.md` must exist in the knowledge store.

## 1. Assemble candidate list

Open `02-code-analysis--<scope>.md` and read the *Notable findings* section. Each finding becomes a candidate. If multiple findings share a root cause, group them — one candidate may cover several symptoms.

For each candidate, record:
- **Finding** — the symptom as observed in Step 2 (evidence-based: metric, count, file)
- **Hypothesized root cause** — the underlying issue behind the symptom
- **Component(s) involved** — which Step-2 component(s) this touches
- **Evidence signals** — which Step-2 metrics support it (cycles, PMD, churn, bus-factor, coverage, …)

## 2. Classify: symptom vs. issue

For each candidate, distinguish whether it is an **issue** (root cause worth addressing directly) or a **symptom** (observable effect of a deeper issue). A symptom becomes a finding *under* its issue, not a standalone candidate. Example:
- Symptom: „many bidirectional component cycles"
- Issue: „layer A bypasses the service layer and accesses domain objects directly"

Only issues are kept; symptoms are documented as supporting evidence under their issue.

## 3. Prioritize the improvement backlog

Score each improvement (the proposed way of addressing an issue) on three dimensions (low / medium / high):

- **Impact** — how much does this improvement block or enable modernization? (architectural blocker > isolated smell)
- **Effort** — how large is the change? (LOC affected, components touched, blast radius from Step 1)
- **Risk** — what could go wrong? (test coverage in the affected area, bus-factor=1 concentration, coupling to other components)

Order the backlog by a sensible default: **high impact, low-to-medium effort, manageable risk** first. This ordering is a *proposal*, not a commitment — the user reorders it in the interactive validation (§5, Stage A).

## 4. Define an improvement card for each issue

For every issue in the backlog, write a compact card so the user can make an informed selection:

- **Name** — a short, clear label
- **Issue statement** — the root cause in one or two sentences
- **Scope** — the concrete components/files/packages to change
- **Expected benefit** — what improves (decoupling, testability, tech-stack replacement path, knowledge distribution, …)
- **Effort estimate** — rough size (S/M/L) with the main cost driver
- **Risk assessment** — top 2–3 risks and the main mitigation.
- **Characterization test effort** — estimate (low / medium / high) how much test-net work is needed before the change can be made safely. Base this on the Step-2 data for the affected area: existing test coverage (ratio), complexity baseline, and the size of the public surface to lock in. Low = thin change surface or existing coverage; medium = moderate surface, some coverage; high = large surface, little/no coverage. The precise test plan is designed in Step 4; this estimate informs the selection.
- **Success metric** — how do we know it worked? (e.g. „0 direct cross-layer imports in the affected package", „cycle count reduced")
- **Blast radius** — from Step 1: which downstream modules may be affected

## 5. Interactive validation (two stages)

**Stage A — Validate the issue list and improvement backlog (confirm):**
Present the prioritized backlog with improvement cards to the user. The user:
- confirms or rejects issues and their improvements
- corrects the symptom/issue classification
- reorders the backlog
- adds context the metrics cannot see (business drivers, deadlines, team knowledge)
- may split a large issue into smaller candidates

Record the decisions. The result is the **validated improvement backlog** — a prioritized list of confirmed improvements, not a sequence to execute.

**Stage B — Select one work item for this iteration:**
From the validated improvement backlog (or the feature register), the user selects **exactly one** work item for the immediate deep dive (Step 4). Record it as „selected for this iteration". The remaining items stay in the backlog unchanged — they are not sequenced.

Rationale: implementing one work item changes the codebase, the metrics, and the team's understanding. Fixing a sequence of multiple work items upfront pretends a planability that does not exist. After the selected work item is implemented and validated, the user returns to this step and re-evaluates the backlog with fresh data (re-run the relevant Step-2 metrics, re-classify, re-prioritize).

## Report format

Persist the outcome as `modernization-knowledge/03-issue-analysis.md`:

```
## Issue Analysis — <scope>

**Input:** 02-code-analysis--<scope>.md
**Date:** <date>
**Iteration:** <n> (increment on each return after an implemented work item)

### Candidates from Step 2
| # | Finding (symptom) | Hypothesized root cause | Type | Impact | Effort | Risk |
|---|---|---|---|---|---|---|
| 1 | … | … | issue | high | M | medium |
| 2 | … | … | symptom of #1 | — | — | — |
| …

### Validated improvement backlog
| # | Improvement | Impact | Effort | Risk | Status |
|---|---|---|---|---|---|
| 1 | <name> | high | M | medium | open |
| 2 | <name> | medium | L | high | open |
| …

#### Improvement cards
##### 1. <name>
- **Issue statement:** …
- **Scope:** …
- **Expected benefit:** …
- **Effort:** <S/M/L> — <main cost driver>
- **Risk:** <top risks + mitigation>
- **Characterization test effort:** <low/medium/high> — <basis: coverage ratio + surface size>
- **Success metric:** <how to verify>
- **Blast radius:** <from Step 1>

##### 2. <name>
…

### Selected for this iteration
- **<work item name>** — <one-line reason for the choice>

### Deferred (backlog)
- <improvement names remaining in the backlog, no sequence>

### Next
- Deep dive on the selected work item → Step 4
- After implementation and validation (Steps 5–6): return here (increment iteration), re-run Step-2 metrics, re-evaluate the backlog
```

The general rules from SKILL.md apply throughout — explicit negative results (if a finding yields no issue, say so), evidence-based, glossary terms used exactly. Persist as `modernization-knowledge/03-issue-analysis.md` inside the analyzed repository's root.
