---
name: codebase-documenter
description: >
  Documents any codebase, directory, or file into a structured AGENTS/ memory
  directory so agents can answer questions, navigate code, and execute tasks with
  full context. Use when asked to document a codebase, index a repo, build agent
  memory, create documentation for an agent, or update docs after code changes.
  Run once per repo to bootstrap, then incrementally as code changes.
---

# Codebase Documenter

Produces a structured `AGENTS/` memory directory from any codebase, directory,
or file. The output is consumed by `codebase-navigator` at question time.

**Companion skill**: `codebase-navigator` — install alongside this skill.
This skill WRITES the memory. The navigator READS it.

---

## Purpose

Three goals this documentation serves:

1. **Explain code** — agents can describe any module, formula, or system to
   developers and non-technical users in plain English
2. **Debug failures** — agents can reason from a production symptom to a root
   cause using failure signatures and hazard maps
3. **Write features** — agents have the context they need to write code that
   fits the existing system's conventions, contracts, and constraints

---

## Phase 1: Determine Scope

| Input | Approach |
|---|---|
| Entire repo | Full pipeline — all phases (Phase 3.7 only if notebooks detected) |
| Directory / module | Phases 3, 3.5, 3.7 (if notebooks), 4, 4.5, 5, 6 — scoped to that directory |
| Single file | Phase 4 only (contract extraction) |
| Already-indexed repo, new files added | Phase 7 — incremental update only |

Ask if unclear: *"Should I document the whole repo or a specific part?"*

---

## Standing Rule: Always Fetch Today's Date

Whenever you write a `_Last updated: {DATE}_` field in any AGENTS document,
**never guess or hallucinate the date**. Run this command first and use the output:

```bash
python3 -c "import datetime; print(datetime.date.today())"
```

This applies at initial documentation time, during Phase 7 incremental updates,
and any time a document is revised.

---

## Phase 2: Choose Output Format

**Before doing any file reading, decide whether to produce a flat `AGENTS.md` or
the full `AGENTS/` directory structure.**

Use the flat format when ALL of the following are true:
- Fewer than ~15 source files in scope
- Fewer than 3 distinct modules with their own interfaces
- No non-trivial domain calculations or reusable decision logic
  (no `calculate_`, `score_`, `rate_`, feature/label builders, thresholds,
  eligibility rules, model/scoring helpers, policy decisions, etc.)
- Not called by multiple higher-level workflows, pipelines, services, or agents
- Single contributor or very small team — playbooks add no value
- The whole output will fit comfortably under ~150 lines

Use the full `AGENTS/` directory when ANY of the following are true:
- 15+ source files
- 3+ modules with distinct interfaces or contracts
- Domain calculations that need their own indexed section
- The scope is small but semantically important: it owns reusable business rules,
  feature/label/model logic, scoring, transformations, policy decisions, thresholds,
  validation rules, or orchestration helpers called by higher-level workflows
- Other docs need to link here as a canonical subsystem memory via
  `AGENTS/memory_index.md`
- Multiple contributors who need playbooks
- The flat format would exceed ~150 lines

**Sensible default**: small and boring utilities can be flat. Small but important
modules should use full directory format when future agents will need contracts,
business logic, source-read landmarks, or stable links from higher-level docs.
This applies broadly to pipeline helpers, service/domain modules, ML helpers,
rules engines, data transformations, and any module whose behavior is reused
outside its own folder.

**If flat format**: produce a single `AGENTS.md` at the repo root using the
`agents-entry` schema from `references/document-schemas.md`. This file includes
YAML frontmatter so the navigator can load it as the agent memory entry point.
`AGENTS.md` in flat format IS the agent entry point — stop here. Do not continue
to Phase 3 or the Final Step.

**If full directory**: check whether an `AGENTS.md` already exists at the repo
root. Two sub-cases:

