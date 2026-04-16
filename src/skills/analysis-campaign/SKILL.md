---
name: analysis-campaign
description: Use when a quest needs one or more follow-up runs such as ablations, robustness checks, error analysis, or failure analysis after a main experiment.
skill_role: stage
---

# Analysis Campaign

Use this skill when follow-up evidence is needed after a durable result.
The goal is to answer a bounded evidence question, not to keep opening more slices because they are imaginable.
All supplementary experiments after a durable result use this shared protocol: ordinary analysis, review-driven evidence gaps, rebuttal-driven extra runs, and write-gap follow-up experiments.

## Interaction discipline

Follow the shared interaction contract injected by the system prompt.
Keep campaign updates brief unless evidence boundary, blocker state, cost, or next route changed materially.
For ordinary active work, prefer a concise progress update once work has crossed roughly 6 tool calls with a human-meaningful delta, and do not drift beyond roughly 12 tool calls or about 8 minutes without a user-visible update.
For meaningful long-running slices, include the estimated next reply time or next check-in window whenever defensible.

## Tool discipline

- **Do not use native `shell_command` / `command_execution` in this skill.**
- **All shell, CLI, Python, bash, node, git, npm, uv, and environment work must go through `bash_exec(...)`.**
- Use managed `bash_exec` sessions for long-running slices instead of relaunching blindly.

## Authority and freedom

The agent owns the analysis path: slice ordering, workspace layout, filenames, environment route, monitoring strategy, and whether to use smoke, direct verification, or the real run first.

Do not treat `PLAN.md`, `CHECKLIST.md`, paper-matrix files, smoke tests, detached runs, `tqdm`, or a fixed phase order as required paths.
They are tactics.
The hard requirement is traceable slice-level evidence that changes, confirms, or blocks the evidence boundary of the parent claim and leaves an explicit next route.

Artifact state is not optional for supplementary experiments that launch slices.
If a slice is launched as part of analysis-campaign, use the artifact flow below.

## Hard artifact flow

Analysis-campaign has a hard artifact boundary.

- Before launching slices, call `artifact.create_analysis_campaign(...)` with the currently justified slice list when the work is an analysis campaign, affects durable lineage, needs Canvas/branch visibility, supports paper/rebuttal/review claims, or has more than one slice.
- Even one extra experiment can still be represented as a one-slice campaign when durable lineage matters.
- If `artifact.create_analysis_campaign(...)` returns slice worktrees, run each returned slice in its returned workspace unless a recorded reason makes another location more faithful.
- After each launched slice finishes, fails, becomes infeasible, or is superseded, call `artifact.record_analysis_slice(...)` with the honest outcome.
- Do not replace `artifact.record_analysis_slice(...)` with chat, memory, a local note, or a campaign summary for any launched slice.
- If no slice is launched and the answer is only a bounded read-only audit, a durable report or decision may be enough; do not call that a completed campaign slice.
- When ids or refs are unclear, recover them with `artifact.resolve_runtime_refs(...)`, `artifact.get_analysis_campaign(...)`, `artifact.get_quest_state(...)`, or `artifact.list_paper_outlines(...)` instead of guessing.

For writing-facing campaigns, include available paper-mapping fields such as `selected_outline_ref`, `research_questions`, `experimental_designs`, and `todo_items` when they exist and matter.
Treat `campaign_id` as system-owned, and treat `slice_id` / `todo_id` as agent-authored semantic ids.

## Hard success gates

An analysis campaign succeeds when it changes or confirms the evidence boundary of a parent claim with traceable slice-level evidence, preserves comparability or records why comparability broke, and leaves a durable next-route decision.

Before treating analysis as successful, all applicable gates must be true:

- parent object is explicit: main run, accepted idea line, paper gap, reviewer item, rebuttal item, or route decision
- claim, question, failure mode, or decision being tested is explicit
- every launched slice has a durable artifact outcome: completed, partial, failed, blocked, infeasible, or superseded
- every evidence-bearing slice records question, intervention or inspection target, fixed conditions, metric or observable, evidence path, claim update, comparability verdict, and next action
- null, negative, failed, partial, and contradictory findings remain visible
- campaign-level interpretation is derived from per-slice evidence, not impressions
- next route is explicit: continue campaign, return to `experiment`, return to `idea`, move to `write`, route through `decision`, stop, reset, or record a blocker

