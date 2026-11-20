# Muro Studio Operations

**Handbook repo for Muro cloud agents.** Attach this repository to every Cursor cloud run so agents load shared skills, subagents, and handoff rules—even when the task has no product codebase.

Handoff sequence: [docs/handoff-sequence.md](docs/handoff-sequence.md).

## How runs use this repo

| Task type | Repos attached |
|-----------|----------------|
| MCP-only (Notion, Webflow, Airtable, Sanity dashboard, etc.) | `muro-studio-operations` only |
| Code / deploy / PR work | `muro-studio-operations` **+** work repo (from task **Project**) |

Notion holds **profiles**, **tasks**, and **run logs**. This repo holds **how** work is done.

## Layout

```
.cursor/
  agents/          # Subagent personas (cloud reads project agents only)
  rules/           # Always-on handoff guardrails
  skills/          # Playbooks (load by name from Notion agent profile)
docs/
  handoff-sequence.md  # Notion to cloud sequence diagram
  notion-schema.md # Suggested Notion DB properties
profiles/
  README.md        # Agent name to skills map (Notion page ID is permanent key)
```

## Agent profiles (quick reference)

See [profiles/README.md](profiles/README.md). Identity in Notion is the **Agents page ID** (via Tasks relation).

## Cloud prerequisites (setup once — not on every task)

- Cursor account MCP: Notion (required), plus connectors the agents use (Webflow, Sanity, Airtable, etc.)
- Webhook or API: passes **Notion task page ID** (or URL) and always attaches **`muro-studio-operations`**; adds a work repo only when the task says so

## Light tasks (what you actually fill in)

| On the task | Required? |
|-------------|-----------|
| **Agent** (relation to Agents DB) | Yes — one pick; skills/autonomy live on the profile |
| **Title + short description** (page body; links are fine inline) | Yes |
| **Status** → e.g. Send To Agent / Ready for agent | Yes (trigger) |
| **Project** (with repo when needed) | For git/code work; MCP-only if Project has no repo |
| Separate fields for inputs or acceptance criteria | No — optional; body holds links and done-when criteria |

Runs and agent writeback can live on the task or a related **Audit Log** so the task row stays thin.

## Authoring skills

Add or edit folders under `.cursor/skills/<name>/SKILL.md`. Follow Cursor skill frontmatter (`name`, `description`). Do not write skills only on your laptop—commit them here so cloud agents see them.