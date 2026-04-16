# Baseline Gate Checklist Template

Use this compact checklist only when it helps.
The hard requirement is a durable accepted, blocked, waived, or route-changed state.

## Identity

- baseline id:
- route:
- acceptance target:
- primary comparator:

## Core Gate

- [ ] comparator identity and provenance are explicit
- [ ] dataset, split, evaluation path, required metrics, and metric directions are explicit
- [ ] trusted outputs or metrics are traceable
- [ ] expected result files or trusted-output pointers have been checked
- [ ] `<baseline_root>/json/metric_contract.json` exists before acceptance
- [ ] `artifact.confirm_baseline(...)` or `artifact.waive_baseline(...)` is the gate result

## Artifact Flow

- [ ] registry reuse is attached with `artifact.attach_baseline(...)` when applicable
- [ ] reusable publication uses `artifact.publish_baseline(...)` only after verification
- [ ] attach/import/publish is not treated as confirmation by itself

## Blocked Boundary

- [ ] failure class is explicit
- [ ] tried steps and evidence paths are recorded
- [ ] next best move is attach, import, retry, repair, reset, waive, or ask the user

## Closeout

- [ ] concise baseline summary written
- [ ] next anchor named explicitly
