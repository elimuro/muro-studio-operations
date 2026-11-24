---
name: notion-handoff
description: >-
  Executes Muro Studio tasks from Notion with structured writeback. Fetches task
  and agent profile, logs runs via Audit Log, and respects autonomy. Use for every
  Notion-triggered cloud run or when the user provides a Notion task URL.
---

# Notion handoff (Muro)

Agents do **not** stream live progress into Notion during a run. **Outcomes** write back once at completion, block, or failure: task **properties**, Audit Log **Summary** + **Ended**, and an optional single narrative block on the run **page body** — never task discussion comments (see **No task comments** below).

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
- If another run is already active on this task, set coarse task status only if needed, write idempotency outcome on the active audit run (**Summary** property and/or one end-of-run body update) — do not use `notion-create-comment` on the task (idempotency)

### Run page from database template

Do **not** hand-roll run page structure (no blank page with only `## Run log` in `content`). New run rows must use the linked **Audit Log** database’s **default template**.

1. **Fetch the database** (`notion-fetch` on the task’s linked Audit Log database, or the workspace standard **Agent Audit Log** if the task has no linked DB). Read `<data-source url="collection://…">` for `data_source_id` and the `<templates>` block for template IDs.
2. **Pick `template_id`**: use the template entry marked as **default** when Notion returns one; otherwise use the only template if there is just one.
3. **Create the run page** (`notion-create-pages`):
   - `parent`: `{ "type": "data_source_id", "data_source_id": "collection://…" }` (from fetch — **do not hardcode** IDs in agent code)
   - `template_id`: chosen default template
   - Set properties: title, Status `Running`, Task + Agent relations, Started date as needed
   - **Do not** pass `content` when using `template_id`; the template supplies the body.
4. **Wait for template apply** (async): `notion-fetch` the new run page and retry until the template body is present before doing work. Do **not** append step-by-step bullets to the run page during execution.

If fetch shows no templates or no default, stop and block via task status — Eli must set a default template on the Audit Log database (see `docs/notion-schema.md`). If a run row already exists, set **Summary** with `Message from AI: …` explaining the template gap.

## 3. Load skills

From profile **Skills** (comma-separated `name` values):

1. Always include `notion-handoff` (this skill)
2. Load each handbook skill from this repo’s `.cursor/skills/`
3. Load plugin skills when profile says so (e.g. `tasks-build`, Webflow `safe-publish`)

## 4. Execute

Follow the agent profile mission. Keep progress in Cursor only — **no** live Notion run-page updates while work is in flight.

**Do not** use run **Summary**, **Progress**, or task **Agent Summary** until the run ends (final outcome only).

## No task comments

Agents must **not** post discussion comments on **Tasks** pages. Never call **`notion-create-comment`** with the task as parent.

| Need | Where to write |
|------|----------------|
| Coarse lifecycle | Task **Status** / **Agent Status** only (during run: in-progress; at end: Done / Blocked / Failed) |
| Blocked narrative for Eli | **At run end**: Audit Log run **page body** — one block with `Message from AI: …` (optional; can fold into final body narrative) |
| Final outcome (2–4 sentences) | Audit Log **Summary** property; same text on task **Agent Summary** if that property exists |
| Deliverables / links | Audit Log properties (**Deliverables**, **Cursor run link**) at end |
| Duplicate-run / idempotency stop | Audit Log **Summary** (+ optional one body block) + coarse task status — no task comment |

Optional: append structured notes to the **task page body** via `notion-update-page` when the task explicitly asks for in-page deliverables — still not comments.

If blocked on a decision only Eli can make:

- Set task **Agent Status** to Blocked (or equivalent)
- Stop without guessing on publish, credentials, or client-facing copy
- At run end: Audit Log **Summary**; optional `Message from AI: …` on the run page body; mirror **Agent Summary** on the task when present — **not** a comment

## 5. Complete

When the run finishes (success, block, or failure), write back **once**:

- Audit Log row: Status Done / Failed, **Summary** (2–4 sentences), deliverable URLs, **Ended** date
- Optional: **one** narrative update on the run **page body** (e.g. blocked context, links, or notes) — not a step-by-step log
- Task: mark **Agent Status** / status Done, Blocked, or Failed; set **Agent Summary** to match run **Summary** when the property exists; do **not** prefix `Agent run:` or post a task comment

For code work in a work repo, use plugin skill `tasks-explain-diff` when useful.

## Required inputs (refuse if missing)

| Field | When required |
|-------|----------------|
| Agent profile (relation) | Always |
| Clear ask in title/body | Always — “done when …” can be one line in the body |
| Work repo (via Project) | Only when implementing in git — from related Project |
| Input URLs | Only when needed — inline links in the body count |

If the profile defines routine **definition of done**, do not require a duplicate acceptance-criteria field on the task.