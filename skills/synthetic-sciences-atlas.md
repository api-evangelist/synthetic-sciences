---
name: atlas
description: Operate Atlas by Synthetic Sciences from the CLI. Use for the research loop (brief, hypotheses, experiment plans, recorded runs, decisions), research maps, source indexing, grounded search, evidence, exports, and keys.
license: MIT
compatibility: Requires Node.js 18+ and the @synsci/atlas CLI.
metadata:
  author: Synthetic Sciences
  package: "@synsci/atlas"
x-provenance:
  method: searched
  source: https://app.syntheticsciences.ai/documentation/skill.md
  saved_verbatim: true
---

# Atlas CLI and skills

Use this when an agent needs to read or write Atlas state from the command
line. Atlas is direct REST plus CLI; do not add a public MCP transport.

## Install and verify

```bash
npm i -g @synsci/atlas@latest
atlas login
atlas doctor --format=json
atlas install          # wire the bundled skills into detected coding agents
```

## Session bootstrap (the research loop)

```bash
atlas init                           # dedupe-aware find-or-create + write atlas.json
atlas brief --project <id>           # context packet: read before acting
```

Then: `hypothesis:add` → `experiment:plan` → `run:record --plan <id>` →
`leaderboard` / `run:compare` → `decision:add` → `hypothesis:update`.
Code state is auto-captured onto every recorded run — commit before
recording; pin milestones with `repo:capture --node <id> --pin true`.

## Ambient mode (only in atlas.json projects)

`atlas init` writes `atlas.json` (project id + `auto` block), the opt-in
activation contract. When present, stay near-invisible and fail open:

- Prefer `web:search` / `web:fetch` over ad-hoc WebSearch/WebFetch so sources
  are metered and logged to the project; `library:*` auto-scopes to
  `source_ids`.
- Drop compact breadcrumbs with `log:append`; read them with `log:tail` /
  `log:search`.
- `atlas install` also wires a post-tool hook (Claude Code + Cursor) that logs
  web activity automatically (no-op outside atlas.json projects; `--no-hooks` to
  skip). Outside an Atlas project, behave normally.

## Rules

- Atlas stores research as maps: nodes, links, evidence, sources, runs, and decisions.
- Sharing is graph-scoped: visibility (private/unlisted/public) and roles (viewer/editor/admin) live on the project root and descendants inherit; child nodes have no separate visibility. Public graphs are browsable and forkable with `atlas fork --node <id>` (preserves code-state + provenance).
- Skills are bundled inside `@synsci/atlas`; do not install a separate Atlas skill bundle.
- Output is JSON when piped; force with `--format=json` for automation.
- Do not print full Atlas API keys into shared logs.
- Use `research_paper` for paper sources.
- Slugs (`able-helm-1359`) work everywhere ids do: `--project`, `--node`, `--plan`, `--hypothesis`.

## Core command families

- Setup: `init` (writes atlas.json), `install`, `doctor`
- Research loop: `brief`, `hypothesis:*`, `experiment:plan`, `decision:add`, `eval:define`, `run:record/compare`, `leaderboard`, `reproduce`, `note:add`, `notebook:cell`
- Ambient journal: `log:append`, `log:tail`, `log:search`
- Maps: `node:*`, `map:*`, `link:*`, `access:*`, `draft:*`, `label:*`, `project:*`
- Library/research: `library:*` (search `--save-to`, ask `--attach`), `web:search`, `web:fetch`, `research:*`, `usage:summary`
- Evidence: `evidence:*`
- Code state: `repo:capture/pin/push`, `github:*`
- Sharing & forks: `node:share` (graph-scoped visibility), `access:*` (viewer/editor/admin on the root), `fork` (copy a visible public graph into your account)
- Export/import & summaries: `map:export`, `map:import`, `map:summary:*`, `history:export`
- Keys/config: `key:*`, `config:*`, `secret:*`

## Bundled Atlas skills

`atlas` (router + map reading), `atlas-lab` (record runs + the research
loop), `atlas-frontier` (plan or autonomously advance the frontier),
`atlas-optimize` (GEPA loop over one artifact), `atlas-autoresearch`
(refereed campaign), `atlas-map` (source-to-map authoring),
`atlas-search` (library search/Q&A), `atlas-reproduce` (validate results),
and `atlas-paper` (write a paper from a graph).
