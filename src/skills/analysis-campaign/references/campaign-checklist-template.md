# Analysis Evidence Gate Checklist

Use this compact checklist only when it helps.
The hard requirement is durable launched-slice outcomes, evidence boundaries, blockers, and next routes.

## Identity

- parent object:
- parent claim or gap:
- route:
- campaign id:

## Evidence Gate

- [ ] parent claim, paper gap, reviewer item, or decision is explicit
- [ ] each launched slice has a durable outcome or active monitoring path
- [ ] evidence-bearing slices record question, intervention or inspection target, fixed conditions, metric or observable, and evidence path
- [ ] claim update and comparability verdict are explicit
- [ ] null, negative, partial, failed, blocked, or contradictory findings are visible
- [ ] campaign-level interpretation is backed by per-slice evidence

## Artifact Gate

- [ ] `artifact.create_analysis_campaign(...)` is used for launched campaign slices
- [ ] `artifact.record_analysis_slice(...)` is used after each launched slice finishes, fails, becomes infeasible, or is superseded
- [ ] local notes, chat, memory, or final summary are not substitutes for launched-slice artifact state

## Comparability Gate

- [ ] baseline or main comparison contract is preserved, or deviation is recorded
- [ ] new dataset / split / metric / protocol changes are labeled
- [ ] additional comparators do not overwrite the canonical quest baseline gate

## Paper / Review Gate

- [ ] paper-ready slices map to outline, paper matrix, evidence ledger, section, claim, table, reviewer item, or rebuttal item
- [ ] completed paper-facing slices update the write-back target or record a stale-contract blocker

## Closeout

- [ ] strongest evidence boundary is summarized
- [ ] main claim is classified as strengthened, weakened, narrowed, abandoned, or still ambiguous
- [ ] next route recorded explicitly
