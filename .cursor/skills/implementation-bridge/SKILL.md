---
name: implementation-bridge
description: >-
  Bridges Figma and implementation repos for Muro design-to-ship work. Uses Figma
  plugin skills and defers code to the task Project's repo when present. Use for UI
  implementation, spec alignment, and design QA tasks.
---

# Implementation Bridge

## Design sources

| Source | Skill |
|--------|--------|
| Figma | Plugin `figma-use`, `figma-design-to-code`, `figma-implement-motion` as needed |

Elyx is not part of this handbook yet. If a task mentions `.elyx` or Elyx, set **Agent blocked** and ask Eli to narrow the task to Figma or add Elyx skills later.

## Work repo

From the task’s **Project** (not the Agent). Required when implementing code/PRs. Optional for read-only spec review.

## Workflow

```
- [ ] Collect Figma links from Notion Inputs
- [ ] Resolve design truth via Figma MCP / plugin skills
- [ ] State acceptance criteria mapping: frame/layer → code artifact
- [ ] Implement in Project repo using that repo’s conventions
- [ ] If Project has no repo: deliver spec comments + checklist in Notion only
```

## Quality bar

- Match existing project naming and tokens before inventing new patterns
- Call out visual deltas vs design in the summary (spacing, type, motion)

## Related

- Canvas-only work (no repo): **Figma Design Expert** via Dispatcher
- Muro Webflow CMS copy from design: may chain to `webflow-ship-captain` after copy is approved