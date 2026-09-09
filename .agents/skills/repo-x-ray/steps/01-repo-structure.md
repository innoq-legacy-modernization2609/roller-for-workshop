# Step: Repository Structure Analysis

**Goal:** Analyze a repository's structure in two passes: first the repository as a whole (top-level view only), then each module individually. Determine layout classification, build system and dependency manager, infrastructure and CI setup, documentation coverage, license, and package counts. This is pure filesystem analysis — no build or execution required.

**Outcome:** A completed repository structure report (see Report format), persisted as `modernization-knowledge/01-repo-structure.md` **inside the analyzed repository's root** — create the directory if it does not exist. The basis for all following steps: a clear understanding of the repository's structure and key components. The artifact name is "01-repo-structure".

## Prerequisites

Run cloc once at the repository root (excluding build output and vendored dependencies) and keep the output — Part 1, Part 2, and the report reference it. See the recommended-tooling notes in SKILL.md for why cloc is preferred.

## Part 1 — Repository-level analysis (top level only)

As output of part 1 I expect you to know all the info mentione in the template on Repository level.

### 1.1 Map the top level

Inspect the root directory listing, **including hidden files**. Note which top-level directories and files exist. Descend into modules only as far as you need to describe what roughly is in there.

Identify the applications and other top-level technical components (including infrastructure, docs, and tooling folders) and give **each of them a one-sentence description or less**, e.g. "Spring Boot application source", "Docker image build for deployment", "Selenium integration tests".

### 1.2 Classify the repository layout

Apply these rules **in order** — the first match wins:

| Classification | Criteria |
|---|---|
| **Standard (single project)** | One application at the root: `src/` (incl. `src/main`, `src/test`), optional `docs/`, exactly **one** root build file (`pom.xml`, `build.gradle`, `package.json`, …) that builds only itself |
| **Multi-module build** | One root build file that declares submodules — Maven `<modules>`, Gradle `settings.gradle` includes — with sibling module folders (e.g. `app/`, `db-utils/`) |
| **Monorepo** | Dedicated `apps/`, `packages/`, `libs/`, `services/`, or `modules/` directories at the root, **or** multiple independent build files without a shared root |
| **Other / legacy** | None of the above — describe what you actually see |

Always confirm by inspecting the root build file(s) — Maven `<modules>` section, `workspaces` in `package.json`, `include(...)` in `settings.gradle(.kts)`. Never classify on directory names alone.

### 1.3 Identify infrastructure

Look for dedicated infrastructure-as-code / deployment folders (`k8s/`, `kubernetes/`, `terraform/`, `helm/`, `charts/`, `docker/`, `deploy/`, `infrastructure/`) and deployment files at the root (`Dockerfile`, `docker-compose.yml`, `Chart.yaml`, `*.tf`). For each hit, note what it deploys (container image, cluster manifests, compose stack). If nothing matches, state explicitly: **"No dedicated infrastructure found."** Do not silently omit this.

### 1.4 Identify CI configuration

Look for CI/CD pipeline definitions — usually at the root or in a hidden directory: `.gitlab-ci.yml`, `.github/workflows/`, `Jenkinsfile`, `.circleci/`, `azure-pipelines.yml`, `bitbucket-pipelines.yml`, `.travis.yml`, `buildspec.yml`, `.drone.yml`. Identify the CI system, what the pipeline does (build, test, static analysis, deploy), and where the config lives. If no CI configuration exists, state it explicitly — common and relevant in legacy repos.

### 1.5 Top-level documentation

Check the root for a `README`, `CHANGELOG`, `CONTRIBUTING`, `AUTHORS`, `NOTICE` and a `docs/` or `doc/` directory. Quantify documentation volume from the cloc output (Markdown, AsciiDoc, RST, plain-text lines). Check if this is process-related documentation (like backlogs, resolved tickets) or product documentation. Defer the inspection of documentation inside the module directories until Part 2. Classification if the repo has documentation centralized or distributed for each module can be estimated from the cloc output.

### 1.6 Top-level license