- **No existing `AGENTS.md`**: create one after the full pipeline completes,
  using the `agents-readme` schema. This is the human-readable index for the
  `AGENTS/` directory — no frontmatter, just clear prose and a module index
  pointing into the directory.

- **Existing `AGENTS.md` (promoted from flat format)**: rewrite it entirely as
  the directory README using the `agents-readme` schema. Remove the YAML
  frontmatter — it is no longer the agent entry point. The content shifts from
  self-contained memory to a current, accurate index of the `AGENTS/` directory.

In both cases `AGENTS.md` remains a first-class file — always current, always
useful to a human browsing the repo. Then continue to Phase 3.

---

## Phase 2.5: Choose Memory Topology

**Before scanning source in depth, choose how nested documentation should be
handled.**

Use **summary-up, detail-local** for deep or modular repos:
- The repo is 3+ directory layers deep
- A subdirectory already has a meaningful `AGENTS/` memory
- Subsystems have distinct ownership, business domains, or test commands
- Copying every lower-level contract into root would make root docs exceed
  the quality checklist line limits

In this topology:
- Root `AGENTS/` is a routing layer: concise map, hazards, global playbooks,
  and `memory_index.md`
- Subdirectory `AGENTS/` remains canonical for that subsystem's contracts,
  business logic, narratives, and local playbooks
- Root docs include summaries and pointers, not duplicated function-level detail
- Cross-subsystem links point to the canonical subsystem docs and exact source files

Use **root-canonical merge** only when the lower-level memory is small enough to
merge without bloating root docs, or when the user explicitly asks for a single
root memory. In that topology, Phase 4.2 may merge lower-level contracts upward.

Output for deep repos → `AGENTS/memory_index.md`
Format → `references/document-schemas.md#memory-index`

---

## Phase 3: Reconnaissance

**Goal**: Gather structural reconnaissance data without reading every file.

```bash
find . -type f \( -name "*.py" -o -name "*.ts" -o -name "*.tsx" \
  -o -name "*.js" -o -name "*.go" -o -name "*.rs" \) \
  | grep -v node_modules | grep -v .git | grep -v __pycache__ \
  | head -200
```

Also check for:
- `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml` → language/runtime
- `docker-compose.yml`, `.env.example` → services and environment
- `Makefile`, `justfile` → task runner commands
- `.github/workflows/` → CI/CD, test commands, deploy steps
- Existing `AGENTS.md`, `CLAUDE.md` → do NOT duplicate, reference them

Identify: entry points, data model location, module boundaries.

Also run these to detect Databricks notebooks:

```bash
grep -rl "^# %%" . --include="*.py" | grep -v __pycache__ | grep -v .git
grep -rl "^# COMMAND ----------" . --include="*.py" | grep -v __pycache__ | grep -v .git
find . -type f -name "*.ipynb" | grep -v .git | grep -v .ipynb_checkpoints
```

If this returns files, Phase 3.7 will run. Hold the file list in memory.

Also run this to detect subdirectory AGENTS/ directories from prior documentation runs:

```bash
find . -mindepth 2 -type d -name "AGENTS" | grep -v .git | grep -v __pycache__
```

If this returns directories (e.g., `./libs/AGENTS`), their contracts will be
routed or consolidated according to Phase 2.5 in Phase 4.2. Hold the list in memory.

Hold all other findings in working memory — `AGENTS/00_map.md` is written in Phase 4.5
after contract files exist so the Module Index links are accurate.

---

## Phase 3.5: Business Logic & Narrative Extraction

Two outputs. Both serve goals 1 and 2 (explain + debug).

### Business Logic Index

