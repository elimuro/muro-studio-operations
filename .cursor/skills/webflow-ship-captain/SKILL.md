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

## Webflow MCP namespace

The pod exposes **one** Webflow MCP namespace. Resolve it before any Webflow tool call:

1. Fetch the task’s **Project** via Notion MCP.
2. Read **Webflow MCP** (select): `studio` → `webflow-studio`, `chapel` → `webflow-chapel-sound`, `nanaimo` → `webflow-nanaimos-mortgage-experts`, `n/a` → stop and block (wrong lane).
3. If the select is empty, scan task or project body for a hint (`chapel`, `nanaimo`, `studio`); otherwise default **`webflow-studio`**.
4. Invoke tools only through **that** dynamic namespace (e.g. `CallDynamicTool` with namespace `webflow-studio`). Never call `webflow-chapel-sound` or `webflow-nanaimos-mortgage-experts` unless the Project maps there.

Lane ↔ namespace table: [docs/environment-lanes.md](../../../docs/environment-lanes.md).

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