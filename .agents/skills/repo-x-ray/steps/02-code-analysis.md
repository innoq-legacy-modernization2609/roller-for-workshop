# Step: Code Analysis (overview, scoped)

**Goal:** For a chosen application or module (the *scope*), produce an overview across three pillars — Structure, Behavior, State — plus baseline metrics and a maintenance-health snapshot, after verifying the scope builds and its tests pass. This is **breadth, not depth**: identify what is there and where the hotspots are, so Issue Analysis (Step 3) can decide what to look at closely.

Unlike Step 1, this step **executes** build and test commands. I strongly expect you to verify your findings and to avoid all kind of guesswork. Please collect the calling command and the output of every tool call in a separate file as .txt.

**Scope parameter:** one application or module, taken from `01-repo-structure.md`. One run = one scope. For multiple targets, trigger separately. Record the scope in the outcome file name: `modernization-knowledge/02-code-analysis--<scope>.md`.

**Boundary context:** open `01-repo-structure.md` and pull the module dependency graph for the scope — upstream (what the scope consumes) and downstream (who consumes the scope = blast radius). This is the map for any later impact assessment; do **not** assess change impact here.

## 1. Healthcheck

Verify the scope builds and its tests pass before anything else. Record build status, warnings, test pass rate, and timing. If build or tests fail, stop and record the failure — deep analysis on a non-building system is unreliable, and the failure itself is a finding.

