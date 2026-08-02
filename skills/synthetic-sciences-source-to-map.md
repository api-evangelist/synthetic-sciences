---
name: atlas-source-to-map
description: Index research material into the Atlas Library, search and grounded-ask over it, and save the useful results onto graph nodes as durable, cited map state.
api: https://app.syntheticsciences.ai/api/v1
operations: [library:add, library:list, library:search, library:ask, research:ask, usage:summary]
x-provenance:
  method: generated
  source: https://docs.syntheticsciences.ai/library/commands
  grounding: All commands verified against the Atlas Library CLI & API reference.
---

# Atlas source-to-map

Turn indexed sources into grounded, cited map state. Auth is a `thk_*` bearer key.
Most Library operations are included; research answers and managed compute are metered —
check `atlas usage:summary` (GET /usage/summary) before an autonomous session.

## Steps
1. **Add sources.** `atlas library:add <url>` (POST /sources), or `--path <dir>` to index a local folder (private by default). Source types: `repository`, `documentation`, `research_paper`, `huggingface_dataset`, `local_folder`. Use `research_paper` for papers — never `paper`.
2. **Confirm indexing.** `atlas library:list --format=json` (GET /sources); `atlas library:show <id>` (GET /sources/{id}) for status; `--errored` filters failures.
3. **Search (included).** `atlas library:search "<query>" --mode universal --save-to <node>` (POST /search). Prefer `--mode universal` for routine lookups; reserve `--mode deep` (deducts `deep_research` quota) for questions that justify it. `--save-to` persists hits onto a graph node.
4. **Grounded Q&A.** `atlas library:ask "<question>" --attach <node>` (POST /documents/ask) for an answer grounded in your indexed sources, attached to a node.
5. **Metered synthesis (optional).** `atlas research:ask "<question>"` (POST /research/oracle) runs Oracle-style synthesis over the library and deducts from the `oracle` quota. Prefer `library:search` (included) when a grounded match is enough.
6. **Long-running work.** Use async jobs: `atlas library:jobs:create` / `research:jobs:create`, then `...:jobs:stream <job_id>` to follow events; cancel with `...:jobs:cancel <job_id> --yes`.

## Conventions
- Everything saved becomes durable, forkable, provenance-carrying map state (`atlas fork --node <id>`).
- Watch quotas: stop the session when `oracle` or `deep_research` is exhausted (429 on metered surfaces).
- Output is JSON when piped; force with `--format=json`.
