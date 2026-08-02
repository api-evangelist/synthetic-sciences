---
name: atlas-research-loop
description: Run one full Atlas research loop — brief the project, pose a hypothesis, plan an experiment, record a run with auto-captured code state, compare runs, and commit a decision.
api: https://app.syntheticsciences.ai/api/v1
operations: [brief, hypothesis:add, experiment:plan, run:record, run:compare, leaderboard, decision:add, hypothesis:update]
x-provenance:
  method: generated
  source: https://docs.syntheticsciences.ai/atlas/commands
  grounding: All commands verified against the Atlas Graph CLI & API reference.
---

# Atlas research loop

Drive one delegated research iteration through the Atlas graph. Every command is a
direct call to `https://app.syntheticsciences.ai/api/v1`; authenticate with a `thk_*`
bearer key (`Authorization: Bearer $ATLAS_API_KEY`). Pass `--format=json` for lossless output.

## Preconditions
- `atlas login` (or `export ATLAS_API_KEY=thk_...`) and `atlas doctor --format=json` green.
- A project root exists: `atlas project:create` (or `atlas init` to also write `atlas.json`).

## Steps
1. **Read state first.** `atlas brief --project <id> --format=json` (GET /projects/{id}/brief) returns the context packet: open hypotheses, recent runs, decisions, next actions. Never act before reading the brief.
2. **Pose a falsifiable claim.** `atlas hypothesis:add --project <id> ...` (POST /hypotheses:add) — include assumptions and disconfirmers.
3. **Plan before running.** `atlas experiment:plan --hypothesis <id> ...` (POST /experiments:plan) — declare config, metric, and success criterion up front.
4. **Commit code, then record.** Commit your working tree first, then `atlas run:record --plan <id> --format=json` (/runs:record). Code (git) state is auto-captured onto the run node. Every run carries an outcome (`success` / `failure` / `inconclusive`) and, on failure, a `failure_mode` (`diverged` / `oom` / `data_bug` / `code_bug` / `underperformed` / `other`). Recording failures is first-class — it stops the next agent repeating them.
5. **Compare.** `atlas run:compare --project <id>` (GET /runs:compare) for the matrix, or `atlas leaderboard` to rank runs by one metric.
6. **Decide.** `atlas decision:add --project <id> ...` (POST /decisions:add) — record the choice and the runs/evidence it rests on.
7. **Advance the hypothesis.** `atlas hypothesis:update ...` (POST /hypotheses:update) to move it through its lifecycle.

## Conventions and safety
- **Idempotency:** pass `--idempotency-key <key>` (HTTP `Idempotency-Key`) on retried writes so a replayed record/commit is not duplicated.
- **Retries:** the CLI retries only transient failures (408/425/5xx). A `429` (rate limit) or `409` (conflict) is deliberate — back off / refetch, do not hammer.
- **Destructive ops** require `--yes` (or `--force`).
- Pin a milestone commit with `atlas repo:capture --node <id> --pin true`.