Do not aggregate campaign conclusions without per-run evidence.
Do not bury null or contradictory findings.

## Analysis routes

Use the lightest route that preserves trust and downstream utility, including efficiency or cost questions when they affect the claim.

- `analysis-lite`: one clear follow-up question and one slice or very small slice set, including small highlight-validation or efficiency / cost checks when they affect the claim
- `artifact-backed campaign`: any launched supplementary slice that needs durable lineage, branch/worktree isolation, Canvas visibility, or later replay
- `writing-facing campaign`: evidence supports a selected outline, paper experiment matrix, evidence ledger, section, claim, or table
- `review/rebuttal campaign`: evidence answers reviewer pressure or audit findings
- `failure-analysis route`: evidence explains why a result failed, diverged, or became non-comparable

Useful slice classes:

- `auxiliary`: helps understand settings, thresholds, or mechanisms but does not carry the main claim by itself
- `claim-carrying`: directly affects whether the main narrative or route decision is justified
- `supporting`: broadens confidence or interpretability after the main claim is credible

Start the smallest route that can answer the current question.
Run claim-critical slices first and stop widening once the next route is clear.
For campaign prioritization and writing-facing slice design, read `references/campaign-design.md`; for mapping examples, read `references/writing-facing-slice-examples.md`.

## Slice evidence contract

For each meaningful slice, define and record:

- research question
- hypothesis, expected pattern, or decision-relevant expectation
- intervention, ablation, variation, inspection target, or failure bucket
- controls or fixed conditions
- metric, observable, table, qualitative artifact, or rubric
- comparison target
- stop or completion condition
- evidence path
- claim update
- comparability verdict
- next action

Code-based, fully automatable analysis is preferred when it is the most faithful and repeatable path.
Failure-bucket inspection, qualitative artifact review, extracted-text audits, reviewer-linked example checks, and table/figure consistency checks can still be valid when evidence is concrete, scoped, and reproducible enough for the claim.
Do not present subjective judgment as objective measurement; record rubric, sample, prompt or inspection basis, caveats, and why it is sufficient.

`evaluation_summary` is the preferred stable routing summary for UI, Canvas, review, and rebuttal.
When useful, include `takeaway`, `claim_update`, `baseline_relation`, `comparability`, `failure_mode`, and `next_action`.

## Comparability contract

Comparability is a hard boundary.

- keep the same evaluation contract unless the variation is the point
- when `active_baseline_metric_contract_json` exists, read it before defining slice success criteria or comparison tables when baseline comparison matters
- when `active_baseline_metric_contract_json` exists, keep slice comparisons aligned with it unless the slice explicitly records why it differs
- state what changed and what stayed fixed
- if the variation changes evaluation setup, record that and do not present it as direct apples-to-apples comparison

A new dataset can be valid as a generalization, external-validity, stress-test, or limitation-boundary slice, but it must be labeled that way and must not replace the accepted baseline or main comparison contract.
If a slice needs an extra comparator baseline, place it under normal baseline roots, do not overwrite the canonical quest baseline gate, and record it through `record_analysis_slice(..., comparison_baselines=[...])`.

## Writing-facing boundary

Paper-facing evidence must be write-backable.
Paper-ready slices must map cleanly back to a selected outline, paper experiment matrix, evidence ledger, section, claim, table, reviewer item, or rebuttal item.

- if the slice is the only thing keeping a main-text section unsupported, mark it `main_required` / `main_text`
- if useful but non-blocking, mark it `appendix`
- if informative but not manuscript-bound, keep it durable and mark it `reference_only`
- if `paper/paper_experiment_matrix.md` exists and the campaign supports the paper, read it before launching or reordering slices
- prefer stable ids such as `exp_id`, `todo_id`, or `slice_id`
- paper-ready slices should carry available write-back fields such as `paper_role`, `section_id`, `item_id`, and `claim_links`
- after completing a paper-ready slice, update or verify the relevant matrix, section notes, evidence ledger, or active paper-line summary

If no selected outline exists yet but the evidence question decides whether writing is worthwhile, run it as pre-outline analysis and route to `write` or `decision` afterward.

## Durable route records