Per-ecosystem build + test commands (run only the scope's module, not the whole repo):

| Ecosystem | Build | Test |
|---|---|---|
| Java (Maven) | `mvn -q -DskipTests -pl <module> -am compile` | `mvn -pl <module> -am test` |
| Java (Gradle) | `./gradlew :<module>:classes` | `./gradlew :<module>:test` |
| Kotlin | same as Java (Maven/Gradle) | same |
| Python | `python -m compileall -q <pkg>` or `pip install -e .` | `pytest <pkg>` (or `python -m unittest`) |
| TypeScript | `tsc --noEmit` (or `npm run build -w <pkg>`) | `npm test -w <pkg>` / `jest` |
| JavaScript | `npm run build -w <pkg>` | `npm test -w <pkg>` |

## 2. Structure (components from static analysis)

Identify **components** — logical groupings of code with a responsibility — *not* packages. Group source packages/folders into components (e.g. presentation, business/domain services, persistence, integration/API, utilities) and size each by LOC and class/file count. For a monolith, state "1 process, components: …"; for multiple processes/services, state "n processes, together ~<LOC>".

- **Java / Kotlin:** group by package clusters under the source root; size each cluster (class count + LOC). Treat `.kt` and `.java` together where both exist.
- **Python:** group top-level packages/modules into components; size by file count + LOC.
- **TS / JS:** group by source folders / feature modules; size by file count + LOC.

Goal of this section: a component map with sizes — not a package listing.

**Component dependency diagram:** derive the inter-component dependency graph from import statements (Java/Kotlin) / import graph (Python) / require-import graph (TS/JS). Render it with Graphviz (`dot`) as SVG+PNG to `modernization-knowledge/diagrams/components.{dot,svg,png}`. Count bidirectional edges — each pair is a cycle and a structural smell.

## 3. Behavior

Identify the behavioral patterns/archetypes present around the system — patterns, not traces. Look for: web request handling, CLI, scheduled/batch jobs, event-driven/message handling, rendering/templating pipeline, API exposure (REST/SOAP/XML-RPC/gRPC), data pipeline/ETL, real-time/streaming. List the ones present with a one-line description each and the owning component.

## 4. State

Locate where mutable state lives: relational DB (ORM/SQL), search index, HTTP session, cache (in-memory/external), filesystem, message queue, in-memory singletons. For each, name the owning component. This is the "where state is" map — quantify per-component load in the deep dive, not here.

## 5. Baseline metrics

Compute coarse, scope-wide distributions (not per-class deep numbers) as a baseline for later deep dives:

- **Complexity:** distribution summary + top hotspots. Tools — Java/Kotlin: ckjm or SonarQube (Kotlin also: detekt complexity); Python: radon (`cc`, `mi`); TS/JS: eslint `complexity` rule / complexity-report.
- **Coupling & cohesion:** afferent/efferent coupling where tooling exists (Java/Kotlin: ckjm; Python: approximate via the import graph; TS/JS: dependency-cruiser / madge). Summarize, do not enumerate. **Count bidirectional edges in the component dependency graph and record directional weights per pair** — a cycle where one direction is 336 imports and the reverse is 1 is a stray import (trivial), while 40 vs 24 is genuine mutual coupling. Distinguish *lopsided* (reverse count is small — likely a stray import to prune) from *balanced* (comparable both ways — genuine cycle). List the cyclic pairs with their weights and this classification. Do **not** pre-judge severity or priority here — that is Step 3's job; report the data and the lopsided/balanced classification only.
- **Best-practice violations:** run a static analyzer and report total count + top categories + distribution per component. Tools — Java: PMD, Checkstyle, SpotBugs; Kotlin: detekt, ktlint; Python: ruff, pylint, bandit; TS/JS: eslint. Also run copy-paste detection (CPD for Java/Kotlin; equivalent for others) and report duplication count + total duplicated lines.
- **Maintenance health:** CI status (from Step 1), commit recency (`git log` last-commit dates), TODO/FIXME density, dependency freshness (Maven versions plugin / `pip list --outdated` / `npm outdated`), license/header hygiene. Give a short verdict — well-kept / mixed / neglected — with evidence.
- **Change hotspots & bus-factor (git history):** derive three signals from `git log`, scoped to a recent meaningful window (e.g. the last 5 years): (1) change-frequency hotspots — the most-modified files in the scope (proxy for defect likelihood); (2) bus-factor per component — the share of files touched by only one author (knowledge-concentration risk); (3) predictive hotspots — files that are *both* frequently changed *and* bus-factor=1 (highest modernization risk, the primary input for Step 3). No tool install needed — git is universal. State the commit count, time window, and top author concentration.
- **Test coverage (rough):** test-file vs. main-file ratio (from cloc / Step 1) and coverage % if easily available (Java/Kotlin: JaCoCo; Python: coverage.py; TS/JS: jest/nyc). Precise per-component coverage is deferred to the deep dive.

## Report format

Return the findings in this structure:

```
## Code Analysis — <scope>

**Scope:** <application or module> (from 01-repo-structure.md)
**Ecosystem:** <language, build system>

### Boundary context
- Upstream: <modules the scope consumes>
- Downstream (blast radius): <modules that consume the scope>

### Healthcheck
- Build: <ok / failed, warnings, time>
- Tests: <passed / failed / skipped, pass rate, time>

### Structure
- <n> process(es), ~<total LOC>
- Components:
  - <component>: <one-line responsibility>, ~<LOC> LOC, <n> classes/files
  - …

### Behavior
- <pattern> — <one line, owning component>
- …

### State
- <location> — <owner component>
- …

### Baseline metrics
- Complexity: <distribution + top hotspots>
- Coupling/cohesion: <summary, bidirectional-edge cycles>
- Best-practice violations: <total, top categories, per-component distribution>
- Code duplication: <CPD count + total duplicated lines>
- Change hotspots & bus-factor: <top change-frequency files, BF=1 per component, predictive hotspots>
- Maintenance health: <verdict + evidence>
- Test coverage (rough): <ratio, % if available>

### Notable findings (for Issue Analysis)
- <bullets feeding Step 3>
```

The general rules from SKILL.md apply throughout — explicit negative results, evidence-based, glossary terms used exactly, one cloc run reused. Persist as `modernization-knowledge/02-code-analysis--<scope>.md` inside the analyzed repository's root.