Scan any directory whose name suggests shared or reusable code. Common Python
conventions include: `utils/`, `util/`, `helpers/`, `helper/`, `lib/`, `libs/`,
`common/`, `core/`, `shared/`, `services/`, `service/`, `domain/`, `src/`. Skip
directories that don't exist. Filter by functions whose names or docstrings contain
domain-significant terms: `calculate_`, `compute_`, `derive_`, `score_`, `rate_`,
`margin_`, `forecast_`, `roi`, `ltv`, `churn`, `threshold`, `feature`,
`features`, `label`, `train`, `training`, `predict`, `prediction`, `infer`,
`inference`, `score`, `scoring`, `evaluate`, `metric`, `auc`, `f1`, `precision`,
`recall`, `lift`, `calibration`, `drift`, `baseline`, `backtest`, or any term
that maps to a concept in this domain.

For each: extract the formula in plain English, return edge cases, business
concept it implements, model/feature meaning if applicable, and its `Produces:`
output if it generates a named artifact.

Output → `AGENTS/02_business_logic.md`
Format → `references/document-schemas.md#business-logic`

### Module Narratives

For each module: write a plain-English description covering what it does in
business terms, why it exists as a separate module, what it explicitly does NOT
handle, and what a failure looks like from outside — error signatures, log
patterns, user-visible symptoms.

Output → `AGENTS/03_narratives.md`
Format → `references/document-schemas.md#module-narratives`

---

## Phase 3.7: Databricks Notebook Pipeline Extraction

**Runs when**: Phase 3 reconnaissance found Databricks notebook source files:
`.py` files containing `# %%` or `# COMMAND ----------`, or `.ipynb` notebooks.
Skip this phase entirely if none were found.

**Goal**: Produce dense structured summaries of Databricks pipeline notebooks and
the data flow graph between them. Do NOT load full notebook content — scan for
landmark lines only. An agent must be able to answer structural, logical, and
business-level pipeline questions from these two files — stage roles, data flow,
business purpose, dependencies, and failure signatures. For questions that require
reading actual transformation code, direct the agent to the specific notebook file
by name (listed in `04_pipeline_stages.md`) rather than loading all notebooks.

**Scan all notebooks before writing either file.** Collect inputs, outputs,
called helper functions, ML artifacts, widgets, and library dependencies across
the full set of notebooks first. Do not finalize `04_pipeline_stages.md` or
`05_pipeline_dag.md` until after Phase 4 and Phase 4.2 have created or routed
contracts for the helper modules. Stage docs must link notebook orchestration to
the canonical helper contracts; if a helper contract is missing, flag it as an
unresolved documentation gap.

**Databricks Workflows note**: Execution order and task dependencies come from the
Databricks Workflows job definition — not from anything in the notebook code itself.
Do NOT infer stage sequence from `dbutils.notebook.run()` calls or file names.
Document stage order as defined by the job.

### Reconnaissance: Workflows job definition

Before scanning notebooks, locate and read the Databricks Workflows job definition
(usually `jobs/*.json`, `jobs/*.yml`, `.databricks/bundle.yml`, or a `databricks.yml`
at the repo root; also check Databricks Asset Bundle resources under
`resources/*.yml` and `resources/**/*.yml`). This is the authoritative source for:
- Stage execution order and task dependencies
- Conditional task execution rules (run-if conditions, task success/failure branches)
- Widget default values per environment (dev vs prod job configs)
- Cluster assignment per task

If no job definition file is found, note this — stage order will be inferred from
notebook filenames and must be manually verified.

### Landmark scanning approach (Azure Databricks / Spark)

For each notebook, scan for these patterns to build the summary skeleton:
- `spark.read`, `spark.table(`, `.load(`, `spark.sql(` → Delta table inputs
- `.write`, `.saveAsTable(`, `.insertInto(`, `MERGE INTO`, `CREATE TABLE`,
  `CREATE OR REPLACE TABLE`, `CREATE VIEW` → Delta table or view outputs
- `mlflow.start_run`, `mlflow.log_`, `mlflow.sklearn.log_model`,
  `mlflow.pyfunc.log_model`, `mlflow.register_model`,
  `MlflowClient().transition_model_version_stage` → MLflow runs, metrics,
  model artifacts, and registry state changes
