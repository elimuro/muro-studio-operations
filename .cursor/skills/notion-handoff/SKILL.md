---
name: notion-handoff
description: >-
  Executes Muro Studio tasks from Notion with structured writeback. Fetches task
  and agent profile, appends heartbeats to the Audit Log run page, logs runs, and respects
  autonomy. Use for every Notion-triggered cloud run or when the user provides a
  Notion task URL.
---

# Notion handoff (Muro)

The user may watch the Notion board during the run. **Live progress heartbeats append to the active Audit Log run page body** (not properties). **Outcomes** write back via task **properties** and/or task **page body** updates plus the Audit Log run **Summary** property — never task discussion comments (see **No task comments** below).

## 1. Load context

Use Notion MCP to:

- Open the **task** page (ID or URL from user/webhook)
- Read title, body, **Project** relation, and Agent Status
- Open the related **Agent** profile via **relation** (Notion page ID is the permanent key)
- Read profile: **Mission**, **Skills**, **Autonomy**, **MCP**, page body for in/out of scope
- If the work needs git: resolve **work repo from the task’s Project** (not from the Agent). MCP-only if Project has no repo.

Cross-check agent names ↔ skills with [profiles/README.md](../../../profiles/README.md).

## 2. Acknowledge and claim

- Set task **Status** / **Agent Status** to in-progress equivalents (e.g. `In Progress`) — coarse lifecycle only, no step-by-step text on the task
- Create **Audit Log** run row (see **Run page from database template** below): Status `Running`, link to task and agent; note the run **page ID**
- If another run is already active on this task, append an idempotency note to the audit run page (or the existing active run’s page body), set coarse task status only if needed, and exit — do not use `notion-create-comment` on the task (idempotency)

### Run page from database template

Do **not** hand-roll run page structure (no blank page with only `## Run log` in `content`). New run rows must use the linked **Audit Log** database’s **default template**.

1. **Fetch the database** (`notion-fetch` on the task’s linked Audit Log database, or the workspace standard **Agent Audit Log** if the task has no linked DB). Read `<data-source url="collection://…">` for `data_source_id` and the `<templates>` block for template IDs.
2. **Pick `template_id`**: use the template entry marked as **default** when Notion returns one; otherwise use the only template if there is just one.
3. **Create the run page** (`notion-create-pages`):
   - `parent`: `{ "type": "data_source_id", "data_source_id": "collection://…" }` (from fetch — **do not hardcode** IDs in agent code)
   - `template_id`: chosen default template
   - Set properties: title, Status `Running`, Task + Agent relations, Started date as needed
   - **Do not** pass `content` when using `template_id`; the template supplies the body (including the run-log section).
4. **Wait for template apply** (async): `notion-fetch` the new run page and retry until the template’s run-log section exists (e.g. `## Run log` or the heading your workspace template uses). Only then call `insert_content` for the first heartbeat.

If fetch shows no templates or no default, stop and append `Message from AI: …` on the audit run page if one exists, or block via task status — Eli must set a default template on the Audit Log database (see `docs/notion-schema.md`).

## 3. Load skills

From profile **Skills** (comma-separated `name` values):

1. Always include `notion-handoff` (this skill)
2. Load each handbook skill from this repo’s `.cursor/skills/`
3. Load plugin skills when profile says so (e.g. `tasks-build`, Webflow `safe-publish`)

## 4. Execute

Follow the agent profile mission. **Heartbeat every major step** by **appending** one line to the run page body — do not overwrite properties for live progress.

**Heartbeat write (Notion MCP):**

- Target: the current run row’s **page** (`page_id` of the Audit Log entry)
- Use `notion-update-page` with `command: insert_content`, `position: { "type": "end" }`
- Each heartbeat is one bullet, brief: emoji + few words, e.g. `- Claimed run` or `- 🔍 Fetching agent profile`
- Optional: prefix with local time once per line if useful (`- 14:32 — …`); keep lines short

**Do not** use run **Summary**, **Progress**, or task **Agent Summary** for heartbeats (those fields are final outcome only). Do not heartbeat on the task page.

## No task comments

Agents must **not** post discussion comments on **Tasks** pages. Never call **`notion-create-comment`** with the task as parent.

| Need | Where to write |
|------|----------------|
| Live step / heartbeat | Audit Log run **page body** (`insert_content` at end) |
| Blocked narrative for Eli | Audit Log run **page body**: append `Message from AI: …` |
| Final outcome (2–4 sentences) | Audit Log **Summary** property; same text on task **Agent Summary** if that property exists |
| Coarse lifecycle | Task **Status** / **Agent Status** only |
| Duplicate-run / idempotency stop | Audit Log run **page body** + coarse task status if needed — no task comment |

Optional: append structured notes to the **task page body** via `notion-update-page` when the task explicitly asks for in-page deliverables — still not comments.

If blocked on a decision only Eli can make:

- Append `Message from AI: …` to the **active Audit Log run page body**
- Set task **Agent Status** to Blocked (or equivalent)
- Stop without guessing on publish, credentials, or client-facing copy
- At run end only: optional task **Agent Summary** (2–4 sentences) via property update — **not** a comment

## 5. Complete

- Mark task / agent status Done or Blocked / Failed as appropriate
- Audit log row: **Summary** property (final only, 2–4 sentences), deliverable URLs, ended time — leave the appended run log as the step history
- Task: set **Agent Summary** property to the same final summary when the property exists; do **not** prefix `Agent run:` or post a task comment

For code work in a work repo, use plugin skill `tasks-explain-diff` when useful.

## Required inputs (refuse if missing)

| Field | When required |
|-------|----------------|
| Agent profile (relation) | Always |
| Clear ask in title/body | Always — “done when …” can be one line in the body |
| Work repo (via Project) | Only when implementing in git — from related Project |
| Input URLs | Only when needed — inline links in the body count |

If the profile defines routine **definition of done**, do not require a duplicate acceptance-criteria field on the task.