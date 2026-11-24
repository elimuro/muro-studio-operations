# Notion schema (suggested)

Three databases power cloud handoffs (plus existing **Projects**). Property names can be renamed if automation uses the same API field IDs.

## Tasks

| Property | Type | Notes |
|----------|------|--------|
| Name | title | |
| Task Status / Agent Status | status | Human vs agent; e.g. Agent Status `Send To Agent` triggers cloud |
| Agent | relation → Agents | Permanent key = Notion page ID |
| Project | relation → Projects | Source of work repo when code is needed |
| Description / page body | text | Brief + links; keep light |
| Agent Summary | text | **Final agent outcome only** (2–4 sentences when the agent completes or blocks at end): mirrors Audit Log run **Summary**; set via `notion-update-page` on properties — not live run log, not discussion comments |
| Runs / Audit Log | relation | Optional; keeps task thin |

No per-task Work repo field required if Projects carries the repo.

## Projects (existing)

| Property | Type | Notes |
|----------|------|--------|
| Project Name | title | |
| Work repo (add if missing) | text or URL | `owner/repo` or Origin URL — resolved by agents from task → Project |
| URL / File Links | already present | May hold related links until Work repo is added |

## Agents

| Property | Type |
|----------|------|
| Name | title |
| Tier | select | Meta / Core / Specialist / Legacy |
| Mission | text |
| Skills | text | Comma-separated handbook skill `name`s |
| MCP | multi-select |
| Autonomy | select | research / draft / pr / publish |
| Active | checkbox |
| Tools | relation | Optional catalog |
| Prompt | text | Optional short runtime cue |

**Do not** put default work repo on Agents — one agent serves many projects.

## Audit Log (agent runs)

Agents create new run **pages** from this database’s **default Notion template** (not a blank page or hand-written `## Run log`). The template defines page layout; agents set properties during the run and write **final notes only** to the page body at the end — not a live step-by-step log.

**Resolve IDs at runtime** — do not hardcode workspace UUIDs in agent code or this handbook:

| Resource | How agents find it |
|----------|-------------------|
| Agent Audit Log database | `notion-search` or fetch from task’s Runs / Audit Log relation |
| Data source | `notion-fetch` on the database; read `<data-source url="collection://<audit-log-data-source>">` |
| Example placeholders (docs only) | `AGENT_AUDIT_LOG_DATABASE_ID`, `collection://<audit-log-data-source>` |

**Ops (Eli):** In Notion, set **one default template** on the Agent Audit Log database so `notion-fetch` returns a default entry under `<templates>`. Agents use whatever default fetch returns; no template / no default → agent blocks via task status and records the gap in Audit Log **Summary** (no task comments).

| Property | Type |
|----------|------|
| Name | title |
| Task | relation |
| Agent | relation |
| Status | select | Queued / Running / Done / Failed |
| Summary | text | **Final** outcome when Status is Done / Failed |
| *(page body)* | content | From **DB default template**; optional **single** end-of-run narrative (blocked notes, links) — **not** live progress bullets |
| Deliverables | URL |
| Cursor run link | URL |
| Started / Ended | date |

## Webhook payload (minimal)

```json
{
  "notion_task_id": "TASK_PAGE_ID",
  "handbook_repo": "muro-studio/muro-studio-operations"
}
```

Cloud agent re-fetches task → Agent + Project. Attach work repo only when Project (or explicit task override) provides one.