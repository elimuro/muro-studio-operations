---
name: dispatcher
description: >-
  Routes a Notion task to the correct Muro agent profile and skill set. Validates
  inputs, refuses ambiguous or under-specified work, and delegates without
  executing specialist work itself. Use as the first skill for webhook runs or
  when the Agent relation is empty.
---

# Dispatcher

The Dispatcher does **not** replace specialist agents. It **chooses** one primary Agents DB profile (by page ID / Name) and loads that agent’s **Skills**.

## Workflow

```
Task progress:
- [ ] Load task + linked profile via notion-handoff steps 1–2
- [ ] If Agent relation empty, infer from tags/project; set relation or document guess in comment
- [ ] Validate required inputs for that agent (see table)
- [ ] If invalid, block with Message from AI listing missing fields
- [ ] Load specialist skills from profile; proceed as that agent
```

## Selection heuristics

| Signals on task | Primary agent (Name) |
|-----------------|----------------------|
| Webflow, CMS, publish, site | Webflow Ship Captain |
| Sanity, schema, GROQ, Studio | Content Architect |
| Figma canvas, mockup, frame, component, variant, FigJam, diagram (no repo) | Figma Design Expert |
| Figma + implement UI, design-to-code, PR, Project repo | Implementation Bridge |
| Airtable, NocoDB, chapel, ops, status | Studio Ops Coordinator |
| Unclear | Ask via blocked comment; do not pick at random |

## Validation by agent

| Agent | Minimum |
|-------|---------|
| Webflow Ship Captain | Target site or collection hint |
| Content Architect | Project/dataset or Studio URL |
| Figma Design Expert | Figma file link or permission to create file; clear frame/screen brief |
| Implementation Bridge | Figma link; Project with repo if code change |
| Studio Ops Coordinator | Which base/table/page to touch |

## Out of scope

- Performing destructive publishes under `research` or `draft` autonomy
- Running two specialist personas in one run without explicit task subtasks