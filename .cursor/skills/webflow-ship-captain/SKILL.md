---
name: webflow-ship-captain
description: >-
  Executes Webflow CMS and site operations for Muro with safe publish discipline.
  Uses Webflow MCP and plugin skills. Use for CMS updates, component work, audits,
  and publish tasks. Code/CMS tone rules come from the task Project’s repo when present.
---

# Webflow Ship Captain

## Session start

1. Use Webflow MCP `webflow_guide_tool` once per session when using Webflow tools
2. Confirm **site**, **collection**, or **page** from task Inputs

## Skills to combine

| Situation | Also load |
|-----------|-----------|
| Muro Projects collection copy | Project skill in work repo `webflow-projects-cms` if attached |
| Pre-launch checks | Plugin `pre-deploy-check`, `link-checker`, `accessibility-audit` |
| Publish / staging | Plugin `safe-publish` |

This handbook skill holds **process**; detailed CMS tone rules live in the Project’s work repo when one is attached.

## Autonomy

| Level | Allowed |
|-------|---------|
| research | Read site/CMS only |
| draft | Prepare copy or list changes in Notion; no publish |
| pr | N/A for pure Webflow unless DevLink/code repo |
| publish | `safe-publish` only when task + profile allow |

## Writeback

List changed items (slug/name) and environment (staging vs production) in Agent run **Summary**.

## Out of scope

- Netlify deploy (hand off or load Netlify plugin if task says so)
- Arbitrary custom code in site without a Project work repo