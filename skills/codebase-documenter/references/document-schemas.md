# Document Schemas

All AGENTS/ output documents follow these schemas.
Copy-paste these templates when creating new documents.

> **Date rule**: Every `{DATE}` placeholder must be filled by running:
> `python3 -c "import datetime; print(datetime.date.today())"`
> Never guess or hallucinate the date.

---

## agents-entry

**File**: `AGENTS.md` (repo root — small repo flat format)

Used when the codebase is small enough that a single file is cleaner than a
full `AGENTS/` directory. Includes YAML frontmatter so the navigator loads it
as the agent memory entry point. When the repo grows and promotes to directory
format, this file is rewritten using the `agents-readme` schema below.

```markdown
---
name: codebase-memory
description: >
  Structured memory for this codebase. Load whenever working on any code task.
---

# Agent Instructions
_Last updated: {DATE}_

## Overview
- **Stack**: {language}, {framework}, {DB}
- **Entry points**: {list}
- **Test command**: `{command}`
- **Deploy command**: `{command}`

## Module Map

| Module | Path | Purpose |
|--------|------|---------|
| {name} | `{path}` | {one-line purpose} |

## Hazards

🔴 **NEVER**: {specific thing} — {why}
🟡 **CAUTION**: {specific thing} — {detail}
⚪ **CONVENTION**: {specific thing} — {detail}

## Key Contracts

### {FunctionName}
**File**: `{path}:{line}`
**Non-obvious**: {what's non-obvious — omit if nothing}
**Returns**: {edge cases only}
**Failure modes**: {what errors mean — omit if standard}

## Business Logic

### {ConceptName}
**Function**: `{function_name}` in `{file}`
**Formula**: {plain English}

## How to do common tasks

**Add a new endpoint**: {steps}
**Run tests**: `{command}`
**Deploy**: `{command}`
```

**Rules for the flat format:**
- Keep the whole file under 150 lines. If approaching that limit, the repo
  should use directory format instead — re-run the documenter.
- Use flat format only for small, self-contained scopes that do not need stable
  contract files, source-read landmarks, subsystem manifests, or root
  `memory_index.md` links.
- If a small scope owns reusable business rules, pipeline transformations,
  feature/label/model logic, scoring, thresholds, validation rules, policy
  decisions, or other behavior called by higher-level workflows, use directory
  format instead.
- Omit any section that has nothing to say.
- YAML frontmatter is required — the navigator uses it to identify this as
  the agent entry point.

---

## agents-readme

**File**: `AGENTS.md` (repo root — directory format human index)

Used when the repo has a full `AGENTS/` directory. This file has no YAML
frontmatter — it is not agent memory, it is the human-readable index of the
directory. Always kept current alongside the directory contents.

```markdown
# Agent Memory

This repo's agent memory lives in `AGENTS/`. It is generated and maintained
by the `codebase-documenter` skill and read by the `codebase-navigator` skill.

_Last updated: {DATE}_

## What's in here

| Document | Purpose |
|----------|---------|
| `AGENTS/00_agent_instructions.md` | Agent entry point — read this first |
| `AGENTS/00_map.md` | Module index, entry points, stack, test/deploy commands |
| `AGENTS/01_hazards.md` | NEVERs, CAUTIONs, and team conventions |
| `AGENTS/02_business_logic.md` | Domain calculations and business formulas |
| `AGENTS/03_narratives.md` | Plain-English module descriptions |
| `AGENTS/memory_index.md` | Deep-repo routing index for subsystem-local memories |
| `AGENTS/contracts/` | Per-module call contracts and traceability |
| `AGENTS/playbooks/` | Step-by-step task instructions for this repo |
| `AGENTS/04_pipeline_stages.md` | Per-notebook stage docs — role, business purpose, inputs, outputs, helper deps, failure sig *(if notebooks present)* |
| `AGENTS/05_pipeline_dag.md` | Cross-notebook data flow, critical path, Workflows execution order *(if notebooks present)* |

## Module index

| Module | Path | Contract |
|--------|------|---------|
| {name} | `{path}` | `→ AGENTS/contracts/{module}.md` |
| {subsystem} | `{subdir}/` | `→ {subdir}/AGENTS/contracts/{module}.md` |

## Subsystem memory

If this repo uses nested subsystem memories, start at `AGENTS/memory_index.md`
to route to the canonical local docs before reading source.

## Updating this memory

Run `codebase-documenter` to regenerate after significant code changes.
For small changes, the documenter's incremental update mode (Phase 7) will
surgically update only the affected documents.
```

**Rules for the directory README:**
- No YAML frontmatter — absence of frontmatter is what tells the navigator
  this is human reference, not agent memory.
- Module index must stay in sync with `AGENTS/00_map.md`. Update both together.
- Keep it short — this is an orientation document, not a summary of the memory.

---

## structural-map

**File**: `AGENTS/00_map.md`

