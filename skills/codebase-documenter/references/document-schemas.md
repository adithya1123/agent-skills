# Document Schemas

All AGENTS/ output documents follow these schemas.
Copy-paste these templates when creating new documents.

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
| `AGENTS/contracts/` | Per-module call contracts and traceability |
| `AGENTS/playbooks/` | Step-by-step task instructions for this repo |

## Module index

| Module | Path | Contract |
|--------|------|---------|
| {name} | `{path}` | `→ AGENTS/contracts/{module}.md` |

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

## Module Index

| Module | Path | Purpose | Key contracts |
|--------|------|---------|---------------|
| auth | `src/auth/` | JWT + session handling | `→ contracts/auth.md` |
| users | `src/users/` | User CRUD + roles | `→ contracts/users.md` |
| ...   | ...          | ...                 | ...                    |

**Key contracts column**: link only to files actually created in Phase 4.
Write `—` for modules with no contract file. Do not speculate.

## Data Model Summary
Short prose (3–5 sentences) describing core entities and their relationships.
Include the primary keys and the most important foreign key constraint.

→ For full contracts: `contracts/`
→ For hazards: `01_hazards.md`
→ For task playbooks: `playbooks/`
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

**Non-obvious behavior**:
- Returns `0.0` (not None, not raises) when no transactions exist in period
- `period` must be a closed range — open-ended periods raise `ValueError`
- Negative ROI is valid and expected for churned customers in their final period

**Produces**: `CustomerROI` dataclass — consumed by executive dashboard and `/api/v2/roi`

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
- Any constant or lookup table that encodes business rules (tax rates, tier thresholds, etc.)
- Data transformations that produce named business artifacts

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

Five document types. Each answers a different category of question.

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

**`AGENTS/playbooks/`** — answers *how to do a specific task in this repo*
Step-by-step instructions for common tasks, specific to this codebase's conventions.
Always check here before improvising an approach.

## How to reason

Map any question to the document type that answers it, read that document, then
act. You do not need explicit instructions for every possible question — the
document types cover the full question space:

- *Where is X / what module handles X?* → `00_map.md`
- *Is it safe to do X / what should I never do?* → `01_hazards.md`
- *How do I call X correctly / what does X return?* → `contracts/`
- *What is the formula for X / how is X calculated?* → `02_business_logic.md`
- *What does module X do / why does it exist?* → `03_narratives.md`
- *How do I add X / deploy / run tests in this repo?* → `playbooks/`
- *What produced output X / what calls function X?* → `contracts/` (search `Produces:` / `Consumed by:`)
- *Why did X fail / what does this error mean?* → `contracts/` (search `Failure modes:`), then `01_hazards.md`, then `03_narratives.md` (failure signature)

For any question not listed above: identify which category it belongs to and go
to the corresponding document. If genuinely ambiguous, read `00_map.md` first —
it always points to the right place.

## After completing a task

Update the memory if your task introduced any of the following:
- A new function with a named business output → add `Produces:` contract note
- A new calculation or formula → add entry to `02_business_logic.md`
- A new module or significant refactor → add/update entry in `03_narratives.md`
- A new `# HACK:`, `# WARNING:`, or `# NOTE:` comment → add to `01_hazards.md`
- A new CLI command or Makefile target → update the relevant playbook
- A renamed or moved module → update `00_map.md`
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
