---
name: repo-x-ray
description: Structural x-ray of an unfamiliar codebase in two passes — repository level (layout classification. standard vs multi-module vs monorepo; infrastructure folders like docker/k8s/terraform; CI configuration; documentation coverage; license) and per module (build system, dependency manager and lockfiles, layout conformance, package counts). Pure filesystem analysis, no build required. Use when onboarding to, auditing, or planning the modernization of a (legacy) codebase.
metadata:
  version: 1.4.0
---

# Repo X-Ray

You are in the role of an experienced software architect and engineer. Your task is to analyze an unknown code repository.

The analysis proceeds in several steps. Each step delivers an outcome that is persisted in the knowledge store: **`<repo-root>/modernization-knowledge/`** by default — i.e. inside the root of the **analyzed repository**, not the agent's current working directory. One numbered markdown file per step (e.g. `01-repo-structure.md`). Create the directory if it does not exist. If nothing special is defined, write md files to ./modernization-knowledge. Each step defines an artifact name as basename for the file.

1. **Repository Exploration** — Understand the structure and contents of the repository: layout, top-level components with one-sentence descriptions, building blocks and relationships, infrastructure, CI, documentation, and license. Detailed instructions: [steps/01-repo-structure.md](steps/01-repo-structure.md). Outcome: `modernization-knowledge/01-repo-structure.md` — a clear understanding of the repository's structure and key components.
2. **Code Analysis (overview, scoped)** — For one application or module (the _scope_): run a build+test healthcheck, then map Structure / Behavior / State in breadth and record baseline metrics (complexity, coupling/cohesion, best-practice violations, maintenance health, rough test coverage). Detailed instructions: [steps/02-code-analysis.md](steps/02-code-analysis.md). Outcome: `modernization-knowledge/02-code-analysis--<scope>.md`.
3. **Issue Analysis** — Turn Step-2 findings into a validated issue list and improvement backlog, then select **one** work item for the immediate deep dive. Interactive and iterative: the agent proposes, the user validates and selects one. After implementation and validation, return here and re-evaluate with fresh metrics. Detailed instructions: [steps/03-issue-analysis.md](steps/03-issue-analysis.md). Outcome: `modernization-knowledge/03-issue-analysis.md`.
4. **Deep Dive** — Targeted depth (per-component complexity/coupling/cohesion, precise test coverage, state load, behavioral traces) for the selected work item (improvement backlog or feature register), turned into hard, testable guardrails: the work item's characterization tests and executable metric gates that every work package must pass. Detailed instructions: [steps/04-deep-dive.md](steps/04-deep-dive.md). Outcome: `modernization-knowledge/04-deep-dive--<work-item-slug>.md`.
5. **Implementation** — Execute the deep-dive's implementation plan package by package, gated by the guardrails: package 0 implements the characterization tests and must be green before any behavior change; every following package counts as done only when its guardrails pass (characterization tests green except intended diffs, metric gates within thresholds). Guardrails are never adjusted to make a change pass — failures abort the package and return to the deep dive. Detailed instructions: [steps/05-implementation.md](steps/05-implementation.md). Outcome: `modernization-knowledge/05-implementation--<work-item-slug>.md`.
6. **Validation** — Close the iteration by validating the implemented work packages **by measurements, not judgment**: every success criterion becomes a metric (target, procedure, measured value, verdict), all file-and-line citations of the step documents (deep dive and implementation) are mechanically verified against the code (drifts are corrected in the document, never the code), and the Step-4 baselines are re-measured as before→after deltas. The validation outcome is the iteration report and hands back to Step 3 with fresh metrics. Detailed instructions: [steps/06-validation.md](steps/06-validation.md). Outcome: `modernization-knowledge/06-validation--<work-item-slug>.md`.

Step 1 is pure filesystem analysis; from Step 2 onward the skill may execute build and test commands.

## Language coverage

The skill is language-agnostic in structure and covers **Java, Kotlin, Python, and TypeScript/JavaScript**. Concrete tooling is elaborated for Java and Python; Kotlin reuses the JVM tooling with Kotlin-specific linters (detekt, ktlint), and TS/JS uses the npm-ecosystem equivalents (tsc, eslint, jest). Each step notes the per-ecosystem tools.

The skill's steps (especially the Healthcheck and Baseline metrics) depend on the target ecosystem's toolchain being present — e.g. Maven/Gradle, a JDK and JaCoCo for Java/Kotlin; Python and pytest/coverage.py/radon for Python; Node and tsc/jest for TS/JS. **These tools may need to be installed before a step can run.** Install only what the current scope's ecosystem actually requires, prefer standard managers (sdkman for JVM, use pyenv/uv or pip for Python, nvm for Node), and record every installation in the project's `install-log.txt` (date, one-word trigger, full command) so it can be added to the sbx-kit. If a required tool cannot be installed (e.g. dependency resolution blocked by network policy), treat it as a finding: record what failed and proceed with the checks that do not need it, rather than aborting the whole step.

## Recommended tooling

- **cloc** — run it once at the repository root (excluding build output and vendored dependencies) and reuse the output across all checks: language distribution, documentation volume, and comment ratio. cloc is preferred over faster counters like tokei because it reliably recognizes legacy formats (JSP, properties, Ant) and tolerates old non-UTF-8 encodings, which faster tools silently skip — a real risk in legacy codebases.
- A directory-tree listing (e.g. `tree`) helps for a visual overview but is optional.
- **Graphviz (`dot`)** — render component and module dependency diagrams from the import/build graph; outputs SVG and PNG to `modernization-knowledge/diagrams/`. Provides an immediate visual on coupling and cycles — bidirectional edges in the component graph are a strong architectural smell.
- In general, it is up to you to find a successful path to answer, either with specific tools made for the job or with basic file juggling and LLM use. But try in any way to validate the central findings with concrete evidence generated by tools made for the job.

## General rules

- Every check produces an explicit negative result when it finds nothing ("none found", "no lockfile", "no license", "no CI") — never silently omit a check.
- Always justify classifications with concrete evidence (build file contents, directory names), not assumptions.
- The cloc output is reused in multiple report fields (languages, comment ratio, documentation volume) — one run, many uses.
- Each step's outcome is persisted in the knowledge store — `modernization-knowledge/` at the root of the analyzed repository, one numbered markdown file per step — before moving on to the next step. Create the directory if it does not exist.
- In case there are non-trivial patterns found in the project, these should be persisted in `modernization-knowledge/patterns-rules.md`.
- **Follow the glossary ([references/glossary.md](references/glossary.md)):** read it before writing any report or step outcome, and use its terms with exactly the defined meanings in all steps, reports, and knowledge-store entries — no synonyms, no interchange.
- All tool output that contributes to significant findings should be recorded in a text file like a log, with a timestamp at the beginning, exact (reproducible) commands including all flags and the output of the command. Every step will create a separate logfile.

## Glossary

The glossary lives in [references/glossary.md](references/glossary.md), sorted alphabetically. Read it before writing any report, step outcome, or knowledge-store entry, and use its terms with exactly the defined meanings — no synonyms, no interchange.

Glossary rule: only terms with a fixed local meaning that could be ambiguous globally belong in the glossary; each is defined in one sentence, plus a source at most.
