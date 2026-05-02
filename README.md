# Agent Skills Guide

This repo contains two companion skills for building and using structured agent
memory in codebases:

- `codebase-documenter` writes the memory.
- `codebase-navigator` reads the memory to answer questions and guide code work.

The intended outcome is a repo-local `AGENTS/` memory layer that helps agents
understand a codebase without repeatedly scanning everything from scratch.

## Why This Exists

Large codebases, especially Databricks projects, are hard for agents to navigate
from source alone. The useful context is spread across notebooks, helper modules,
workflow definitions, model training code, table writes, configuration, and local
team conventions.

These skills create a structured memory layer that answers:

- Where is the relevant code?
- Which module owns this behavior?
- How do I call a helper safely?
- What business rule, feature, metric, or model logic does this implement?
- What notebook stage produced this table or model?
- What can break silently?
- When should the agent stop reading docs and open source?

The goal is not to replace source code. The goal is to route agents to the right
docs and then to the exact source files only when implementation detail is needed.

## The Two-Skill Model

### `codebase-documenter`

Use this skill when creating or updating codebase memory.

It scans a repo, directory, or module and writes:

- `AGENTS.md` for human orientation
- `AGENTS/00_agent_instructions.md` as the agent entry point
- `AGENTS/00_map.md` for structure and entry points
- `AGENTS/01_hazards.md` for risks and conventions
- `AGENTS/02_business_logic.md` for formulas, rules, features, labels, metrics
- `AGENTS/03_narratives.md` for module explanations
- `AGENTS/contracts/` for callable behavior
- `AGENTS/playbooks/` for repeated tasks
- `AGENTS/04_pipeline_stages.md` for notebook stages, when present
- `AGENTS/05_pipeline_dag.md` for pipeline/table/model flow, when present
- `AGENTS/memory_index.md` for deep repos with subsystem-local memories

### `codebase-navigator`

Use this skill when asking questions or making changes in a documented repo.

It reads the memory first, routes to the right document, and reads source only
when the docs point to exact source landmarks or the task needs implementation
detail.

## Core Concepts

### Contracts

Contracts document how to call code correctly.

They capture things that are not obvious from a function signature:

- required invariants
- return edge cases
- side effects
- idempotency
- environment dependencies
- named outputs via `Produces`
- important callers via `Consumed by`
- failure modes
- `Read source when` landmarks

Use contracts for questions like:

- Can this helper be retried?
- What does this function return on empty input?
- What table or event does this write?
- What production symptom maps to this failure?

### Business Logic

`02_business_logic.md` documents what calculations and rules mean.

For Databricks and ML projects this includes:

- feature definitions
- label windows
- scoring logic
- thresholds
- metric definitions
- drift or calibration checks
- model selection rules
- expected ranges
- leakage guards

Use this for questions like:

- How is this feature calculated?
- What does this score mean?
- What range is normal?
- What makes this metric suspicious?

### Narratives

`03_narratives.md` explains modules in plain English.

A good narrative says:

- what the module owns
- why it exists
- what it does not handle
- what a failure looks like from outside

Use this for onboarding, orientation, and explaining system behavior to humans.

### Hazards

`01_hazards.md` documents what can break, especially things that are not enforced
by code.

Examples:

- never write directly to a published Delta table
- dev and prod catalogs differ
- a helper silently drops rows on malformed input
- a model registry alias has deployment impact
- a test requires a specific cluster or secret scope

### Playbooks

Playbooks are step-by-step instructions for common tasks.

Examples:

- add a pipeline stage
- add a new feature builder
- run a subset of tests
- promote a model
- deploy a bundle
- update agent memory after a refactor

Playbooks should include exact commands, files to edit, validation steps, and
common failure modes.

## Databricks Project Structure

For Databricks repos, notebooks are treated as orchestration. Helper modules hold
the reusable logic.

Notebook docs should answer:

- what stage this is
- what inputs it reads
- what outputs it writes
- which widgets it uses
- which helper functions it calls
- which MLflow artifacts or models it writes
- which task values it reads or sets
- which notebook sections matter

Helper module docs should answer:

- how to call the function safely
- what feature, label, metric, or rule it implements
- what edge cases matter
- what failures mean
- when source must be read

## Recommended Workflow For Large Repos

Run the documenter bottom-up, then run root last.

1. Document reusable helper modules first.

   Examples:

   - `libs/features/`
   - `libs/modeling/`
   - `libs/scoring/`
   - `utils/`
   - `helpers/`

2. Document mid-level subsystems next.

   Examples:

   - `pipelines/customer_churn/`
   - `pipelines/forecasting/`
   - `src/data_quality/`

3. Run the root pass last.

   The root pass should mostly create routing and pipeline context:

   - `AGENTS/memory_index.md`
   - `AGENTS/00_map.md`
   - `AGENTS/04_pipeline_stages.md`
   - `AGENTS/05_pipeline_dag.md`

This avoids a single huge root-first scan and preserves lower-level detail.

## Flat Versus Directory Format

Small, self-contained utility folders can use a flat `AGENTS.md`.

Small but important modules should use full `AGENTS/` even if they have few
files. Use full directory format when the module owns:

- reusable business rules
- feature or label logic
- scoring or training helpers
- thresholds
- validation rules
- policy decisions
- transformations called by notebooks or services
- behavior that root docs need to link to

## Layered Memory

Deep repos should avoid copying all lower-level docs into root.

Preferred structure:

```text
AGENTS/
  00_agent_instructions.md
  00_map.md
  memory_index.md
  04_pipeline_stages.md
  05_pipeline_dag.md

libs/features/AGENTS/
  manifest.md
  02_business_logic.md
  contracts/*.md

libs/modeling/AGENTS/
  manifest.md
  02_business_logic.md
  contracts/*.md
```

Root `memory_index.md` routes agents to canonical subsystem docs. Subsystem
`manifest.md` explains ownership, local docs, source entry points, produced
artifacts, dependencies, and local validation.

## Reviewer Checklist

Before accepting these skills, review whether they enforce the right behavior:

- They document callable behavior as contracts, not prose summaries.
- They separate business logic from call contracts.
- They treat notebooks as orchestration unless logic is explicitly inline.
- They track Databricks table flow, model flow, task values, and workflow graph.
- They preserve lower-level subsystem memory instead of flattening everything.
- They include source-read landmarks for implementation-sensitive behavior.
- They keep root memory small enough to route, not large enough to replace source.
- They update packaged `.skill` files when source skill folders change.

## Expected Commit Contents

When updating these skills, commits should usually include both:

- source folders such as `skills/codebase-documenter/SKILL.md`
- packaged files such as `skills/codebase-documenter.skill`

This keeps editable source and installable bundles aligned.
