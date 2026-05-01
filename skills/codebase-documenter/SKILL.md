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
| Entire repo | Full pipeline — all phases |
| Directory / module | Phases 3, 3.5, 4, 5 — scoped to that directory |
| Single file | Phase 4 only (contract extraction) |
| Already-indexed repo, new files added | Phase 7 — incremental update only |

Ask if unclear: *"Should I document the whole repo or a specific part?"*

---

## Phase 2: Choose Output Format

**Before doing any file reading, decide whether to produce a flat `AGENTS.md` or
the full `AGENTS/` directory structure.**

Use the flat format when ALL of the following are true:
- Fewer than ~15 source files in scope
- Fewer than 3 distinct modules with their own interfaces
- No non-trivial domain calculations (no `calculate_`, `score_`, `rate_` functions)
- Single contributor or very small team — playbooks add no value
- The whole output will fit comfortably under ~150 lines

Use the full `AGENTS/` directory when ANY of the following are true:
- 15+ source files
- 3+ modules with distinct interfaces or contracts
- Domain calculations that need their own indexed section
- Multiple contributors who need playbooks
- The flat format would exceed ~150 lines

**If flat format**: produce a single `AGENTS.md` at the repo root using the
`agents-entry` schema from `references/document-schemas.md`. This file includes
YAML frontmatter so the navigator can load it as the agent memory entry point.
Skip straight to the Final Step — the flat file IS the agent instructions. Stop.

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

## Phase 3: Reconnaissance

**Goal**: Build the structural map without reading every file.

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

Output → `AGENTS/00_map.md`
Format → `references/document-schemas.md#structural-map`

---

## Phase 3.5: Business Logic & Narrative Extraction

Two outputs. Both serve goals 1 and 2 (explain + debug).

### Business Logic Index

Scan `utils/`, `helpers/`, `lib/`, `common/`, `core/` for functions whose
names or docstrings contain domain-significant terms: `calculate_`, `compute_`,
`derive_`, `score_`, `rate_`, `margin_`, `forecast_`, `roi`, `ltv`, `churn`,
`threshold`, or any term that maps to a concept in this domain.

For each: extract the formula in plain English, return edge cases, business
concept it implements, and its `Produces:` output if it generates a named artifact.

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

One contract note per function/endpoint with non-obvious behavior.

Output → `AGENTS/contracts/{module_name}.md`
Format → `references/document-schemas.md#contract-note`

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

**Goal**: Step-by-step instructions for the 10 most common agent tasks.

Always include:
1. Add a new API endpoint
2. Add a database migration
3. Add a new test
4. Run the test suite
5. Deploy / build

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
- New module added → add narrative to `03_narratives.md`, add row to `AGENTS.md` module index
- New CLI command / Makefile target → update playbook
- Renamed module → update `00_map.md` + all cross-references + `AGENTS.md` module index
- Module removed → remove from `00_map.md`, `AGENTS.md`, and its contract file

Procedure: read changed files → identify affected AGENTS/ documents →
surgical edits only → update `last_updated` timestamp.

---

## Final Step: Generate Agent Instructions

After all other documents are written, generate `AGENTS/00_agent_instructions.md`.

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
- [ ] `00_map.md` — every module listed, test/deploy commands accurate
- [ ] `01_hazards.md` — every hazard specific ("never call X" not "be careful")
- [ ] `02_business_logic.md` — every domain calculation has formula in plain English
- [ ] `03_narratives.md` — every module has a "what it does NOT do" section
- [ ] `contracts/` — every note has something non-obvious; omit if obvious from signature
- [ ] `contracts/` — key functions have `Failure modes:` populated
- [ ] `playbooks/` — every playbook mentally traced end-to-end
- [ ] `00_agent_instructions.md` — memory index matches files actually produced
- [ ] No document exceeds ~150 lines (split by module if needed)

Read `references/document-schemas.md` for all output format specifications.
Read `references/memory-taxonomy.md` for the theoretical framework.
