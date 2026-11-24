# Handoff sequence

Notion task → Cursor cloud agent → writeback. Agent identity is the **Agents page ID** (Tasks relation). This handbook repo is always attached. A **work repo** (when needed) comes from the task’s **Project**, not from the Agent profile.

```mermaid
sequenceDiagram
  participant N as Notion Tasks
  participant P as Notion Projects
  participant A as Notion Agents
  participant W as Webhook / Cursor API
  participant C as Cloud agent
  participant H as muro-studio-operations
  participant R as Work repo optional
  participant T as MCP tools
  participant L as Audit Log

  N->>W: Agent Status = Send To Agent (task page ID)
  W->>C: Start run (task ID + handbook repo)
  C->>H: Load skills / rules / agents
  C->>N: Ack In Progress (task status only)
  C->>L: Fetch Audit Log DB + default template_id
  C->>L: Create run row (template, no content) Status Running
  C->>L: Fetch run page until template body ready
  C->>N: Fetch task body + Project relation
  C->>A: Fetch related Agent profile Skills Autonomy MCP
  C->>P: Resolve work repo from Project if code work
  opt Project has a repo
    W->>C: Attach work repo or C uses Project repo field
    C->>R: Implement / PR
  end
  C->>T: Do work per profile + skills MCP
  alt Done
    C->>N: Agent Status Done + Agent Summary property
    C->>L: Summary Ended Deliverables Cursor run link optional body narrative
  else Blocked or failed
    C->>N: Agent Status Blocked or Failed (properties only)
    C->>L: Summary Ended optional Message from AI on run page body
  end
```

Webhook payload stays thin (`notion_task_id`, handbook repo). Re-fetch task, Agent, and Project from Notion. Attach a work repo only when the Project (or task override) names one.