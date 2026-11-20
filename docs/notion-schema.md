# Notion schema (suggested)

Three databases power cloud handoffs. Property names can be renamed if automation code uses the same API field IDs.

## Tasks

| Property | Type | Notes |
|----------|------|--------|
| Name | title | |
| Task Status / Agent Status | status | Human workflow vs agent run; e.g. Agent Status `Send To Agent` triggers cloud |
| Agent | relation → Agents | Permanent key = Notion page ID |
| Work repo | text | Empty = MCP-only; else `owner/repo` |
| Branch | text | Optional; default `main` |
| Description / page body | text | Brief + links; keep light |
| Agent Summary | text | Optional short writeback |
| Runs / Audit Log | relation | Optional; keeps task thin |

## Agents

| Property | Type |
|----------|------|
| Name | title |
| Tier | select | Meta / Core / Specialist / Legacy |
| Mission | text |
| Skills | text | Comma-separated handbook skill `name`s |
| MCP | multi-select |
| Autonomy | select | research / draft / pr / publish |
| Default work repo | text |
| Active | checkbox |
| Tools | relation | Optional catalog (Figma, Webflow, …) |
| Prompt | text | Optional short runtime cue |

Identity for routing and logs: **page URL / ID**, not a slug.

## Audit Log (agent runs)

| Property | Type |
|----------|------|
| Name | title |
| Task | relation |
| Agent | relation |
| Status | select | Queued / Running / Done / Failed |
| Summary | text |
| Deliverables | URL |
| Cursor run link | URL |
| Started / Ended | date |

## Webhook payload (minimal)

```json
{
  "notion_task_id": "…",
  "handbook_repo": "muro-studio/muro-studio-operations",
  "work_repo": null,
  "branch": "main"
}
```

The cloud agent must re-fetch the full task and related Agent page from Notion using the task ID (and Agent relation → page ID).