- `dbutils.widgets.get(` → runtime widget parameters
- `dbutils.jobs.taskValues.get/set` → cross-task values not visible in tables
- `import lib.`, `import libs.`, `from lib import`, `from libs import`,
  `import utils.`, `from utils import`, `import helpers.`, `from helpers import`,
  imports from repo-specific nested packages → helper dependencies
- Calls into imported helper modules, e.g. `feature_utils.build_features(...)`,
  `modeling.train_model(...)`, `scoring.score_batch(...)` → called contracts
- `display(` → significant output checkpoints
- Databricks source markers `# COMMAND ----------`, `# MAGIC %md`, and
  `# %% [markdown]` → natural section headers, use as structure
- `%run ./path/to/notebook` → notebook dependency; document separately from
  Python helper imports

### Output 1: Stage docs

One entry per notebook. Each entry: stage name and pipeline role, inputs (tables,
widget params, upstream deps), outputs (tables written, models registered, artifacts),
helper imports and called helper functions linked to canonical contracts, cell-level
summary of significant operations only (not every cell), and failure signature.
Treat the notebook as an orchestrator: it wires widgets, Spark reads/writes, MLflow,
and calls into lower-level helper modules. Put business formulas and reusable logic
in helper contracts and `02_business_logic.md`, not in the notebook stage summary.
If important business or ML logic is inline in the notebook rather than a helper,
add a `02_business_logic.md` entry pointing to the exact notebook section and mark
the stage operation as `inline logic`. Do not hide inline logic inside a vague
stage summary.

Output → `AGENTS/04_pipeline_stages.md`
Format → `references/document-schemas.md#pipeline-stage`

### Output 2: DAG

Cross-notebook execution graph: ordered stage sequence, data flow table (which tables
each stage produces vs. consumes, traced by table name across notebooks), helper
dependency map per stage, ML artifact flow, task-value flow, and critical path
(stages whose failure blocks downstream work because their outputs, models, or
task values have downstream consumers).

Output → `AGENTS/05_pipeline_dag.md`
Format → `references/document-schemas.md#pipeline-dag`

**What is deliberately excluded from Phase 3.7:**
- Playbooks — running notebooks is a Databricks Jobs concern, not a code task
- Hazard map entries — covered by existing helper module hazards
- Function contracts — notebooks export no callable interface

---

## Phase 4: Contract Extraction

**Goal**: For each non-trivial module, extract what an agent needs to call it
correctly. Do NOT extract implementation details.

A contract captures what is NOT obvious from the function signature:
- Required invariants before calling
- Side effects (writes, events emitted)
- Return shape surprises (returns None instead of raising, etc.)
- Env dependencies read at import time
- Idempotency
- `Produces:` — named business output this function originates
- `Consumed by:` — what calls this, if it sits at a key junction
- `Failure modes:` — what errors mean at the system level
- `Read source when:` — task conditions where docs are not enough and the agent
  should open the exact source file/section

One contract note per function/endpoint with non-obvious behavior.

Output → `AGENTS/contracts/{module_name}.md`
Format → `references/document-schemas.md#contract-note`

---

## Phase 4.2: Subdirectory Documentation Consolidation

**Runs when**: Phase 3 found subdirectory `AGENTS/` directories. Skip entirely if none.

**Goal**: Preserve lower-level documentation without flattening useful subsystem
detail. Use the topology chosen in Phase 2.5:

- **summary-up, detail-local**: keep subdirectory `AGENTS/` as canonical; write
  root routing entries and local manifests
- **root-canonical merge**: merge subdirectory docs into root `AGENTS/` while
  preserving detail

**Processing order — shallowest first**: Sort the detected subdirectory AGENTS/ dirs
by path depth, shallowest before deepest. Example: process `libs/AGENTS/` before
`libs/core/AGENTS/`. This ensures that a mid-level directory that already consolidated
its children (e.g., `libs/AGENTS/` which absorbed `core/` in an earlier run) is merged
into root before deeper directories are visited.