```markdown
# Codebase Map
_Last updated: {DATE}_

## Quick Reference
- **Stack**: {language}, {framework}, {DB}
- **Entry points**: {list}
- **Test command**: `{command}`
- **Deploy command**: `{command}`

## Environment Configuration
_(include this section for Databricks repos; omit for standard web service repos)_
- **Dev catalog**: `{dev_catalog}` | **Prod catalog**: `{prod_catalog}`
- **Cluster policy**: `{policy_name}` (DBR `{version}`, `{node_type}`)
- **Key Vault scope**: `{scope_name}` — secrets: `{secret_name}`, `{secret_name}`
- **Widget defaults that differ by env**: `{widget_name}` — dev: `{value}`, prod: `{value}`
- **Known version sensitivity**: `{library}=={version}` required — {what breaks otherwise}

## Module Index

| Module | Path | Purpose | Key contracts |
|--------|------|---------|---------------|
| auth | `src/auth/` | JWT + session handling | `→ contracts/auth.md` |
| users | `src/users/` | User CRUD + roles | `→ contracts/users.md` |
| pricing | `libs/pricing/` | Discount and tier calculations | `→ ../libs/pricing/AGENTS/contracts/discounts.md` |
| ...   | ...          | ...                 | ...                    |

**Key contracts column**: link only to files actually created in Phase 4.
For subsystem-local canonical contracts, link to the subdirectory contract path
registered in `memory_index.md`. Write `—` for modules with no contract file.
Do not speculate.

## Subsystem Memory Index
_(include for deep repos with nested `AGENTS/`; omit for flat or shallow repos)_

| Subsystem | Path | Owns | Canonical memory |
|-----------|------|------|------------------|
| Pricing | `libs/pricing/` | Discounting, tiering, invoice preview math | `→ memory_index.md#pricing` |

## Data Model Summary
Short prose (3–5 sentences) describing core entities and their relationships.
Include the primary keys and the most important foreign key constraint.

→ For full contracts: `contracts/`
→ For subsystem-local memory: `memory_index.md`
→ For hazards: `01_hazards.md`
→ For task playbooks: `playbooks/`
→ For pipeline stages: `04_pipeline_stages.md`   (if present)
→ For pipeline DAG: `05_pipeline_dag.md`           (if present)
```

---

## contract-note

**File**: `AGENTS/contracts/{module_name}.md`

```markdown
# Contracts: {Module Name}
_Last updated: {DATE}_
_Covers: {list of files}_

## {FunctionName / ClassName / EndpointPath}

**Summary**: One line — what this does.
**File**: `src/{path}.py:{line_number}`

