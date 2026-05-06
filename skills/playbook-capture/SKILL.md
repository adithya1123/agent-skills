---
name: playbook-capture
description: >
  Manually capture a completed codebase exploration, debugging path, solution,
  or explanation into a reusable root AGENTS/playbooks/ playbook. Use when the
  user asks to capture this as a playbook, document this path, save this
  exploration, make a playbook from what we learned, or record the direct path
  for next time.
---

# Playbook Capture

Turns a solved exploration into a concise root playbook that future agents can
follow directly. This skill is manual-only: use it when the user explicitly asks
to capture what happened.

Use the current session context as the primary source of truth: what was asked,
which AGENTS documents and source files were read, which dead ends were avoided,
what conclusion or fix was reached, and how it was verified. Do not ask the user
to restate information already present in the session.

---

## First action

Confirm the target repo has root directory-format memory:

1. Check for `AGENTS/`.
2. Check for `AGENTS/00_agent_instructions.md`.
3. Check or create `AGENTS/playbooks/`.

If root `AGENTS/` or `AGENTS/00_agent_instructions.md` is missing, stop and tell
the user to run `codebase-documenter` first. Do not create standalone playbooks
outside a documented repo.

Always write captured playbooks to:

```text
AGENTS/playbooks/{task_name}.md
```

Do not write subsystem-local playbooks in v1.

---

## What to capture

Capture the direct reusable path, not the full exploration transcript.

Include:
- The task, symptom, workflow, artifact, or explanation this applies to
- The AGENTS documents that gave the shortest route
- Exact source landmarks that mattered
- Why this path was the right path to take
- The solution or explanation learned from the exploration
- Exact validation command or verification signal, when available
- Common failures or misleading paths discovered during exploration

Omit:
- Dead-end searches unless they belong in `Avoid`
- Full source summaries
- Raw conversation history
- Speculation that was not verified

If a matching playbook already exists, update it instead of creating a duplicate.

---

## Naming

Use a short verb-noun or symptom-based filename:

```text
AGENTS/playbooks/debug_missing_score.md
AGENTS/playbooks/explain_customer_score.md
AGENTS/playbooks/add_feature_builder.md
```

Use lowercase words separated by underscores.

---

## Date rule

Immediately before creating or updating a playbook file, run:

```bash
python3 -c "import datetime; print(datetime.date.today())"
```

Use that exact output for `_Last updated: {DATE}_`. This command is mandatory
for every write, including updates to an existing playbook. Do not use the
conversation date, system date from memory, or a guessed date.

---

## Playbook format

```markdown
# Playbook: {Task or Explanation Name}
_Last updated: {DATE}_

## Use when
{When this playbook applies.}

## Fast path
1. Read `{specific AGENTS file or section}`.
2. Open `{specific source file/function/section}`.
3. Check or edit `{specific config/module/test}`.
4. Run `{validation command}`.

## Why this path
{Explain briefly why these docs/files are the direct route and what broader
searches they replace.}

## Solution explanation
{Explain the answer, fix, or codebase behavior learned from the exploration.}

## Source landmarks
- `{file}`: `{function, section, class, config key, or test}`

## Validation
Run `{command}` — expect `{success signal}`.

## Common failures
**{symptom}**: {fix}

## Avoid
{Wrong broad searches, misleading files, or stale assumptions to avoid.}

→ See also: `{related AGENTS doc or contract}`
```

If no command-based validation exists, write the concrete check that confirmed
the answer, such as the exact document, code path, test, log pattern, or source
landmark that proves it.

---

## Quality bar

Before finishing, verify:
- The playbook lives under root `AGENTS/playbooks/`
- It has `Use when`, `Fast path`, `Why this path`, `Solution explanation`,
  `Source landmarks`, `Validation`, `Common failures`, and `Avoid`
- It reflects the current session context, not a generic template
- `_Last updated` came from the mandatory date command run immediately before
  the file write
- The fast path is shorter than the original exploration
- Links point to existing AGENTS docs or source files where possible
- It does not require reading broad directories before trying the known path
