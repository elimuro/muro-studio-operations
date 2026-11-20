# Agent profiles (Notion ↔ handbook)

Notion **Agents** database is canonical for mission, autonomy, and which skills to load. Permanent identity is the **Notion page ID** (use a Tasks → Agents relation). **Work repo** comes from the task’s **Project**, never from the agent.

| Display name | Skills to load |
|--------------|----------------|
| Dispatcher | `dispatcher`, `notion-handoff` |
| Studio Ops Coordinator | `studio-ops-coordinator`, `notion-handoff` |
| Webflow Ship Captain | `webflow-ship-captain`, `notion-handoff` |
| Content Architect | `content-architect`, `sanity-best-practices`, `notion-handoff` |
| Implementation Bridge | `implementation-bridge`, `notion-handoff` |

## Notion profile page template (top of page)

```yaml
mission: one sentence
in_scope: bullets
out_of_scope: bullets
autonomy: research | draft | pr | publish
default_skills: webflow-ship-captain, notion-handoff
mcp: Notion, Webflow
```

## Plugin skills (not in this repo)

These ship with Cursor plugins—reference them on the profile when needed:

- Notion: `tasks-build`, `tasks-explain-diff`
- Webflow: `safe-publish`, `pre-deploy-check`, `cms-best-practices`
- Netlify: `netlify-deploy`, `netlify-functions`
- Figma: `figma-design-to-code`, `figma-use`
