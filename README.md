# Muro Studio Operations

Handbook for **Muro Cursor cloud agents**: skills, handoff rules, and agent↔Notion mapping. Attach this repo on every Notion-triggered run; add a **work repo** only when the task’s Notion **Project** needs git.

**Agents do not read this file.** Runs load `.cursor/rules/muro-cloud-handoffs.mdc`, the `notion-handoff` skill, then skills named on the Notion **Agent** profile.

| For | See |
|-----|-----|
| Agent name → skills | [profiles/README.md](profiles/README.md) |
| Notion → cloud flow | [docs/handoff-sequence.md](docs/handoff-sequence.md) |
| Suggested Notion fields | [docs/notion-schema.md](docs/notion-schema.md) |

**Human setup (once):** Notion MCP on the agent account; webhook or automation sends `notion_task_id` and attaches this handbook. Work repo comes from the task **Project**, not from the agent profile.