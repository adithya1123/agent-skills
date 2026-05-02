---
name: codebase-navigator
description: >
  Use for any question about code in this repository — explaining what a module
  does, finding where a business formula lives, debugging a production failure,
  tracing where an output came from, understanding what a function returns, or
  writing new features safely. Also use for Databricks pipeline questions: which
  stage produces a table, why a pipeline stage failed, what the execution order
  is, how a model KPI is calculated end-to-end, or what data flows where.
  Triggers on any code or pipeline question regardless of whether the repo has
  been documented yet.
---

# Codebase Navigator

Answers any code question by reading structured agent memory — either a flat
`AGENTS.md` file or a full `AGENTS/` directory, depending on how the repo was documented.

**Companion skill**: `codebase-documenter` — generates the memory this skill reads.
Install both skills together.

---

## First action — always

**Check which memory format this repo uses.**

**Option B — directory format**: look for `AGENTS/00_agent_instructions.md`.
- If it exists → read it immediately before doing anything else. It contains
  the full memory index and tells you exactly which document to read for any
  question. Do not scan the codebase. Do not read random files. Start there.

**Option A — flat format**: if `AGENTS/00_agent_instructions.md` does NOT exist,
look for `AGENTS.md` at the repo root.
- Check whether it has YAML frontmatter (`---` at the top with `name:` and `description:` fields).
- If it has frontmatter → flat format memory. Read it and use it directly.
- If it has no frontmatter → it is a human-readable directory index, not agent memory.
  Continue to "If neither exists" below.

**If neither exists** → the repo has not been documented yet. Tell the user:
*"This repo doesn't have agent memory yet. Run the `codebase-documenter` skill
on it first — ask me to document this codebase and I'll build the memory.
Then I can answer questions about it efficiently."*
Then offer to answer from source files directly as a fallback, with the caveat
that responses will be slower and less reliable than from documented memory.

---

## The memory structure (directory format)

Map the question to the document type. Read that document. Answer.
Pipeline documents (04, 05) are present only in repos with Databricks notebooks.
Deep repos may also include `AGENTS/memory_index.md`, which routes to canonical
subsystem-local `AGENTS/` memories.

For the flat format (`AGENTS.md`), the same question categories apply — use the
corresponding section heading within the single file instead of a separate document.

| Document | Answers |
|---|---|
| `AGENTS/00_map.md` | Where is X? What module handles X? What are the entry points? What environment/cluster config applies? |
| `AGENTS/memory_index.md` | Which subsystem-local memory owns X? Which local docs/source entry points should I read? |
| `AGENTS/01_hazards.md` | What should I never do? What are the sharp edges? What's the convention? |
| `AGENTS/02_business_logic.md` | What is the formula for X? How is X calculated? What is the expected range for this metric? |
| `AGENTS/03_narratives.md` | What does module X do? Why does it exist? What does it NOT handle? |
| `AGENTS/contracts/{module}.md` | How do I call X? What does it return? What produced output Y? What calls Z? |
| `AGENTS/playbooks/{task}.md` | How do I add X / run tests / deploy in this specific repo? |
| `AGENTS/04_pipeline_stages.md` | What does pipeline stage X do? What tables does it read/write? What columns? Where are its logs? |
| `AGENTS/05_pipeline_dag.md` | Which stage produces table X? What is the execution order? Why didn't stage X run? What are its conditions? What is the critical path? |

---

## How to reason

Any question maps to one of these categories. Do not default to reading
source files — use the memory first.

**For deep repos with `AGENTS/memory_index.md`:**
→ Start with `00_map.md` for root orientation
→ Read `memory_index.md` when the question mentions a subsystem, nested path,
business concept, produced artifact, or cross-subsystem flow
→ Route into the subsystem's `AGENTS/manifest.md`, then its local contracts,
business logic, hazards, or narratives
→ Treat subsystem-local docs as canonical for that subsystem unless root docs
explicitly say they were merged upward

**Explaining code:**
→ Start with `03_narratives.md` for what the module does and why
→ If `memory_index.md` maps the module/path to a subsystem, read that subsystem's
manifest and local `03_narratives.md`
→ Use `02_business_logic.md` for what any calculation means
→ Use `contracts/` for precise behavior of specific functions

**Explaining a pipeline stage:**
→ Start with `04_pipeline_stages.md` — role, inputs, outputs, and key operations for that stage
→ Use `05_pipeline_dag.md` to explain how it fits in the full pipeline flow
→ Use the stage's `Helper dependencies` to find each called helper function
→ If a helper contract is subsystem-local, route through `memory_index.md` and
the subsystem `AGENTS/manifest.md`
→ Use the helper's contract for call behavior and `02_business_logic.md` for
formulas, feature logic, metric definitions, model training/scoring logic, and
business meaning
→ Treat notebooks as orchestration unless the stage entry says `inline logic`;
for inline logic, read the exact notebook section and the matching
`02_business_logic.md` entry

**Debugging a failure:**
→ If the failure is in a notebook pipeline stage → use **Debugging a pipeline stage failure** below
→ Start with `03_narratives.md` — find the failure signature for this module
→ Check `contracts/` for `Failure modes:` on the relevant function
→ Check `01_hazards.md` for known sharp edges in this area
→ Reason backwards: symptom → failure mode → root cause

**Writing new features:**
→ Start with `00_map.md` to orient
→ If `memory_index.md` exists, route to the owning subsystem before editing
→ Read `01_hazards.md` before writing anything
→ Read `contracts/` for the module you're working in; if the module is subsystem-local,
read the subsystem's canonical `AGENTS/contracts/{module}.md`
→ Check `playbooks/` for the task type — follow it if it exists