**Condensed local refs — skip them for root-canonical merge**: Before merging any contract file, check if its
first line contains `→ Full contract:`. If it does, the file is a condensed local
reference written by a prior Phase 4.2 step — the actual content has already been
consolidated upward into a parent AGENTS/. Skip this file entirely; do not copy the
pointer into root. The real content will arrive via the parent directory's merge.

For each subdirectory AGENTS/ found (e.g., `libs/AGENTS/`), in shallowest-first order:

### Summary-up, detail-local mode

Use this by default for deep repos.

#### Step 1: Read local memory entry points

Read `{subdir}/AGENTS/00_agent_instructions.md` if present; otherwise read the
subdirectory `AGENTS.md` if it has YAML frontmatter. Then inspect only the local
map, business-logic index, and contract headings needed to create a root routing
summary. Do not copy every function note into root.

#### Step 2: Write or update subsystem manifest

Write `{subdir}/AGENTS/manifest.md` using the `subsystem-manifest` schema. It
must list:
- What this subsystem owns
- Canonical docs within this subsystem
- Source entry points an agent should read for implementation details
- Named artifacts produced here
- Upstream/downstream subsystem dependencies
- Local test commands, if different from root

#### Step 3: Add root routing entries

Write or update `AGENTS/memory_index.md` using the `memory-index` schema. Include
one row per subsystem with:
- Scope/path
- What it owns
- Canonical local docs
- Source entry points
- Produced artifacts
- When the navigator should route into that subsystem

Root `AGENTS/00_map.md` should include the subsystem in the Module Index, but the
Key contracts column should point to the subsystem's canonical contracts, e.g.
`→ ../libs/pricing/AGENTS/contracts/discounts.md`, not a duplicated root contract.

#### Step 4: Link, do not duplicate

Where root docs mention a lower-level contract, business concept, or hazard, link
to its canonical subdirectory document. Keep root summaries short. If a root-level
contract needs to describe cross-subsystem behavior, include only the orchestration
contract and link to each subsystem's local contract for implementation detail.

### Root-canonical merge mode

Use only when Phase 2.5 selected root-canonical merge.

#### Step 1: Merge contracts

Read all existing contracts from `{subdir}/AGENTS/contracts/*.md` in full.
These were written with focused knowledge of that module — do not discard them.

Merge into root `AGENTS/contracts/`:
- Module not yet in root → copy content; update file paths to repo-root-relative
  (e.g., `utils.py:42` inside `libs/` becomes `libs/utils.py:42`)
- Module already in root from Phase 4 → merge both:
  - Functions only in subdir contract → add to root file
  - Functions in both → keep more detailed entry; pull in any failure modes,
    invariants, or non-obvious behavior the root version is missing
  - Update all file paths to repo-root-relative in the merged result

#### Step 2: Merge business logic, narratives, and hazards

Read `{subdir}/AGENTS/02_business_logic.md`, `03_narratives.md`, `01_hazards.md`.

For each document:
- Entries not yet in the root equivalent → add them, updating file paths
- Entries in both → keep the more detailed version; never truncate an `Expected range`,
  failure mode, `Read source when`, or hazard that appears in the subdir version but
  not the root version

#### Step 3: Write the subdirectory redirect

Write `{subdir}/AGENTS.md` — tells agents and humans where the full docs live and
confirms no detail was dropped during consolidation.
Format → `references/document-schemas.md#subdirectory-redirect`

#### Step 4: Leave condensed local contract references

For each merged contract, rewrite `{subdir}/AGENTS/contracts/{module}.md` as a
condensed local reference. This is not a duplicate — it is a local-context supplement.
It should contain:
- A pointer to the canonical root contract: `→ Full contract: ../../AGENTS/contracts/{module}.md`
- Any detail that is specifically useful when working inside the subdirectory:
  local-relative file paths for quick navigation, local test invocation patterns,
  subdir-specific edge cases or import conventions

