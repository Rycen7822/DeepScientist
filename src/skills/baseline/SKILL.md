---
name: baseline
description: Use when a quest needs to attach, import, reproduce, repair, verify, compare, or publish a baseline and its metrics.
skill_role: stage
---

# Baseline

This skill establishes the reference system the quest will compare against.
The goal is one trustworthy comparator with a durable comparison contract, not a reproduction diary.

## Interaction discipline

Follow the shared interaction contract injected by the system prompt.
Keep baseline updates brief unless trust state, blocker state, route, cost, or user-facing risk changed materially.

## Tool discipline

- **Do not use native `shell_command` / `command_execution` in this skill.**
- **All shell, CLI, Python, bash, node, git, npm, uv, and environment work must go through `bash_exec(...)`.**
- **For git work inside the current quest repository or worktree, prefer `artifact.git(...)` before raw shell git commands.**
- Use web search for discovering papers or repos, but use `artifact.arxiv(paper_id=..., full_text=False)` for actually reading a source arXiv paper when it exists.
- Set `full_text=True` only when the short form is insufficient.

## Authority and freedom

The agent owns the execution path.
It may choose the workspace layout, local paths, environment manager, command order, debugging route, smoke strategy, and whether the route is attach, import, verify-local-existing, reproduce, or repair.

Do not treat templates, filenames, `uv`, smoke tests, detached runs, or the phase order as required paths.
They are tactics.
The hard requirement is objective evidence sufficient to accept, block, waive, or switch the baseline route.

Ask the user only when the next move depends on a real scope, cost, permission, data-access, or scientific-preference decision that cannot be inferred from the quest contract.

## Comparator-first rule

The baseline stage is comparator-first, not reproduction-first.
For `comparison_ready`, ask:

- what is the lightest trustworthy comparator?

not:

- how do I reproduce the whole source package most completely?

Default to the lightest baseline path that can still support a fair downstream comparison.
Default to a fast path when it can establish trust with less work.
Do not restart broad discovery, front-load a full audit, or force a fresh memory pass when `requested_baseline_ref`, `confirmed_baseline_ref`, a local comparator, or a concrete command path already exists.
Use `memory.list_recent(...)` or `memory.search(...)` when resuming, reopening old command paths, or avoiding repeated failures.

## Hard artifact flow

The baseline gate is a hard artifact boundary.

- Attach/import/publish alone do not open the downstream gate.
- If reusing a registry baseline, first materialize it with `artifact.attach_baseline(...)` when available, then verify the comparator and metric contract, then call `artifact.confirm_baseline(...)`.
- If publishing a quest-local baseline for reuse, call `artifact.publish_baseline(...)` only after verification is complete and provenance, metrics, and caveats are trustworthy.
- Once the accepted baseline root and trusted comparison contract are clear, call `artifact.confirm_baseline(...)`.
- If the quest must continue without a baseline, call `artifact.waive_baseline(...)` with the reason.
- Do not treat chat, memory, `attachment.yaml`, `PLAN.md`, local files, or published registry entries as a substitute for `artifact.confirm_baseline(...)` or `artifact.waive_baseline(...)`.

## Hard acceptance gates

Baseline success means later stages can compare against one accepted comparator without guessing task, data, split, metric, source, command or evaluation path, provenance, or caveats.

A baseline is successful only when all applicable gates are true:

- the comparator identity is explicit and stable enough for later stages to cite
- the task, dataset, split, evaluation path, required metric ids, metric directions, source identity, and known deviations are durably recorded
- trusted metric values or trusted output pointers are traceable to real files, logs, service responses, source artifacts, or an accepted registry/package record
- verification checked that the evidence came from the intended dataset/split and metric definitions
- `<baseline_root>/json/metric_contract.json` exists as the canonical accepted comparison contract
- `artifact.confirm_baseline(...)` opened the gate, or `artifact.waive_baseline(...)` explicitly bypassed it

The comparison-ready minimum still requires `<baseline_root>/json/metric_contract.json`.
Once a comparison-ready baseline is durably confirmed, stop baseline work unless one named unresolved comparison risk remains.

## Acceptance targets

- `comparison_ready`: the default target; one comparator is trustworthy enough for downstream comparison
- `paper_repro_ready`: the baseline supports paper-facing reproduction or comparison claims
- `registry_publishable`: the package is reusable enough to publish as a durable baseline package
- `blocked`: the route cannot clear the gate cleanly and the next move is explicit
- `waived`: the quest continues without a baseline for a recorded reason

Not every baseline needs a paper-grade exact reproduction.
A verified attached, imported, or local-existing comparator can be enough for `comparison_ready`.

## Route success criteria

Choose the route that maximizes trust per unit time and compute.
Keep one dominant route active at a time.
If a lighter route satisfies the current acceptance target, stop there.

- `attach`: baseline identity, provenance, trusted outputs pointer, core metric contract, `attachment.yaml` or equivalent provenance, and `artifact.confirm_baseline(...)` are complete
- `import`: package is materialized/readable in the quest, provenance is durable, trusted outputs or metrics are traceable, and `artifact.confirm_baseline(...)` is complete
- `verify-local-existing`: local path or service, command or endpoint, output location, required metrics, and core metric contract are verified and confirmed
- `reproduce`: source identity, command or evaluation path, expected outputs, verification evidence, deviations, and metric contract are explicit and confirmed
- `repair`: broken point is identified, bounded fix or route change is made, rerun or re-read evidence supports the new trust state, and the result is accepted or blocked

