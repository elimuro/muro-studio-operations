---
name: notion-handoff
description: >-
  Executes Muro Studio tasks from Notion with structured writeback. Fetches task
  and agent profile, updates Agent status heartbeats, logs runs, and respects
  autonomy. Use for every Notion-triggered cloud run or when the user provides a
  Notion task URL.
---

# Notion handoff (Muro)

The user may watch the Notion board during the run. **Progress and outcomes live in Notion**, not only in chat.

## 1. Load context

Use Notion MCP to:

- Open the **task** page (ID or URL from user/webhook)
- Read title, body, **Project** relation, and Agent Status
- Open the related **Agent** profile via **relation** (Notion page ID is the permanent key)
- Read profile: **Mission**, **Skills**, **Autonomy**, **MCP**, page body for in/out of scope
- If the work needs git: resolve **work repo from the task’s Project** (not from the Agent). MCP-only if Project has no repo.

Cross-check agent names ↔ skills with [profiles/README.md](../../../profiles/README.md).

## 2. Acknowledge and claim

- Set task **Status** / **Agent Status** to in-progress equivalents (e.g. `In Progress`)
- Heartbeat briefly if the board has an agent status / summary field
- Create or update **Audit Log** / run row: Status `Running`, link to task
- If another run is already active on this task, comment and exit (idempotency)

## 3. Load skills

From profile **Skills** (comma-separated `name` values):

1. Always include `notion-handoff` (this skill)
2. Load each handbook skill from this repo’s `.cursor/skills/`
3. Load plugin skills when profile says so (e.g. `tasks-build`, Webflow `safe-publish`)

## 4. Execute

Follow the agent profile mission. Heartbeat every major step (brief: emoji + few words).

If blocked on a decision only Eli can make:

- Comment: `Message from AI: …`
- Stop without guessing on publish, credentials, or client-facing copy

## 5. Complete

- Mark task / agent status Done or Blocked / Failed as appropriate
- Audit log: Summary, deliverable URLs, ended time
- Task comment: `Agent run: <2–4 sentence summary>`

For code work in a work repo, use plugin skill `tasks-explain-diff` when useful.

## Required inputs (refuse if missing)

| Field | When required |
|-------|----------------|
| Agent profile (relation) | Always |
| Clear ask in title/body | Always — “done when …” can be one line in the body |
| Work repo (via Project) | Only when implementing in git — from related Project |
| Input URLs | Only when needed — inline links in the body count |

If the profile defines routine **definition of done**, do not require a duplicate acceptance-criteria field on the task.