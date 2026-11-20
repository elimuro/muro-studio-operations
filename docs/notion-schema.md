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
| Agent Summary | text | Optional human or one-line outcome — **not** live agent heartbeats |
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

| Property | Type |
|----------|------|
| Name | title |
| Task | relation |
| Agent | relation |
| Status | select | Queued / Running / Done / Failed |
| Summary | text | **Final** outcome when Status is Done / Failed — not live heartbeats |
| *(page body)* | content | **Run log**: agents append bullet heartbeats under `## Run log` via Notion MCP |
| Deliverables | URL |
| Cursor run link | URL |
| Started / Ended | date |

## Webhook payload (minimal)

```json
{
  "notion_task_id": "…",
  "handbook_repo": "muro-studio/muro-studio-operations"
}
```

Cloud agent re-fetches task → Agent + Project. Attach work repo only when Project (or explicit task override) provides one.
