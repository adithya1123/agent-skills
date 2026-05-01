---
name: codebase-navigator
description: >
  Use for any question about code in this repository — explaining what a module
  does, finding where a business formula lives, debugging a production failure,
  tracing where an output came from, understanding what a function returns, or
  writing new features safely. Triggers on any code question regardless of
  whether the repo has been documented yet.
---

# Codebase Navigator

Answers any code question by reading structured agent memory — either a flat
`AGENTS.md` file or a full `AGENTS/` directory, depending on how the repo was documented.

**Companion skill**: `codebase-documenter` — generates the memory this skill reads.
Install both skills together.

---

## First action — always

**Check which memory format this repo uses.**

**Option A — flat format**: look for `AGENTS.md` at the repo root.
- If it exists → read it immediately. It contains the full memory in a single
  file. Use it directly — there is no `AGENTS/` directory to navigate.

**Option B — directory format**: look for `AGENTS/00_agent_instructions.md`.
- If it exists → read it immediately before doing anything else. It contains
  the full memory index and tells you exactly which document to read for any
  question. Do not scan the codebase. Do not read random files. Start there.

**If neither exists** → the repo has not been documented yet. Tell the user:
*"This repo doesn't have agent memory yet. Run the `codebase-documenter` skill
on it first — ask me to document this codebase and I'll build the memory.
Then I can answer questions about it efficiently."*
Then offer to answer from source files directly as a fallback, with the caveat
that responses will be slower and less reliable than from documented memory.

---

## The memory structure (directory format)

Six document types. Each answers a distinct question category.
Map the question to the type. Read that document. Answer.

For the flat format (`AGENTS.md`), the same question categories apply — use the
corresponding section heading within the single file instead of a separate document.

| Document | Answers |
|---|---|
| `AGENTS/00_map.md` | Where is X? What module handles X? What are the entry points? |
| `AGENTS/01_hazards.md` | What should I never do? What are the sharp edges? What's the convention? |
| `AGENTS/02_business_logic.md` | What is the formula for X? How is X calculated? Where does this metric come from? |
| `AGENTS/03_narratives.md` | What does module X do? Why does it exist? What does it NOT handle? |
| `AGENTS/contracts/{module}.md` | How do I call X? What does it return? What produced output Y? What calls Z? |
| `AGENTS/playbooks/{task}.md` | How do I add X / run tests / deploy in this specific repo? |

---

## How to reason

Any question maps to one of these categories. Do not default to reading
source files — use the memory first.

**Explaining code:**
→ Start with `03_narratives.md` for what the module does and why
→ Use `02_business_logic.md` for what any calculation means
→ Use `contracts/` for precise behavior of specific functions

**Debugging a failure:**
→ Start with `03_narratives.md` — find the failure signature for this module
→ Check `contracts/` for `Failure modes:` on the relevant function
→ Check `01_hazards.md` for known sharp edges in this area
→ Reason backwards: symptom → failure mode → root cause

**Writing new features:**
→ Start with `00_map.md` to orient
→ Read `01_hazards.md` before writing anything
→ Read `contracts/` for the module you're working in
→ Check `playbooks/` for the task type — follow it if it exists

**Tracing an output:**
→ Search `contracts/*.md` for `Produces:` entries matching the output name
→ Single match → that is the origin function
→ Multiple matches → check `Consumed by:` to find the upstream origin
→ No match → the artifact is undocumented; note this to the user

**Finding a formula or calculation:**
→ Search `02_business_logic.md` by concept name
→ If not found → search `contracts/` for function names containing the concept
→ If still not found → the calculation is undocumented; note this to the user

**For any question not listed above:**
Identify which category it belongs to from the table above.
Read that document. If genuinely ambiguous, read `00_map.md` first —
it always points to the right place.

---

## After completing a task

If your work introduced anything new, update the memory:

| What you did | What to update |
|---|---|
| New function with a named business output | Add `Produces:` contract note |
| New domain calculation or formula | Add entry to `02_business_logic.md` |
| New module or significant refactor | Add/update `03_narratives.md` |
| New `# HACK:`, `# WARNING:`, `# NOTE:` comment | Add to `01_hazards.md` |
| New CLI command or Makefile target | Update relevant playbook |
| Renamed or moved module | Update `00_map.md` |

Keeping the memory current means every future agent — and every future
question — benefits from what you just learned.
