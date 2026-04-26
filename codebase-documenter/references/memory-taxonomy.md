# Memory Taxonomy Reference

## The Write–Manage–Read Framework

Agent memory operates as a continuous loop:

1. **Write** — extract from source (code, docs, observations)
2. **Manage** — organize, link, prune, update
3. **Read** — retrieve the right thing at the right time

Most codebase documentation tools focus on Write. This skill optimizes all three.

---

## Three Memory Tiers (for codebase context)

### Tier 1: Semantic Memory (Long-Term, Stable)
What the codebase IS. Changes rarely.

- Module boundaries and ownership
- Data models and their relationships
- Core abstractions and their invariants
- Technology stack and runtime constraints

This maps to: `AGENTS/00_map.md` + `AGENTS/contracts/`

### Tier 2: Episodic Memory (Medium-Term, Changes with Code)
What the codebase DOES in practice. Changes with refactors.

- How features are implemented end-to-end
- Which modules call which
- Common task workflows (the playbooks)
- Historical decisions visible in migration files

This maps to: `AGENTS/playbooks/`

### Tier 3: Procedural Hazard Memory (Durable, Team-Specific)
What the codebase FORBIDS or warns about. Changes slowly.

- Sharp edges and gotchas that bit the team
- Invariants that the code doesn't enforce but must be upheld
- Environment and deployment landmines

This maps to: `AGENTS/01_hazards.md`

---

## The Zettelkasten Principle (Applied to Code)

From A-MEM (NeurIPS 2025): memories linked to related memories retrieve better
than isolated facts. Apply this to code documentation:

**Bad**: A flat list of function signatures.

**Good**: Each contract note links to:
- The data model it operates on
- The playbook that uses it
- The hazard it's adjacent to

In practice: use `→ see also:` cross-references in every document.
The agent follows these links naturally during task execution.

---

## What the ETH Zurich Study (2026) Tells Us

Study finding: LLM-generated AGENTS.md files:
- Reduced task success rates vs. no context file at all
- Increased inference costs by ~20%
- Failed because: 95–100% included repository overviews that agents could derive

The one thing developer-written files had that LLM files lacked:
**Non-inferable, environment-specific, team-specific knowledge.**

Implication for this skill:
The value is ENTIRELY in Phase 5 (Hazard Map) and Phase 4 (Contract Extraction).
Phase 3 (Structural Map) is ONLY valuable for large, opaque monorepos where
the directory structure is misleading or deeply nested.

---

## Agent Retrieval Patterns

Understanding how the agent will USE this memory helps design it better.

### Pattern 1: Task-first retrieval
Agent receives: "Add a /v2/users/:id/deactivate endpoint"
Agent needs: playbook for adding endpoints → hazard map for auth + DB patterns → contracts for UserService

Design implication: Playbooks must explicitly reference contracts and hazard items.

### Pattern 2: Exploration retrieval
Agent receives: "Why is this test failing?"
Agent needs: structural map → locate the module → read contracts → check hazards

Design implication: `00_map.md` must surface hazards relevant to each module inline.

### Pattern 3: Constraint retrieval
Agent is mid-task and hits an ambiguity.
Agent needs: quick lookup of "what does this function actually return when X?"

Design implication: Contract notes must be scannable — one-line summary + bullets only.

---

## Size and Token Budget

Target document sizes (to fit in agent context windows efficiently):

| Document | Target Size | Hard Limit |
|---|---|---|
| `00_map.md` | 60–100 lines | 150 lines |
| `01_hazards.md` | 40–80 lines | 120 lines |
| Per-module contract | 30–60 lines | 100 lines |
| Per-task playbook | 20–40 lines | 60 lines |

If a module contract exceeds the limit, split by sub-module.
If the hazard map exceeds the limit, split by category (auth, DB, external APIs).

**Why this matters**: Agents with focused, short documents outperform agents with
comprehensive, long documents. From the survey literature: "Long context is not
memory." Selective retrieval of short documents beats passive injection of long ones.
