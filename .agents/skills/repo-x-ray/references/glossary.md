# Glossary

These terms have fixed meanings throughout this skill. When writing reports, step outcomes, or knowledge-store entries, use them exactly as defined — no synonyms, no interchange.

Glossary rule: only terms with a fixed local meaning that could be ambiguous globally belong here; each is defined in one sentence, plus a source at most.

| Term                         | Definition                                                                                                                                           |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Application**              | A deployable, self-contained product unit identified by a build artifact (WAR, JAR, executable); may span multiple modules.                          |
| **Baseline metrics**         | Coarse, scope-wide metric distribution computed once as a reference for later deep dives.                                                            |
| **Behavior**                 | How the system acts at runtime (request handling, rendering, API exposure), as opposed to static Structure.                                          |
| **Blast radius**             | The set of modules potentially affected by a change to the scope.                                                                                    |
| **Building block**           | A part of a system (layer, subsystem, feature group) with its relationships to other blocks.                                                         |
| **Bus-factor**               | The number of distinct authors who have touched a file within the analysis window.                                                                   |
| **Characterization test**    | A test that pins the current behavior of a work item's change surface, regardless of correctness (Feathers, _Working Effectively with Legacy Code_). |
| **Churn hotspot**            | A file changed frequently in git history; a proxy for defect likelihood.                                                                             |
| **Component**                | A building block with defined interfaces that follows information hiding — internals are accessed only via the interfaces (Parnas 1972).             |
| **Explicit negative result** | A check that finds nothing is still reported ("none found") — absence is a finding.                                                                  |
| **Feature**                  | An intended change of observable behavior, as opposed to a behavior-preserving improvement.                                                          |
| **Guardrail**                | A hard, testable check that a work package must pass to count as done; either a characterization test or a metric gate.                              |
| **Healthcheck**              | Verification that the scope builds and its tests pass; run at the start of Code Analysis.                                                            |
| **Improvement**              | A validated, proposed solution to one or more issues, with scope, benefit, effort, and risk (aim42: improvement).                                    |
| **Improvement backlog**      | The prioritized list of confirmed improvements — candidates to select from, one per iteration.                                                       |
| **Infrastructure folder**    | A directory holding deployment / infrastructure-as-code artifacts (e.g. `docker/`, `k8s/`, `terraform/`).                                            |
| **Issue**                    | A concrete, evidenced deficiency of the codebase — the root cause, not its symptom (aim42: issue).                                                   |
| **Iteration**                | One pass through select → deep dive → implement → validate → re-evaluate.                                                                            |
| **Layout**                   | The overall repository shape: standard, multi-module, monorepo, or other.                                                                            |
| **Maintenance health**       | Signals of how well the code is cared for: CI status, commit recency, TODO/FIXME density, dependency freshness.                                      |
| **Metric gate**              | An exact measurement command with a pass/fail threshold that ratchets against the deep-dive baseline; may tighten over iterations, never loosen.     |
| **Module (build module)**    | A unit of the build system with its own build file (Maven module, Gradle subproject, npm workspace).                                                 |
| **Optimization**             | A change in observable behavior, but limited to certain aspects.                                                                                     |
| **Outcome**                  | The persisted, self-contained result of one analysis step; the input contract for the following steps.                                               |
| **Predictive hotspot**       | A file that is both a churn hotspot and bus-factor=1; the primary input for Issue Analysis.                                                          |
| **Production-only module**   | A module with sources but no tests.                                                                                                                  |
| **Refactoring**              | A change to internal structure that preserves observable behavior (Fowler, _Refactoring_, 2nd ed.).                                                  |
| **Regression test**          | A test that verifies required behavior and fails on unintended behavior change.                                                                      |
| **Scope**                    | The application or module a scoped step (e.g. Code Analysis) is applied to.                                                                          |
| **Source package**           | A code-level namespace, e.g. a Java package directory under `src/main/java`.                                                                         |
| **State**                    | Where mutable data lives at runtime: relational DB, search index, HTTP session, cache, filesystem, message queue.                                    |
| **Structure**                | The building blocks of a component and their relationships.                                                                                          |
| **Symptom vs. issue**        | A symptom is an observable effect; an issue is its root cause — only issues are analyzed further.                                                    |
| **Test-only module**         | A module containing only tests, e.g. an integration-test module.                                                                                     |
| **Work item**                | The unit of work selected for one iteration: an improvement or a feature.                                                                            |