**Non-obvious inputs**:
- `{param}`: {what's non-obvious about it}

**Returns**: {what it actually returns, especially edge cases}
  - Returns `None` if {condition} (does NOT raise)
  - Returns `[]` never raises `KeyError`

**Produces**: {named business output(s) this function is the origin of}
  - `CustomerROI` dataclass — consumed by executive dashboard and `/api/v2/roi`
  - `invoice_pdf` file — written to `storage/invoices/`, picked up by email worker

**Consumed by**: {what calls this, if it's a key upstream function}
  - `ReportService.build_executive_summary()` — calls this to get ROI per customer
  - `api/v2/roi` route handler — calls this directly on GET requests

**Side effects**:
- Writes to `{table}` on every call
- Emits `{event_name}` event to message bus

**Invariants** (must be true before calling):
- `{entity}_id` must exist in DB
- `request.user` must be set (middleware handles this for route handlers)

**Env dependencies**:
- Reads `{ENV_VAR}` at import time

**Idempotency**: [SAFE / NOT SAFE — do not retry]

**Failure modes**:
- `{ErrorType}` raised when {condition} — means {what it indicates at the system level}
- Silent failure: returns {value} instead of raising when {condition}
- Common production symptom: "{log message or error}" → caused by {root cause}

**Read source when**:
- Changing `{business_rule}` — read `src/{path}.py:{line}` through `src/{path}.py:{line}`
- Debugging `{boundary_case}` — read `src/{path}.py:{line}` and the caller
  listed under `Consumed by`
- Verifying exact branching behavior — docs summarize the contract; source is
  authoritative for implementation order

→ See also: `playbooks/add_endpoint.md`, `01_hazards.md#auth`
```

Only include sections that have non-trivial content.
Omit empty sections entirely.

### Traceability fields — when to use them

**`Produces`** — required on any function that is the *origin* of a named business
artifact: a report, a metric, a dataclass surfaced in a UI, a file written to storage,
an event emitted to a bus. Use the exact name that appears in the codebase (the class
name, the event name, the filename pattern). This is the field an agent searches across
all contracts to answer "what generates X?".

**`Consumed by`** — required on any function that sits at a key junction: called by
multiple callers, called by a route handler, or called by a background job. Omit for
pure leaf functions with a single obvious caller.

**`Read source when`** — required when future work may require implementation
precision beyond the contract: changing formulas, boundary conditions, retries,
authorization checks, transaction ordering, migrations, data-frame transformations,
or side-effect sequencing. Point to exact source landmarks, not broad directories.
This field is how the navigator knows when to stop relying on memory and open the
source script.

**Rule of thumb**: if someone could reasonably ask "where does this output come from?"
or "what uses this function?", fill in the field. If the answer is obvious from the
function name alone, omit it.

### Traceability query pattern (docs-only, no graph)

When an agent receives a question like *"which function generated this customer ROI
output?"*, it should:

1. Search across all `AGENTS/contracts/*.md` files for `Produces:` entries containing
   the artifact name (e.g. `CustomerROI`, `customer_roi`, `roi`)
2. If found in one contract note → that function is the origin. Report it.
3. If found in multiple → check `Consumed by` on each to determine which is upstream.
   The one with no `Consumed by` entry (or whose callers are all external/route
   handlers) is the true origin.
4. If not found → the artifact is not yet documented. Flag it and add a contract note.

This covers single-hop and two-hop chains without a graph. For chains of 3+ hops
through internal services, the docs method requires the agent to follow `Consumed by`
links manually across multiple files — functional but slower. This is the primary
motivating case for adding a graph layer later.

---

## business-logic

**File**: `AGENTS/02_business_logic.md`

Documents functions whose names or docstrings contain domain-significant terms —
calculations, formulas, scores, rates, metrics, transformations. Primarily sourced
from `utils/`, `helpers/`, `lib/`, `common/`, `core/` directories where shared
business logic lives. Indexed by **business concept**, not by module.

```markdown
# Business Logic Index
_Last updated: {DATE}_

Searchable index of functions that implement domain calculations and business rules.
To find how something is calculated: search this file by concept name.

---

## {ConceptName} (e.g. Customer ROI)

**Function**: `calculate_customer_roi(customer_id, period)`
**File**: `utils/metrics.py:142`
**Summary**: Computes ROI for a single customer over a given period.

**Formula / logic**:
Revenue from customer in period minus cost to serve, divided by cost to serve.
Cost to serve includes {list of cost components included}.
Revenue uses {which revenue table/field}.

**ML / feature semantics**: _(include for ML pipelines; omit otherwise)_
- Feature grain: {entity and time grain, e.g. customer_id x week}
- Label window: {lookahead / observation window, if this is label logic}
- Training vs scoring behavior: {differences in filters, joins, defaults}
- Leakage guard: {what prevents future data from entering training features}

**Non-obvious behavior**:
- Returns `0.0` (not None, not raises) when no transactions exist in period
- `period` must be a closed range — open-ended periods raise `ValueError`
- Negative ROI is valid and expected for churned customers in their final period

**Produces**: `CustomerROI` dataclass — consumed by executive dashboard and `/api/v2/roi`

**Expected range**: {normal output range and what deviations signal}
_(e.g. "AUC 0.78–0.87 in prod; below 0.75 indicates upstream data issue not model issue.
F1 typically 0.45–0.60 due to 8% class imbalance.")_

**Failure modes**:
- `MissingCostDataError` when cost table has no entry for customer — means
  customer was onboarded before cost tracking was introduced (pre-2023 cohort)

→ See also: `contracts/reporting.md`, `01_hazards.md#metrics`

---

## {ConceptName} (e.g. Churn Score)
...
```

**What to index here:**
- Any function containing: `calculate_`, `compute_`, `derive_`, `score_`, `rate_`, `formula`
- Any function whose docstring contains business domain terms your team uses
- For Databricks ML pipelines: feature builders, label builders, training target
  definitions, scoring functions, post-processing thresholds, evaluation metrics,
  calibration/drift checks, baseline logic, and model selection rules
- Any constant or lookup table that encodes business rules (tax rates, tier thresholds, etc.)
- Data transformations that produce named business artifacts

**`Expected range` field guidance:** Populate this for any metric or score that a human
might describe as "seeming wrong." Include the healthy range, what values outside it
indicate, and whether the range is expected to shift seasonally or with model retraining.
This is the difference between an agent saying "here is the formula" and "here is the
formula and your current value of X is outside the normal range, which suggests Y."

**What NOT to index here:**
- Pure infrastructure utilities (string formatting, date parsing, HTTP helpers)
- Functions already fully covered by a module contract note

**Deduplication pass (run after Phase 4):** Once contract files exist, review this
file and remove or condense any entry that is fully captured by a contract note.
The contract note is the single source of truth for call-site behavior; this file
is the single source of truth for the business concept and formula. If an entry
here and a contract note overlap, keep the formula/concept here and the
invariants/side-effects/failure modes in the contract. Remove outright duplication.

---

## memory-index

**File**: `AGENTS/memory_index.md`

Used in deep repos where lower-level `AGENTS/` directories remain canonical.
This file is the root router: it tells the navigator which subsystem memory to
read before loading source.

```markdown
# Memory Index
_Last updated: {DATE}_

Root routing index for subsystem-local agent memories. Use this before reading
source when the task mentions a subsystem, business concept, produced artifact,
or path listed below.

| Subsystem | Path | Owns | Canonical docs | Source entry points | Produces | Route here when |
|-----------|------|------|----------------|---------------------|----------|-----------------|
| Pricing | `libs/pricing/` | Discounting, tiering, invoice preview math | `../libs/pricing/AGENTS/manifest.md`, `../libs/pricing/AGENTS/02_business_logic.md`, `../libs/pricing/AGENTS/contracts/discounts.md` | `libs/pricing/discounts.py`, `libs/pricing/tiering.py` | `DiscountQuote`, `CustomerTier` | Task mentions discounts, customer tier, invoice preview, or paths under `libs/pricing/` |

## Cross-Subsystem Flows

| Flow | Starts in | Then reads | Output |
|------|-----------|------------|--------|
| Invoice preview | `../services/billing/AGENTS/manifest.md` | `../libs/pricing/AGENTS/contracts/discounts.md`, `../libs/tax/AGENTS/contracts/tax_rules.md` | `InvoicePreview` |

## Source-read policy

- Read subsystem memory first for orientation, contracts, formulas, and hazards.
- Read source only when a contract's `Read source when` field matches the task,
  when changing code, or when the docs say an artifact is undocumented.
- Prefer exact source landmarks from contracts or manifests over scanning entire
  directories.

→ See also: `00_map.md`, `contracts/`, subsystem `AGENTS/manifest.md` files
```

**Rules:**
- Keep one row per subsystem. Do not add function-level detail here.
- `Canonical docs` must point to existing files.
- `Source entry points` should list the few scripts that define the subsystem's
  public behavior, not every file.
- `Route here when` should use terms a user or agent is likely to ask about.
- If a lower-level subsystem depends on another lower-level subsystem, record the
  relationship in Cross-Subsystem Flows rather than duplicating either contract.

---

## subsystem-manifest

**File**: `{subdir}/AGENTS/manifest.md`

Local machine-readable routing document for a subsystem. Root `memory_index.md`
links here; the navigator reads it before opening local contracts or source.

```markdown
# Subsystem Manifest: {Subsystem Name}
_Last updated: {DATE}_

## Scope
- **Path**: `{subdir}/`
- **Owns**: {business capabilities or technical boundary}
- **Does not own**: {important adjacent responsibilities}

## Canonical docs
| Question | Read |
|----------|------|
| What does this subsystem do? | `03_narratives.md#{section}` |
| How is {concept} calculated? | `02_business_logic.md#{concept}` |
| How do I call {module}? | `contracts/{module}.md` |
| What should I avoid? | `01_hazards.md#{section}` |

## Source entry points
| Source | Read when |
|--------|-----------|
| `{subdir}/{file}.py` | Changing {specific behavior}; exact landmarks are in `contracts/{module}.md` |

## Produced artifacts
| Artifact | Produced by | Contract |
|----------|-------------|----------|
| `{ArtifactName}` | `{function_or_stage}` | `contracts/{module}.md#{anchor}` |

## Dependencies
| Direction | Subsystem | Why |
|-----------|-----------|-----|
| Upstream | `{other_subsystem}` | {what this subsystem consumes} |
| Downstream | `{other_subsystem}` | {what consumes this subsystem's output} |

## Local validation
```bash
{subsystem-specific test command}
```

→ See also: `{relative_path_to_root}/AGENTS/memory_index.md`
```

**Rules:**
- Keep it under 80 lines.
- Use this for routing and ownership, not implementation detail.
- Every source entry point should have a specific "read when" condition.
- Every produced artifact should also appear in a contract `Produces:` field.

---

## module-narratives

**File**: `AGENTS/03_narratives.md`

Plain-English descriptions of what each module does, why it exists, and how it fits
into the larger system. Written for an agent that needs to **explain the codebase to
a human** — a new developer, a non-technical stakeholder, or a user trying to
understand why something behaved a certain way.

This is the only document in AGENTS/ that is intentionally human-readable first.
Every other document is optimized for agent navigation; this one is optimized for
agent explanation.

```markdown
# Module Narratives
_Last updated: {DATE}_

Plain-English explanations of each subsystem. Use this when asked to explain
what part of the codebase does, or to give a new developer orientation.

---

## {Module Name} (e.g. Billing)

**One-line purpose**: Handles all money movement — charging customers, issuing
refunds, and generating invoices.

**Why it exists**: Extracted from the user module in 2024 when Stripe integration
grew complex enough to need its own service boundary. Previously billing logic
was mixed into user management.

**What it does in plain English**:
When a subscription is created or renewed, the billing module calculates what the
customer owes based on their plan tier and usage, calls Stripe to charge the saved
payment method, records the transaction, and triggers the invoice generation worker.
If a charge fails, it retries three times over 48 hours before marking the
subscription as past-due and notifying the customer.

**Key concepts a new developer needs to know**:
- All amounts are stored in cents (integer), never decimal dollars
- "Subscription" and "Plan" are different — a Plan is a template, a Subscription
  is a customer's active instance of a Plan
- Invoices are generated asynchronously by a background worker, not inline

**What it does NOT do**:
- Does not handle refunds — those go through `app/refunds/`
- Does not handle tax calculation — delegated to TaxJar via `app/tax/`

**Failure signature** (what a billing failure looks like in logs/errors):
- `BillingError` in logs with `stripe_code` field
- Subscription status flips to `past_due` in DB
- Customer receives automated email from `notifications/billing_failure.py`

→ See also: `contracts/billing.md`, `01_hazards.md#billing`, `playbooks/add_migration.md`

---

## {Module Name}
...
```

**What makes a good narrative:**
- Explains the *business reason* the module exists, not just what it does technically
- Uses the language a non-developer would use to describe the feature
- Explicitly states what the module does NOT handle (boundaries matter for debugging)
- Includes the failure signature — what a failure in this module looks like from the
  outside, so a debugger knows they're in the right place

---

## hazard-map

**File**: `AGENTS/01_hazards.md`

```markdown
# Hazard Map
_Last updated: {DATE}_

## 🔴 NEVER

### Never call X directly
**What**: `User.objects.delete()` bypasses soft-delete logic.
**Instead**: `UserService.soft_delete(user_id)` — preserves audit trail.
**Why it matters**: Regulatory compliance requires audit trail completeness.

### Never write to legacy_db
**What**: `legacy_db` is a read-only mirror synced every 15 min.
**Instead**: All writes go to `primary_db`.
**Why it matters**: Writes to the mirror are silently dropped.

---

## 🟡 CAUTION

### updated_at does not auto-update
**Where**: All models inheriting `TimestampedModel`
**Detail**: Must set `obj.updated_at = datetime.utcnow()` explicitly before save.
**Exception**: The `UserProfile` model overrides this — it DOES auto-update.

### External service tests require VPN
**Where**: `tests/integration/test_payment_gateway.py`
**Detail**: Mark with `@pytest.mark.skipif(not VPN_AVAILABLE, ...)`

---

## ⚪ CONVENTION

### New endpoints go in api/v2/
**Detail**: `api/` (v1) is deprecated. No new routes there.

### Error response keys use snake_case
**Detail**: `{"error_code": ..., "error_message": ...}` — never camelCase.
**Why**: Mobile clients parse snake_case; mixing breaks them.
```

---

## playbook

**File**: `AGENTS/playbooks/{task_name}.md`

```markdown
# Playbook: {Task Name}
_Last updated: {DATE}_

## Steps

1. **{Step title}**
   ```bash
   {exact command}
   ```
   Or: Edit `{file_path}` and add:
   ```python
   {exact code to add}
   ```

2. **{Next step}**
   ...

## Validation
Run `{test command}` — expect `{what success looks like}`.

## Common failures

**{Symptom}**: {Fix}
**{Symptom}**: {Fix}

→ See also: `contracts/{relevant_module}.md`, `01_hazards.md#{relevant_section}`
```

---

## agent-instructions

**File**: `AGENTS/00_agent_instructions.md`

This file follows the **open Agent Skills standard** — YAML frontmatter declares
the skill's name and description trigger, and the body is plain markdown instructions.
Any agent runner that implements the standard (Claude, Genie Code, Codex, and others)
will auto-load this skill when a task is relevant to the codebase.

```markdown
---
name: codebase-memory
description: >
  Structured memory for this codebase. Load whenever working on any code task —
  writing features, debugging failures, explaining code to developers or users,
  tracing where an output comes from, adding endpoints, running migrations, or
  understanding how any part of the system works.
---

# Agent instructions
_Last updated: {DATE}_

This codebase has structured memory in the `AGENTS/` directory. Before starting
any task, understand what each document type is for and reason from there — do
not re-read the entire codebase from scratch.

## The memory structure

Core document types. Each answers a different category of question.

**`AGENTS/00_map.md`** — answers *where things are*
The structural index: modules, file paths, entry points, data model summary, test
and deploy commands. Start here on any new task to orient yourself.

**`AGENTS/01_hazards.md`** — answers *what not to do*
NEVERs, CAUTIONs, and CONVENTIONs. Read this before any write operation, before
calling any external service, and before touching any shared infrastructure.

**`AGENTS/contracts/`** — answers *how to call something correctly*
One file per module. Each function/endpoint with non-obvious behavior has a contract
note: required invariants, side effects, return edge cases, traceability fields
(`Produces:`, `Consumed by:`), and failure modes. Use this when you need to
call something or understand its behavior precisely.

**`AGENTS/02_business_logic.md`** — answers *what something means or calculates*
Indexed by business concept. Functions in `utils/`, `helpers/`, `lib/`, `common/`
that implement domain calculations — formulas, scores, rates, metrics. Use this
when asked about what a business concept is, how it's calculated, or where the
formula lives.

**`AGENTS/03_narratives.md`** — answers *what something does and why it exists*
Plain-English module descriptions written for explanation. Use this when asked to
explain the codebase to a developer or user, give orientation to a new team member,
or explain why something works the way it does.

**`AGENTS/memory_index.md`** — answers *which subsystem memory should I load*
Root routing index for deep repos. Read it when the task mentions a subsystem,
path, artifact, or business concept that may live below the repo root. It points
to canonical lower-level `AGENTS/` docs and source entry points.

**`AGENTS/playbooks/`** — answers *how to do a specific task in this repo*
Step-by-step instructions for common tasks, specific to this codebase's conventions.
Always check here before improvising an approach.

**`AGENTS/04_pipeline_stages.md`** — answers *what a pipeline stage does and produces* *(present if repo uses Databricks notebooks)*
Per-stage: business role, inputs consumed, outputs written, helper dependencies, key
operations summary, and failure signature. Use this to understand any individual
stage without reading the raw notebook.

**`AGENTS/05_pipeline_dag.md`** — answers *how data flows between stages and what the critical path is* *(present if repo uses Databricks notebooks)*
Cross-stage data flow (which tables flow where), Workflows execution graph,
helper dependency map, ML artifacts/models, task values, and which stages are
critical path. Use this to reason about
pipeline-level questions and impact of stage failures.

## How to reason

Map any question to the document type that answers it, read that document, then
act. You do not need explicit instructions for every possible question — the
document types cover the full question space:

- *Where is X / what module handles X?* → `00_map.md`
- *Is it safe to do X / what should I never do?* → `01_hazards.md`
- *How do I call X correctly / what does X return?* → `contracts/`
- *What is the formula for X / how is X calculated?* → `02_business_logic.md`
- *What does module X do / why does it exist?* → `03_narratives.md`
- *Which lower-level docs should I read for X?* → `memory_index.md`, then the
  subsystem `AGENTS/manifest.md`
- *How do I add X / deploy / run tests in this repo?* → `playbooks/`
- *What produced output X / what calls function X?* → `contracts/` (search `Produces:` / `Consumed by:`)
- *Why did X fail / what does this error mean?* → `contracts/` (search `Failure modes:`), then `01_hazards.md`, then `03_narratives.md` (failure signature)
- *What does pipeline stage X do / what is its business purpose?* → `04_pipeline_stages.md`
- *What tables does stage X read or write?* → `04_pipeline_stages.md`
- *Which stage produces table X / what stage consumes table X?* → `05_pipeline_dag.md`
- *What is the pipeline execution order / what depends on what?* → `05_pipeline_dag.md`
- *What breaks if stage X fails / what is the critical path?* → `05_pipeline_dag.md`
- *What does the transformation logic in stage X actually do at the cell level?* → read the specific notebook file directly (filename listed in `04_pipeline_stages.md`)

For any question not listed above: identify which category it belongs to and go
to the corresponding document. If the repo has `memory_index.md`, check it before
reading source. If genuinely ambiguous, read `00_map.md` first — it always points
to the right place.

## When to read source

Use memory first, then read source only when one of these is true:
- You are editing code
- The relevant contract has a `Read source when` item matching the task
- The user asks for exact implementation details, not contract behavior
- The docs say an artifact, failure, or formula is undocumented
- The task crosses multiple subsystems and `memory_index.md` points to a source
  entry point or manifest that needs confirmation

When reading source, use the exact files, line ranges, section headers, or source
entry points from `contracts/`, `04_pipeline_stages.md`, `memory_index.md`, or a
subsystem `AGENTS/manifest.md`. Avoid broad directory scans unless the memory is
missing or stale.

## After completing a task

Update the memory if your task introduced any of the following:
- A new function with a named business output → add `Produces:` contract note
- A new calculation or formula → add entry to `02_business_logic.md`
- A new module or significant refactor → add/update entry in `03_narratives.md`
- A new subsystem or local AGENTS/ memory → add/update `memory_index.md` and
  that subsystem's `AGENTS/manifest.md`
- A new `# HACK:`, `# WARNING:`, or `# NOTE:` comment → add to `01_hazards.md`
- A new CLI command or Makefile target → update the relevant playbook
- A renamed or moved module → update `00_map.md`
- A new or modified notebook pipeline stage → update `04_pipeline_stages.md` and `05_pipeline_dag.md`
- A Delta table renamed → update the table name everywhere in `04_pipeline_stages.md` and `05_pipeline_dag.md`
```

### The description field is the trigger

The YAML `description` is what the agent runner reads to decide whether to load
this skill. Write it to match how a developer would describe a task in this repo.
Be specific and action-oriented. For domain-specific codebases, customize it to
include relevant terminology — "Lakeflow pipelines", "Nielsen RMS data",
"CPG analytics" — so it loads in the right context for your org.

---

## incremental-update-log

**File**: `AGENTS/CHANGELOG.md` (optional, for long-lived codebases)

```markdown
# AGENTS/ Changelog

## {DATE}
- Updated `contracts/auth.md`: added `TokenRefreshService.rotate()` contract
- Added hazard: `never use UserSerializer directly in admin views`
- Added playbook: `run_migrations.md`

## {DATE}
- Structural map updated: new `notifications/` module added
```

---

## pipeline-stage

**File**: `AGENTS/04_pipeline_stages.md`

One file, one section per notebook. Stages are listed in pipeline execution order.
Produced by scanning landmark lines — not by loading full notebook content.

```markdown
# Pipeline Stages
_Last updated: {DATE}_

Linear pipeline execution order — stages run top to bottom.
For data flow between stages see `05_pipeline_dag.md`.
For helper module behavior see `contracts/` or subsystem-local contracts linked
from `AGENTS/memory_index.md`.

---

## {StageName} (`{notebook_filename}.py`)

**Role**: One sentence — what this stage does in pipeline terms.

**Runs when**: _(omit if this stage always runs unconditionally)_
`{upstream_stage}` succeeds AND Workflows condition `{condition_expression}` passes.

**Inputs**:
- Delta table: `{catalog}.{schema}.{table}`
  - Columns read: `{col1}`, `{col2}`, `{col3}` _(list only columns this stage actually uses)_
  - Filter applied: `{filter_condition}` _(e.g. WHERE is_test=true — omit if full table)_
- Widget param: `{param_name}` — {what it controls, default if any}

**Outputs**:
- Delta table: `{catalog}.{schema}.{table}`
  - Grain: `{entity}` — {one row per what, e.g. "one row per customer_id"}
  - Key columns: `{col1}` ({type}, {meaning}), `{col2}` ({type}, {meaning})
  - Partition: `{partition_column}` _(write `none — full overwrite` if applicable)_
  - Sanity check: `{what normal looks like}` _(e.g. "~2M rows; if < 500k, upstream filter failed")_
- MLflow: model `{model_name}` in experiment `{experiment_path}` — {version strategy or artifact description}

**Helper dependencies**:
- `lib.{module}.{function}` — {why this stage uses it} → `contracts/{module}.md`
- `libs.{subsystem}.{module}.{function}` — {why this stage uses it}
  → `../libs/{subsystem}/AGENTS/contracts/{module}.md`
- `%run {notebook_path}` — notebook dependency; read `{notebook_path}` only
  if changing the shared setup or debugging import-time state

**MLflow / model registry**:
- Run: experiment `{experiment_path}` — logs metrics `{metric}`, `{metric}`
- Model artifact: `{artifact_path}` — registered as `{registry_model_name}`
- Registry transition: `{model_name}` version strategy `{version_strategy}` to
  stage/alias `{stage_or_alias}` _(omit if none)_

**Task values**:
- Sets `{key}` = {meaning}; consumed by `{downstream_stage}`
- Reads `{upstream_stage}.{key}` = {meaning}

**Key operations** (significant cells only — omit boilerplate):
1. [§ {Markdown section header}] {What this cell block does} — {inline or via `lib.module.function()`}
2. [§ {Markdown section header}] {Next significant operation}

**Failure signature**:
- `{ErrorType}` / `{log pattern}` — means {root cause at pipeline level}
- Downstream symptom: {what a reader of a downstream stage would observe}
- **Log location**: cluster driver logs (Workflows UI → run → driver logs tab);
  MLflow run at `{experiment_path}`; _(add custom sink path if applicable)_

→ See also: `05_pipeline_dag.md`, `contracts/{relevant_module}.md`

---

## {NextStage} (`{notebook_filename}.py`)
...
```

**Rules:**
- Omit any field that has nothing to say (e.g. no widget params → omit that line; no conditions → omit `Runs when`).
- `Inputs` columns read: list only the columns this stage actually uses — not the full table schema. An agent needs to know "does this stage depend on column X?" without reading code.
- `Outputs` key columns: list the columns that matter for downstream stages or debugging — not every column. Grain and Sanity check are required.
- "Helper dependencies" is mandatory for notebooks that call reusable code. Link
  each called helper function to its canonical contract. If the contract is
  subsystem-local, link to the subsystem `AGENTS/contracts/` path. If no contract
  exists, write `MISSING CONTRACT`, then add or route the missing contract before
  finalizing the documentation.
- "Key operations" is a summary of intent, not a cell-by-cell transcript. Target 3–7 items. Each item must: reference its `[§ Markdown section header]`, state whether logic is inline or in a named helper call such as `lib.module.function()`. This lets an agent jump to the right section of the notebook if code-read is needed — rather than scanning the whole file.
- Notebooks are orchestration documents. Keep formulas, model feature logic,
  training/scoring internals, and reusable transformations in helper contracts
  and `02_business_logic.md`; the stage entry should point to them. If important
  logic is inline in the notebook, add it to `02_business_logic.md` with the
  exact notebook section and mark the Key operation as `inline logic`.
- Table names must use full Unity Catalog 3-part names (`catalog.schema.table`) — they are the join key with `05_pipeline_dag.md`.
- Every helper import used by the stage must link to its contract file. If no
  contract exists, flag it.
- Do not document upstream execution dependencies here — execution order and task dependencies live in `05_pipeline_dag.md` and are defined by the Databricks Workflows job, not the notebook code.

---

## pipeline-dag

**File**: `AGENTS/05_pipeline_dag.md`

Cross-notebook execution graph. Answers: what order do stages run, what data flows
where, which helper modules are shared, which ML artifacts/models move between
stages, and what breaks downstream if it fails.

```markdown
# Pipeline DAG
_Last updated: {DATE}_

Execution follows the Databricks Workflows task dependency graph. It may be
linear, branched, or fan-in/fan-out. A **critical** stage produces tables,
models, task values, or artifacts consumed by downstream stages; its failure
blocks those consumers.

## Stage Sequence

Stage order reflects the Databricks Workflows task dependency graph — verify against
the job JSON/YAML definition, not notebook filenames or alphabetical order.

| Task | Depends on | Notebook | Role | Run condition |
|------|------------|----------|------|---------------|
| {TaskName} | — | `{notebook}.py` | {one-line role} | always |
| {TaskName} | `{upstream_task}` | `{notebook}.py` | {one-line role} | all_success |
| {TaskName} | `{upstream_task}` | `{notebook}.py` | {one-line role} | if `{condition}` |
| ... | | | | |

## Data Flow

| Table | Produced by | Consumed by |
|-------|-------------|-------------|
| `{catalog}.{schema}.{table}` | `{stage}` | `{stage}`, `{stage}` |
| `{catalog}.{schema}.{table}` | `{stage}` | `{stage}` |

## ML Artifact / Model Flow

| Artifact or model | Produced by | Consumed by | Registry / experiment |
|-------------------|-------------|-------------|-----------------------|
| `{model_name}` | `{training_stage}` | `{scoring_stage}` | `{experiment_or_registry_path}` |

## Task Value Flow

| Task value | Set by | Read by | Meaning |
|------------|--------|---------|---------|
| `{key}` | `{stage}` | `{stage}` | {meaning} |

## Helper Dependency Map

| Helper module/function | Used by stages | Contract |
|------------------------|----------------|----------|
| `lib.{module}.{function}` | `{stage}`, `{stage}` | `contracts/{module}.md` |
| `libs.{subsystem}.{module}.{function}` | `{stage}` | `../libs/{subsystem}/AGENTS/contracts/{module}.md` |

## Critical Path

Stages whose failure blocks downstream work:

- **{StageName}** — produces `{table_or_model_or_task_value}`, consumed by {N} downstream stages:
  `{stage}`, `{stage}`. All halt if this stage fails.

→ See also: `04_pipeline_stages.md`, `contracts/`
```

**Rules:**
- Table names in the Data Flow table must exactly match those in `04_pipeline_stages.md`.
- Do not call the pipeline linear unless the Workflows dependency graph is actually linear.
- If no Workflows definition exists, mark Stage Sequence as inferred and include
  `MANUAL VERIFICATION REQUIRED`.
- A stage can be critical with one consumer if that consumer is a terminal model
  scoring, publishing, or serving stage. Do not require 2+ consumers for criticality.
- If a stage has no helper dependencies, omit it from the Helper Dependency Map.

---

## subdirectory-redirect

**File**: `{subdir}/AGENTS.md` (e.g., `libs/AGENTS.md`)

Written during Phase 4.2 when a subdirectory's prior AGENTS/ documentation is
consolidated into the root AGENTS/. Tells both agents and humans that consolidation
occurred, where the docs now live, and confirms no detail was dropped.

```markdown
# Agent Memory — Consolidated

The agent memory that was in `{subdir}/AGENTS/` has been consolidated into the
root `AGENTS/` directory as part of full-repo documentation.

_Consolidated: {DATE}_
_Previously documented: {list of contract files that were here}_

## Where the docs are now

All contracts, narratives, and hazard notes from this directory are preserved
in the root `AGENTS/` with updated file paths.

| Was here | Now at |
|----------|--------|
| `AGENTS/contracts/{module}.md` | `{relative_path_to_root}/AGENTS/contracts/{module}.md` |

## What changed during consolidation

- File paths updated from subdirectory-relative to repo-root-relative
- Any gaps between this module's prior docs and Phase 4 findings were merged
- No documented behavior, failure modes, or invariants were removed

→ For all contracts covering `{subdir}/` modules: `{relative_path_to_root}/AGENTS/contracts/`
→ For the full memory index: `{relative_path_to_root}/AGENTS/00_agent_instructions.md`
```

**Rules:**
- `{relative_path_to_root}` must be a relative path that works from the subdirectory
  (e.g., for `libs/AGENTS.md` pointing to root, use `../AGENTS/contracts/feature_utils.md`)
- List every contract file that was previously in `{subdir}/AGENTS/contracts/` so an agent
  or human can verify nothing was silently dropped
- Keep it short — this is a redirect, not a summary of the memory

---

## subdirectory-local-contract

**File**: `{subdir}/AGENTS/contracts/{module}.md` (e.g., `libs/AGENTS/contracts/feature_utils.md`)

Written during Phase 4.2 step 4. This is a condensed local-context reference that
sits alongside the full root contract — not a duplicate. It gives agents and humans
working inside the subdirectory locally-relevant shortcuts without needing to navigate
to the root. The root contract remains authoritative.

```markdown
# Local Reference: {Module Name}

→ **Full contract**: `../../AGENTS/contracts/{module}.md`
_This file is a local supplement. The root contract is the single source of truth._

---

## Local file paths

| Function | File (subdir-relative) |
|----------|------------------------|
| `{function_name}` | `{module}.py:{line}` |

## Local import convention

```python
{how this module is imported within the subdirectory, e.g. `from libs import feature_utils`}
```

## Local test invocation

```bash
{command to test just this module in isolation, if different from repo-level test command}
```

## Subdir-specific notes

{Any edge cases, conventions, or invariants that only matter when working in this
subdirectory context — e.g. mock dependencies used in isolation testing, local
config file paths, module-level singletons initialized differently when not in
full pipeline context.}
```

**Rules:**
- Only include sections that have genuinely local-specific content. If the local import
  convention matches the root convention, omit that section.
- Never duplicate function-level behavior, failure modes, or formulas from the root contract.
  Those belong only in the root. The value here is navigation shortcuts and isolation context.
- Keep under 40 lines — if it's getting longer, the content probably belongs in the root contract.

---

## Naming Conventions

| Thing | Convention | Example |
|---|---|---|
| Module contract file | lowercase, underscores | `contracts/user_service.md` |
| Playbook file | verb_noun, underscores | `playbooks/add_endpoint.md` |
| Hazard section anchor | lowercase, no spaces | `#auth`, `#database`, `#external-apis` |
| Cross-reference links | see path rules below | |

**Path rules for cross-references:**
- Documents inside `AGENTS/` (contracts, playbooks, 00_map.md, etc.) use paths
  **relative to `AGENTS/`** — e.g. `→ contracts/auth.md`, `→ 01_hazards.md#auth`
- `AGENTS.md` at the repo root uses paths **relative to the repo root** — e.g.
  `→ AGENTS/contracts/auth.md`, `→ AGENTS/00_map.md`

This keeps links correct regardless of where the reader opens the file.

---

## Cross-Reference Pattern

Every document should end with:
```markdown
→ See also: {comma-separated list of related AGENTS/ documents}
```

For contracts: link to the playbooks that call this function.
For playbooks: link to the contracts of every function used.
For hazards: link to the modules they apply to.

This allows the agent to navigate the memory graph, not just read flat documents.