Format → `references/document-schemas.md#subdirectory-local-contract`

Agents and humans working inside the subdirectory get useful local context. In
root-canonical merge mode, root `AGENTS/contracts/` remains the authoritative
single source of truth. In summary-up, detail-local mode, the subdirectory
`AGENTS/` remains authoritative for that subsystem.

---

## Phase 4.5: Finalize Structural Map

**Goal**: Write `AGENTS/00_map.md` now that the contract files from Phase 4 exist.

Use the structural info collected during Phase 3 reconnaissance (stack, entry
points, module list, data model). For the "Key contracts" column in the Module
Index, list only `→ contracts/{module}.md` links that were **actually created**
in Phase 4 or canonical subsystem contract links registered in `memory_index.md`.
Do not speculate or pre-fill links for contract files that do not exist yet.
Modules with no contract file: write `—` in the Key contracts column.

If Phase 3.7 ran, also populate the Environment Configuration section using the
Workflows job definition data collected during that phase — catalog names by
environment, cluster policy, Key Vault scope, and widget defaults that differ
between dev and prod.

Output → `AGENTS/00_map.md`
Format → `references/document-schemas.md#structural-map`

---

## Phase 5: Hazard Map

**Goal**: Document what will break silently, what the agent must NEVER do.

**🔴 NEVER** — causes data loss, security holes, or irreversible state
**🟡 CAUTION** — works but has gotchas
**⚪ CONVENTION** — team patterns not enforced by code

Source from: `# NOTE:`, `# HACK:`, `# WARNING:`, `# FIXME:` comments,
exception handlers with non-obvious logic, migration files, high-churn files.

Output → `AGENTS/01_hazards.md`
Format → `references/document-schemas.md#hazard-map`

---

## Phase 6: Task Playbooks

**Goal**: Step-by-step instructions for the most common agent tasks in this repo.

Start by identifying what kind of repo this is, then pick the right default set:

**If primarily a Databricks notebook pipeline** (Phase 3.7 ran and most code is notebooks):
1. Add a pipeline stage
2. Add a transformation to an existing stage
3. Trigger a Workflows job manually
4. Run a single stage manually in Databricks
5. Promote a model from staging to production

**If a web service / general Python project**:
1. Add a new API endpoint
2. Add a database migration
3. Add a new test
4. Run the test suite
5. Deploy / build

In both cases, add more playbooks for any task that is codebase-specific,
non-obvious from the directory structure, or has been a source of friction
(e.g. "seed the local DB", "add a background job", "regenerate API client",
"add a new model feature").

Each playbook: exact commands, exact files to create/modify, validation step,
common failure modes + fixes.

Output → `AGENTS/playbooks/{task_name}.md`
Format → `references/document-schemas.md#playbook`

---

## Phase 7: Incremental Updates

When code changes:
- New file in documented module → update that module's contract file
- New `# HACK:` or `# WARNING:` → add to `01_hazards.md`
- New domain calculation in utils → add to `02_business_logic.md`
- New module added → create its contract file in `contracts/`; add narrative to `03_narratives.md`; add row to both `00_map.md` and `AGENTS.md` module indexes
- New subsystem added or existing subsystem boundary changed → create/update
  `{subdir}/AGENTS/manifest.md`; update root `AGENTS/memory_index.md`,
  `00_map.md`, and `AGENTS.md`
- New CLI command / Makefile target → update playbook
- Renamed module → update `00_map.md` + `AGENTS.md` module index + its contract filename + all cross-references that point to it
- Module removed → remove from `00_map.md`, `AGENTS.md`, its contract file, and any cross-references pointing to it
- New notebook stage added to pipeline → create entry in `04_pipeline_stages.md` (landmark scan only); update Stage Sequence and Data Flow table in `05_pipeline_dag.md`; re-evaluate critical path
- Notebook stage removed → remove its entry from `04_pipeline_stages.md`; update Stage Sequence, Data Flow, and critical path in `05_pipeline_dag.md`

