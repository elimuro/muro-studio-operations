# Environment lanes

Cursor **cloud environments** split MCP connectors and default agent tooling so each run loads a smaller cached prompt prefix. Notion stays on **one webhook pattern**; lane choice is which Cursor environment (or automation URL) receives the run.

Handoff flow: [handoff-sequence.md](handoff-sequence.md). Notion fields: [notion-schema.md](notion-schema.md).

## Goals

| Goal | Why |
|------|-----|
| **Smaller cached prefix** | Each environment enables only the MCP namespaces that lane needs — not every Webflow workspace, Figma, Sanity, and Airtable on every pod. |
| **One Notion trigger** | Tasks **Agent Status** → `Send to agent` with a single payload shape (`notion_task_id` + handbook repo). Agents and projects are resolved after start via Notion MCP. |
| **Lane = skills + MCP** | Handbook skills load from this repo per Agent profile; MCP availability is fixed per environment. Wrong lane → missing tools or bloated context. |

## Notion (one trigger)

### Primary automation

| Step | Detail |
|------|--------|
| Trigger | **Tasks** → **Agent Status** = `Send to agent` (or equivalent) |
| Payload | `notion_task_id` + always attach handbook **`elimuro/muro-studio-operations`** (or your org’s `owner/muro-studio-operations`) |
| Routing | Default: one Cursor automation URL bound to **`muro-core`** (or your primary ops environment) |
| In-run | Cloud agent re-fetches task → **Agent** relation (profile, skills, autonomy) → **Project** (work repo, Webflow MCP, optional Cursor environment hint) |

```json
{
  "notion_task_id": "…",
  "handbook_repo": "elimuro/muro-studio-operations"
}
```

Work repos are **not** in the webhook. Attach the Project’s **Work repo** only when the task needs git (see [README](../README.md)).

### Optional lane automations (2–3 max)

Add extra Notion automations **only** when lanes need **different Cursor automation URLs** (different prebuilt environments). Same payload; filter on **Agents → MCP** (or a dedicated Agent tag), for example:

| Filter (Agent **MCP**) | Target environment | When |
|------------------------|-------------------|------|
| Figma | `muro-figma` | Design-only runs; no Webflow namespaces on the pod |
| Webflow | `muro-webflow-*` | Further split by **Project → Webflow MCP** in automation formula, or one automation per Webflow lane |
| Notion-only | `muro-core` | Dispatcher / coordinator tasks with no design or CMS connectors |

Do **not** multiply webhooks per agent display name — one payload, filters on MCP or environment hint.

## Cursor environment lanes

Suggested environment names in the Cursor dashboard. Replace `<team>` with your Cursor team slug; store the real environment public id on **Project → Cursor environment** when helpful for humans.

| Environment | MCPs enabled | Typical agents (Notion) | Work repo rule | Dashboard link pattern |
|-------------|--------------|-------------------------|----------------|------------------------|
| **muro-core** | Notion, Airtable (optional), Granola/Dropbox (optional) | Studio Ops Coordinator, Dispatcher | Handbook always; **+** Project **Work repo** when code/PR | `https://cursor.com/dashboard/cloud-agents/environments/muro-core` |
| **muro-figma** | Notion, Figma | Figma Design Expert | Handbook only for MCP tasks; **+** work repo only if task explicitly needs a code repo | `…/environments/muro-figma` |
| **muro-webflow-studio** | Notion, **`webflow-studio`** | Webflow Ship Captain | Handbook + Project work repo when DevLink/code; else handbook only | `…/environments/muro-webflow-studio` |
| **muro-webflow-chapel** | Notion, **`webflow-chapel-sound`** | Webflow Ship Captain | Same as studio lane | `…/environments/muro-webflow-chapel` |
| **muro-webflow-nanaimo** | Notion, **`webflow-nanaimos-mortgage-experts`** | Webflow Ship Captain | Same as studio lane | `…/environments/muro-webflow-nanaimo` |
| **muro-sanity** *(optional)* | Notion, Sanity (+ Airtable if ops tasks need it) | Content Architect | Handbook today; **future:** attach Project work repo + move `sanity-best-practices` out of handbook-only weight (see below) | `…/environments/muro-sanity` |

**muro-core** is the default for mixed ops, Notion-only, and tasks that need multiple non-Webflow connectors without dedicating a Webflow workspace pod.

## Webflow multi-workspace

Eli’s Webflow MCP installs are **separate namespaces** — only **one** may be enabled per Cursor environment:

| Notion **Webflow MCP** (Project) | MCP namespace on the pod |
|----------------------------------|---------------------------|
| `studio` | `webflow-studio` |
| `chapel` | `webflow-chapel-sound` |
| `nanaimo` | `webflow-nanaimos-mortgage-experts` |
| `n/a` | No Webflow lane — do not run Webflow Ship Captain on a Figma-only env |

Tasks resolve namespace from **Project** (select property) or a short hint in the task body. Agents call Webflow tools **only** from that namespace (`webflow-ship-captain` skill). Default if unset: **`webflow-studio`**.

Wrong pairing (e.g. Chapel project on `muro-webflow-studio`) fails at runtime — fix Project **Webflow MCP** or restart the run on the matching environment.

## Projects database

Add on **Projects** (see [notion-schema.md](notion-schema.md)):

- **Webflow MCP** — select: `studio` / `chapel` / `nanaimo` / `n/a`
- **Cursor environment** *(optional)* — text or environment id for humans and automation routing
- Keep **Work repo** — still the only source for attaching product git

## Handbook repo trimming (future)

Today, `sanity-best-practices` lives in this handbook for Content Architect. **Target state:** **`muro-sanity`** environment + attach the client **work repo** on Sanity/code tasks so large Sanity reference trees are not in every pod’s skill cache. Until then, Content Architect can run on `muro-core` or optional `muro-sanity` with the handbook attached.

## Setup checklist

### Per environment

- [ ] Create environment in Cursor; name matches table (`muro-webflow-chapel`, etc.)
- [ ] Enable **only** the MCP list for that row; verify **one** Webflow namespace on Webflow lanes
- [ ] Attach **`muro-studio-operations`** to the environment repo list (always)
- [ ] Trigger **rebuild** / activate a green snapshot after MCP or `environment.json` changes
- [ ] Note dashboard URL on the environment or in an internal runbook

### Per Notion → Cursor automation

- [ ] Payload: `notion_task_id` + handbook repo only
- [ ] Map automation to the correct environment URL
- [ ] Document which Agent **MCP** filter (if any) routes to that URL
- [ ] Smoke test: task with known Agent + Project → run sees expected MCP tools

## Anti-patterns

| Anti-pattern | Problem | Instead |
|--------------|---------|---------|
| All Webflow MCP namespaces on every environment | Huge tool list; wrong-site edits; slow prefix cache | One Webflow namespace per `muro-webflow-*` env |
| Webhook (or automation) per agent display name | Drift, duplicate runs, payload inconsistency | One payload; Agent relation + optional MCP-filtered automations |
| Attach work repo on MCP-only Figma tasks | Unneeded git clone; wrong skills from repo | Handbook only when Project has no **Work repo** |
| Ignoring **Project → Webflow MCP** | Tools target the wrong Webflow account | Set select on every Webflow project; use matching lane |
| Putting default work repo on **Agents** | One agent spans many projects | **Work repo** on **Projects** only |