**Tracing an output:**
First determine what kind of output it is:
- **Delta table** (named `catalog.schema.table` or matches a known table name) →
  Search `05_pipeline_dag.md` Data Flow table for it under "Produced by". Then read
  its producing stage in `04_pipeline_stages.md` for what the stage does.
- **Function artifact** (a named dataclass, report, file, event) →
  Search root `contracts/*.md` and, if present, `memory_index.md` `Produces` entries
  for the artifact name. Route to subsystem-local contracts when the index points
  there.
  Single match → that is the origin function.
  Multiple matches → check `Consumed by:` to find the upstream origin.
  No match → the artifact is undocumented; note this to the user.

**Finding a formula or calculation:**
→ Search `02_business_logic.md` by concept name
→ If `memory_index.md` exists, search its `Owns`, `Produces`, and `Route here when`
columns; then read the subsystem's `02_business_logic.md`
→ If not found → search `contracts/` for function names containing the concept
→ If still not found → the calculation is undocumented; note this to the user

**Debugging a pipeline stage failure:**
→ Read `04_pipeline_stages.md` for that stage — failure signature tells you what the error pattern means and where logs are
→ Check `Helper dependencies` in the stage entry; read `Failure modes:` on every
called helper contract, routing through `memory_index.md` for subsystem-local helpers
→ Check `01_hazards.md` for known Databricks sharp edges
→ If the failure involves MLflow, model registry, task values, or missing tables,
cross-check the matching table/model/task-value flow in `05_pipeline_dag.md`
→ If code-read is needed: stage entry names the exact notebook file AND the `[§ section]` to jump to

**Tracing a full pipeline output end-to-end:**
→ Start at `05_pipeline_dag.md` Data Flow table — trace the output table backwards to its source
→ For ML model outputs, start at `05_pipeline_dag.md` ML Artifact / Model Flow
→ For control values, start at `05_pipeline_dag.md` Task Value Flow
→ For each stage in the chain, read its entry in `04_pipeline_stages.md` (role, inputs, outputs, key operations)
→ For the helper function doing the core computation, read its canonical contract
(root or subsystem-local); formula and feature/model logic live in `02_business_logic.md`
→ For cell-level logic: jump to the `[§ section]` named in Key operations of the stage entry, read that notebook section only
→ Never load all notebooks — always navigate to the specific file and section

**Answering "why didn't stage X run" / conditional execution:**
→ Read `05_pipeline_dag.md` Stage Sequence — check `Depends on` and `Run condition`
for stage X
→ Cross-check `04_pipeline_stages.md` `Runs when` field for the stage
→ If still unclear: the condition is defined in the Workflows job JSON/YAML (not in notebook code)

**Answering "is this value / metric normal":**
→ Read `02_business_logic.md` for the concept — `Expected range` field gives healthy bounds and what deviations signal
→ Cross-check `04_pipeline_stages.md` Outputs `Sanity check` field for the producing stage

**Answering "why does this work in dev but not prod" / environment questions:**
→ Read `00_map.md` Environment Configuration section — catalog names, cluster policy, Key Vault scope, widget defaults by env

**For any question not listed above:**
Identify which category it belongs to from the table above.
Read that document. If `memory_index.md` exists, use it before reading source.
If genuinely ambiguous, read `00_map.md` first — it always points to the right place.

---

## When to read source

Memory is the routing and contract layer; source remains authoritative for exact
implementation. Read source only after memory routing, and use the narrowest
source target available.

Read source when:
- You are editing code
- A contract's `Read source when` item matches the task
- The user asks for exact implementation details, branching, or line-level behavior
- The docs mark an artifact, formula, or failure as undocumented
- The task crosses subsystems and `memory_index.md` or a subsystem manifest points
  to source entry points that must be checked

Prefer these source pointers, in order:
1. `Read source when` landmarks in the relevant contract
2. `Source entry points` in a subsystem `AGENTS/manifest.md`
3. Specific notebook `[§ section]` entries in `04_pipeline_stages.md`
4. File paths in `00_map.md`

Do not scan entire directories unless the memory is missing, stale, or cannot
identify an owner.

---

## After completing a task

If your work introduced anything new, update the memory:

| What you did | What to update |
|---|---|
| New function with a named business output | Add `Produces:` contract note |
| New domain calculation or formula | Add entry to `02_business_logic.md` (include `Expected range`) |
| New module or significant refactor | Add/update `03_narratives.md` |
| New subsystem or moved subsystem boundary | Update `memory_index.md` and the subsystem `AGENTS/manifest.md` |
| New `# HACK:`, `# WARNING:`, `# NOTE:` comment | Add to `01_hazards.md` |
| New CLI command or Makefile target | Update relevant playbook |
| Renamed or moved module | Update `00_map.md`, contract filename, `memory_index.md` if subsystem-local, and any `04_pipeline_stages.md` helper dependency links pointing to it |
| New or modified pipeline stage | Update `04_pipeline_stages.md` and `05_pipeline_dag.md` |
| Delta table renamed or schema changed | Update table names/columns in `04_pipeline_stages.md` and `05_pipeline_dag.md` |
| Workflows job condition changed | Update `Condition` column in `05_pipeline_dag.md` Stage Sequence and `Runs when` field in `04_pipeline_stages.md` |

Keeping the memory current means every future agent — and every future
question — benefits from what you just learned.