Look for `LICENSE`, `COPYING`, and `NOTICE` files at the root, and check the root build file for license declarations (Maven `<licenses>`, npm `license` field). Identify the license type from the file text — if ambiguous, quote the identifying line instead of guessing. Per-module deviations are checked in Part 2.

### 1.7 Note patterns and conventions

Note any patterns and conventions visible at the repository level that deviate from the expected — naming conventions for modules and directories, shared build or deployment conventions, code style files (`.editorconfig`, Checkstyle, Spotless, Prettier, …), and recognizable commit or branch conventions. If nothing notable, say so explicitly.

## Part 2 — Per-module analysis

Run the following checks **for every module/application** identified in 1.2. Do not aggregate findings across modules — report each module separately.

### 2.1 Build system and dependency manager

Determine per module which build file it uses (`pom.xml`, `build.gradle(.kts)`, `package.json`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, `*.csproj`). Identify the exact dependency manager variant via wrappers and lockfiles — `mvnw`/`.mvn` (Maven Wrapper), `gradlew` (Gradle Wrapper), `package-lock.json` vs. `yarn.lock` vs. `pnpm-lock.yaml` (npm vs. Yarn vs. pnpm), `poetry.lock` vs. `uv.lock` vs. `requirements.txt` (Poetry vs. uv vs. pip), `Cargo.lock`, `go.sum`. Note version pinning where visible, and state explicitly when **no lockfile or wrapper exists** — a reproducibility risk, common in legacy projects. List also which artifacts a module produces and which other modules it depends on.

### 2.2 Module layout conformance

Compare each module's internal structure against its ecosystem's standard layout:

| Ecosystem | Expected standard |
|---|---|
| JVM / Maven / Gradle | `src/main/java`, `src/main/resources`, `src/test/java` (+ optional `src/it` for integration tests) |
| JavaScript / TypeScript | `src/`, tests in `test/` or `__tests__/`, build output in `dist/` |
| Python | package directory + sibling `tests/` |

Report deviations explicitly — production-only modules (no tests at all), test-only modules, non-standard source roots, generated code directories. State conformant modules explicitly as well; do not leave them implicit.

### 2.3 Module documentation and license deviations

Per module, check for an own `README` and own `LICENSE`/`NOTICE` files. An own license in a subfolder often indicates bundled third-party code under a different license — critical for compliance. Flag deviations from the root license.

### 2.4 Count packages

For each application, count and report **separately**: (a) *build modules* (Maven modules, npm workspaces, Gradle subprojects) and (b) *source packages* (e.g. Java package directories under `src/main/java`). Never aggregate these into a single number, and never aggregate across applications.

### 2.5 Identify main building blocks and relationships

For each component, identify its main building blocks and how they relate — this is the component's *structure*. For a JVM application this is typically the package tree and its layering (e.g. service-layer packages called by controller packages, persistence in a data-access package); for other ecosystems, the equivalent grouping (layer folders, feature modules, library boundaries). Describe each block in one sentence and name the notable relationships between blocks and between components.

## Report format

Return the findings in this exact structure:

```
## Repository Structure
### Repository level
- Layout: <standard | multi-module | monorepo | other> — <one-sentence justification>
- Applications: <list; for monorepos every entry under apps/ or equivalent>
- Components: <all top-level technical components with their one-sentence description>
- Languages: <from cloc — top languages with code lines, total code lines>
- Comment ratio: <comments/code in % — notable outliers>
- Infrastructure: <folder(s)/file(s) with purpose> | none found
- CI: <system, pipeline purpose> | none found
- Documentation: <none | minimal | central | distributed | hybrid> — <where the mass lives, formats, volume from cloc>
- License: <type> — <source(s) of truth>, <consistency notes>
- Patterns/conventions: <notable conventions> | none notable

### Per module
- <module>: build <system + dependency manager, lockfile: yes/no>, layout <conformant | deviation>, docs <own README: yes/no>, license <same as root | deviates>
- …
- Building blocks: <component — main blocks and notable relationships>

- Packages per application:
  - <app>: <n> build modules, <m> source packages
- Size distribution: <optional>
```

The general rules from SKILL.md apply throughout — explicit negative results, evidence-based classification, and one cloc run reused everywhere.