## Core metric contract

The baseline stage is not complete just because something ran.
Before declaring a baseline usable, make these fields explicit:

- task identity
- dataset identity and split contract
- evaluation script or evaluation path
- required metric keys for the current downstream comparison
- metric directions
- source commit, source package, local service, or registry identity
- known deviations from the source reference

Metric-contract rules:

- keep `primary_metric` as the headline metric only; do not let it erase the rest of the comparison surface
- submit canonical `metrics_summary` as a flat top-level dictionary keyed by the paper-facing metric ids
- every canonical baseline metric entry should include `description`, either `derivation` or `origin_path`, and `source_ref`
- mark only currently required canonical metrics as required; additional metrics can be supplementary
- preserve multiple metrics, datasets, subtasks, splits, and variants when they matter
- do not collapse multiple metrics, datasets, subtasks, or splits into one misleading scalar
- if the source package already has a richer leaderboard table, structured result file, or `json/metric_contract.json`, reuse that richer contract instead of hand-writing a thinner one
- `Result/metric.md` is optional scratch memory only; reconcile it before `artifact.confirm_baseline(...)`
- for stable payload shapes, read `references/artifact-payload-examples.md`

If later `experiment` work would still have to guess the comparison contract, the baseline is not ready.
For a compact verdict rubric, read `references/comparability-contract.md`.

## Verification

Verification is mandatory before baseline acceptance.
Verify:

- the run, service call, package import, or trusted-output inspection actually finished
- metrics came from the intended dataset and split
- metric definitions and directions match the quest contract
- results are comparable to the paper, source repo, local comparator, registry package, or selected target
- deviations are stated rather than silently normalized away

Use conservative verdicts: `verified_match`, `verified_close`, `verified_diverged`, `trusted_with_caveats`, or `broken`.

## Durable route records

Durable records are required in substance, not in fixed filenames.
For non-trivial, code-touching, expensive, unstable, or long-running work, leave a route record with:

- route and acceptance target
- comparator and source identity
- command, endpoint, or evaluation path if one exists
- expected outputs or trusted-output pointers
- acceptance condition
- blocker or fallback
- verification verdict

`PLAN.md`, `CHECKLIST.md`, `setup.md`, `execution.md`, `verification.md`, `analysis_plan.md`, and `REPRO_CHECKLIST.md` are compatibility surfaces, not mandatory success paths.
Use `references/baseline-plan-template.md` and `references/baseline-checklist-template.md` only when they reduce ambiguity.

`attachment.yaml` or equivalent provenance is required for attached or imported baselines.

## Execution tactics

- If source reproduction or repair is active, read the source paper and source repo before substantial setup.
- For attach, import, or verify-local-existing, inspect only the minimum evidence needed to trust the comparator.
- A bounded smoke test is useful only when command path, environment viability, evaluator wiring, or output schema is uncertain.
- Treat smoke/pilot work as a `0-2` default budget; do not repeat an unchanged check without new evidence, code/environment change, or route change.
- If runtime is uncertain or likely long, prefer `bash_exec(mode='detach', ...)` plus managed monitoring.
- If a run is invalid, wedged, or superseded, stop it cleanly and relaunch only after the route changed.

## Environment tactics

For Python baselines, prefer a reproducible isolated environment, but choose the route most faithful to the source package.
`uv` is a useful default tactic when no stronger repo-native route exists.
Examples include `uv sync`, `uv venv`, `uv pip install ...`, `uv run ...`, and `uv run python ...`.
Switch to repo-native conda, docker, poetry, shell scripts, or service startup when that is more trustworthy or required.
Do not force a global `uv` route when it would make the reproduced baseline less faithful.

## Negative cases and stop rules

Do not accept a baseline when:

- metrics are fabricated, copied, or paraphrased without provenance
- metrics are copied from a paper while the acceptance target requires local verification
- dataset, split, metric direction, or evaluation path is materially unknown
- outputs cannot be tied to the intended command, source, comparator, package, or service
- a local run used a materially different protocol without a caveat
- source code changes alter baseline scope without recording the deviation
- a package imports but trusted metrics or outputs are not traceable
- later experiment work would still need to guess required baseline metric ids

If the same failure class appears again without new evidence, code changes, environment changes, or a route change, stop looping and route through `repair`, `decision`, `blocked`, `waive`, or one bounded clarification.

A blocked result must state what failed, what was tried, which paths or logs show the issue, and whether the next best move is attach, import, retry, repair, reset, waive, or ask the user.

## Memory and reuse

Use memory only to avoid repeated failures or preserve reusable baseline lessons.
Promote to global memory only when another quest is likely to benefit.
Do not publish a baseline for reuse if verification is incomplete, metrics are untrusted, or provenance is weak.

## Baseline id and variant rules

- keep `baseline_id` short, stable, and filesystem-safe
- prefer one baseline id with stable variant names over near-duplicate ids
- if multiple comparators exist, mark the primary downstream baseline

## Exit criteria

Exit once one of these is durably true:

- a baseline is attached/imported/local-existing/reproduced/repaired, verified, and accepted through `artifact.confirm_baseline(...)`
- a broken route is recorded with a next decision
- a waiver decision leaves the baseline gate through `artifact.waive_baseline(...)`
- a route change is recorded because the previous route is no longer the best trust-per-cost path

A good baseline pass leaves one trusted comparator, one explicit blocker, or one explicit route change, not a vague promise to keep rechecking baseline.