Durable records are required in substance, not in fixed filenames.
For multi-slice, writing-facing, route-changing, expensive, unstable, or long-running analysis, leave a route record with:

- parent object and parent claim
- acceptance or stop condition
- slice frontier
- comparability boundary
- available assets and required comparators
- evidence paths or expected outputs
- blocker or fallback
- next route after success or failure

`PLAN.md`, `CHECKLIST.md`, `paper/paper_experiment_matrix.md`, and local matrix/checklist files are control surfaces, not mandatory success paths.
Use `references/campaign-plan-template.md` and `references/campaign-checklist-template.md` only when they reduce ambiguity.

## Execution tactics

- Use a smoke test when slice command, outputs, metric path, or evaluator wiring is uncertain.
- Treat smoke work as a `0-2` default budget; do not repeat unchanged checks.
- If runtime is uncertain or likely long, use `bash_exec(mode='detach', ...)` plus managed monitoring.
- `bash_exec(mode='read', id=...)` returns full logs when 2000 lines or fewer; longer logs return the first 500 lines plus last 1500 lines.
- Use `bash_exec(mode='read', id=..., start=..., tail=...)` for omitted middle sections.
- Monitor with `tail_limit=..., order='desc'`, then `after_seq=last_seen_seq` for incremental reads.
- Recover unclear ids with `bash_exec(mode='history')`.
- Use `silent_seconds`, `progress_age_seconds`, `signal_age_seconds`, and `watchdog_overdue` as stall checks when available.
- Stop invalid, wedged, or superseded slices with `bash_exec(mode='kill', id=..., wait=true, timeout_seconds=...)`.
- If waiting on an existing session, prefer `bash_exec(mode='await', id=..., timeout_seconds=...)`; otherwise use `bash_exec(command='sleep N', mode='await', timeout_seconds=N+buffer, ...)` and do not set `timeout_seconds` exactly equal to `N`.
- If you only need wall-clock waiting between checks, use the canonical sleep choice above.
- prefer `bash_exec(mode='await', id=..., timeout_seconds=...)` instead of starting a new sleep command when you are already waiting on a launched slice
- When you control slice code, prefer a throttled `tqdm` progress reporter and concise structured progress markers when feasible.

## Negative cases and stop rules

Do not treat analysis as successful when:

- slices do not map to a parent claim, parent result, paper gap, reviewer item, or decision
- summary claims stable support without per-slice evidence
- negative, null, contradictory, failed, or partial slices are hidden
- an ablation changes many factors but is interpreted as isolating one factor
- a robustness slice changes dataset, split, or protocol but is reported as direct comparison
- subjective or manual inspection supports a claim without rubric, sample, prompt, trace, or caveat
- a writing-facing slice is called paper-ready but cannot be mapped back to paper artifacts
- a failed slice is silently skipped and replaced by a different slice
- the campaign expands after the next route is already clear
- a new comparator overwrites the canonical quest baseline gate instead of analysis-local comparison evidence
- the underlying main result is untrusted and the work is really baseline recovery or a new main experiment
- a new main experiment is disguised as analysis to bypass the main-experiment gate

If the same failure class appears again without a real route or evidence change, stop widening and route through `decision`, `write`, `experiment`, or an explicit blocker.
If two slices in a row fail to change claim boundary, matrix frontier, or next route, stop widening and route through `decision`, `write`, `experiment`, or an explicit blocker.

A blocked campaign must state the failure class, what was tried, evidence paths, and next best action.

## Aggregation and memory

Campaign reporting should classify stable support, partial support, contradiction, and unresolved ambiguity.
It should state whether the main claim is strengthened, weakened, narrowed, abandoned, or still ambiguous.
Summarize the top `3-5` findings first when there are many slices.

Use memory only to avoid repeated failures or preserve reusable campaign lessons.
At stage end, write `memory.write(...)` only for durable cross-slice lessons, failure patterns, or comparability caveats.

## Exit criteria

Exit once one of these is durably true:

- enough evidence exists for writing or decision-making
- a problem requires returning to `experiment`, `idea`, baseline recovery, or `decision`
- the campaign is blocked and the blocker is recorded
- the route changed because the original slice set is no longer the best evidence-per-cost path

A good campaign closes when the claim got stronger, weaker, narrower, abandoned, or clearly stuck, not when more slice ideas remain possible.
