---
name: notion-knowledge-base
description: >
  Use Notion as a structured knowledge repository to store, organize, and retrieve research results, summaries, and findings from Claude sessions. Trigger this skill whenever the user says things like "save this to Notion", "store this research", "add this to my knowledge base", "look this up in Notion", "check Notion for context", "save your findings", "log this", or when finishing a research task and the user might want it saved. Also trigger when the user asks Claude to reference or search past research that might have been stored in Notion. Always default to Notion for knowledge storage and retrieval — don't just acknowledge, actually do it.
---

# Notion Knowledge Base Skill

## Overview

Adithya uses Notion as his primary knowledge repository. All research results, summaries, guides, and findings from Claude sessions are stored in two databases inside the **Claude Hub** page. Always interact with these databases — never create free-floating child pages under the hub root.

## Key Notion Structure

- **Claude Hub Page**: `https://www.notion.so/3101242b425a815f800ef9f90cf7a2a6`
- **📚 Knowledge Base database**: `https://www.notion.so/248d0d26cf4b4c69ba27f252838dd7e1`
  - For all research, guides, analyses, and reference material
- **📰 Newsletters database**: `https://www.notion.so/2bf49514d2c44dc5b5033fcbf4fcf768`
  - For AI newsletters and market briefs

---

## Taxonomy

### Knowledge Base — `Category` field (pick one)
| Value | Use for |
|-------|---------|
| 💼 Work | Sales enablement, professional knowledge bases |
| 🤖 AI & Tech | Architecture, agents, RAG, cloud, infrastructure |
| 💰 Finance | Portfolio, investing thesis, personal finance |
| 🙋 Personal | Style, grooming, footwear, cooking, home theater, sleep, real estate, lifestyle |
| 💡 Ideas & Futures | Essays, frameworks, civilizational questions, speculative research |

### Knowledge Base — `Type` field (pick one)
| Value | Use for |
|-------|---------|
| Research Guide | Deep-dive on a topic |
| Buyer's Guide | Product comparison with recommendations |
| Analysis | Opinion + argument |
| Reference / How-to | Procedural, look-up material |
| Project Plan | Actionable build plan |
| Conversation Summary | Notes from a single session |

### Knowledge Base — `Status` field
| Value | Meaning |
|-------|---------|
| Active | Currently using / still evolving (default for new pages) |
| Reference | Stable, look-up material |
| Archived | Superseded or no longer relevant |

### Newsletters — required fields
- `Newsletter type` — type of newsletter
- `Issue date` — actual publication date (not today's date)
- `Edition` — edition identifier
- `Topics covered` — multi-select topics
- `Status` — default `Current`

---

## Workflows

### 1. Saving Research to the Knowledge Base

**When to use**: User says "save this", "store this to Notion", "log this research", or after completing a substantial research task.

**Steps**:

1. **Search before creating** — search the Knowledge Base by topic keywords to check for an existing page:
   ```
   notion-search: [topic keywords]
   ```
   If a relevant page exists, update it (see Workflow 3) rather than creating a duplicate.

2. **Determine the fields**:
   - `Category`: one of the five options above
   - `Type`: one of the six options above
   - `Topics`: 2–5 descriptive multi-select tags
   - `Status`: `Active` (default)
   - `Source session`: optional note about where this came from

3. **Create the page** in the 📚 Knowledge Base database with the fields above. Use this content structure:
   ```markdown
   > **Summary**: One or two sentence TL;DR.

   ## Key Findings
   [Main takeaways, recommendations, or results]

   ## Details
   [Supporting information, comparisons, code, data]

   ## Sources / Notes
   Research compiled [date]. [Source attribution if applicable.]
   ```

4. **Confirm** to the user with the Notion page URL.

---

### 2. Saving a Newsletter

**When to use**: User says "save this newsletter", "log this AI brief", or after generating/receiving a newsletter.

**Steps**:

1. **Create the page** in the 📰 Newsletters database.
2. Set: `Newsletter type`, `Issue date` (publication date — not today), `Edition`, `Topics covered`, `Status = Current`.
3. **Confirm** with the page URL.

---

### 3. Updating an Existing Page

**When to use**: User asks to "update", "add to", or "append" an existing page, or you find a relevant existing page while saving new research.

**Steps**:

1. Fetch the existing page to read its current content.
2. Make targeted updates — append new sections or update stale content. Don't replace the whole page unless asked.
3. Confirm with the page URL.

---

### 4. Retrieving / Searching for Context

**When to use**: User asks to "check Notion", "look up past research", or references something Claude may have stored before.

**Steps**:

1. **Search by Topics first** — filter `Status = Active` to scope to the working set:
   ```
   notion-search: [topic keywords]
   ```
2. **Fetch the most relevant result(s)** for full content.
3. **Summarize and present** the relevant context, citing the Notion page URL.
4. If nothing relevant is found, say so clearly — don't fabricate context.

---

### 5. Archiving a Page

**When to use**: A page is superseded, outdated, or no longer relevant.

- Set `Status = Archived` on the database row.
- Do **not** rename the title with `[ARCHIVED]` — the Status field handles that.

---

## Tips

- **Search before creating** — always check for an existing page on the topic before adding a new one.
- **Use the databases** — never create free-floating pages under the hub root; always add rows to the Knowledge Base or Newsletters database.
- **Topics are the primary retrieval key** — always set 2–5 meaningful Topics tags so future searches work.
- **Keep pages focused** — one topic per page is better than one giant page.