Procedure: read changed files → identify affected AGENTS/ documents →
surgical edits only → update `last_updated` timestamp by running
`python3 -c "import datetime; print(datetime.date.today())"` (never guess).

---

## Final Step: Generate Agent Instructions (directory format only)

After all other documents are written, generate `AGENTS/00_agent_instructions.md`.

This step only applies to the full directory format. Flat format stops at Phase 2
with `AGENTS.md` serving as the agent entry point.

This file is the entry point for `codebase-navigator`. It must accurately
reflect every document actually produced — do not reference files that
were not generated.

Format → `references/document-schemas.md#agent-instructions`

---

## Output Structure

```
AGENTS.md                         # Human-readable index (always present)
AGENTS/
├── 00_agent_instructions.md      # Entry point for codebase-navigator
├── 00_map.md                     # Where things are
├── 01_hazards.md                 # What not to do
├── 02_business_logic.md          # What things calculate / mean
├── 03_narratives.md              # What modules do and why
├── memory_index.md               # Deep-repo routing to subsystem AGENTS/ docs
├── 04_pipeline_stages.md         # Per-notebook stage docs (notebooks only)
├── 05_pipeline_dag.md            # Cross-notebook data flow graph (notebooks only)
├── contracts/
│   └── {module}.md               # How to call things correctly
└── playbooks/
    └── {task}.md                 # How to do tasks in this repo
```

`AGENTS.md` is always at the repo root regardless of format:
- Small repo (flat): it IS the agent memory (with frontmatter)
- Larger repo (directory): it is the human-readable index of `AGENTS/` (no frontmatter)

Commit both `AGENTS.md` and the `AGENTS/` directory to version control.

---

## Quality Checklist

- [ ] `AGENTS.md` — present at root; correct schema for format (entry vs. readme)
- [ ] `AGENTS.md` — if directory format: module index matches `00_map.md`, no frontmatter
- [ ] `00_map.md` — every module listed, test/deploy commands accurate; Key contracts column matches files actually in root `contracts/` or canonical subsystem contract links from `memory_index.md` (`—` for modules with no contract file)
- [ ] `01_hazards.md` — every hazard specific ("never call X" not "be careful")
- [ ] `02_business_logic.md` — every domain calculation has formula in plain English; deduplication pass against contracts done (no entry fully duplicates a contract note)
- [ ] `03_narratives.md` — every module has a "what it does NOT do" section and a failure signature
- [ ] `memory_index.md` — for deep repos, every subsystem with local AGENTS/ memory has a routing row, canonical docs, source entry points, produced artifacts, and route-when triggers
- [ ] `04_pipeline_stages.md` — (if notebooks present) every stage has inputs, outputs, helper dependency links to canonical contracts, MLflow/model artifacts where applicable, task values where applicable, and failure signature
- [ ] `05_pipeline_dag.md` — (if notebooks present) Workflows task dependencies are represented as a graph, not assumed linear; intermediate tables appear in both Produced by and Consumed by columns; ML artifacts/models and task values are tracked; no intermediate output is missing; critical path identified
- [ ] `contracts/` — every note has something non-obvious; omit if obvious from signature
- [ ] `contracts/` — key functions have `Failure modes:` populated
- [ ] `contracts/` — key functions include `Read source when` guidance and exact source landmarks when implementation details are likely to matter
- [ ] `playbooks/` — every playbook mentally traced end-to-end
- [ ] `00_agent_instructions.md` — memory index matches files actually produced
- [ ] No document exceeds ~150 lines (split by module if needed)
- [ ] Subdirectory redirect `AGENTS.md` written in each consolidated subdirectory (if any)

Read `references/document-schemas.md` for all output format specifications.
Read `references/memory-taxonomy.md` for the theoretical framework.
