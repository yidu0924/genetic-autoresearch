# Controller journal — apr30-h100long-opus47-trace-selfmod-2

This file is the human-readable memory of why `trace.py` changed across this run.
Every controller-improvement step appends one entry describing the problem, the
planned change, and what behavior should change. After editing `trace.py`, a
follow-up note records what actually changed and the compile sanity check.

---

## 2026-04-30T23:32 — pre-edit note (controller_sha=b233d7b98f69)

**Problem**: at exp0003 the controller suggested `crossover` with primary=elite/1
(DEPTH=9 mutation, val=1.018) and secondary=elite/0 (baseline). The fallback in
`choose_secondary` picks the baseline only because `choose_secondary`'s loop
explicitly skips baseline elites first, and with no other elites available it
falls back to picking the baseline anyway. This is a degenerate crossover —
the baseline carries no "idea" to transplant.

Root cause is in `choose_operator`: the "newest-promotion" trigger forces
crossover as soon as a fresh elite is added, but it does not check whether
there are at least two non-baseline elites available. With a frontier of
{baseline, single-mutate} the only viable secondary is the baseline, which
the existing logic in `choose_secondary` correctly tries to avoid.

**Planned change**: in `choose_operator`, before returning `crossover` from
any branch, check that the frontier has ≥2 elites whose `mutation_kind` is
not `baseline`. If not, return `mutate` instead. This is a one-place gate
that makes the operator decision align with what `choose_secondary` can
actually deliver.

**Expected effect**: while the frontier still contains only one non-baseline
parent, all suggestions will be `mutate` (correctly using the only good
parent we have). Once a second non-baseline elite enters the frontier, the
existing crossover triggers reactivate normally.

## 2026-04-30T23:33 — post-edit note (controller_sha=d650fa1fc126)

**What changed**: Added a single early-return in `choose_operator` after the
existing `len(frontier) < 2` guard: if fewer than 2 non-baseline elites
exist in the frontier, return `mutate`. 9 lines added, no other logic
touched.

**Compile**: `python -m py_compile trace.py` → ok.

**Confirmed behavior change**: re-running `python trace.py suggest` with
the same frontier (elite/0 baseline + elite/1 DEPTH=9) now returns
`operator=mutate, parent1=80d458b (elite/1)` instead of the previous
degenerate `operator=crossover, parent2=elite/0 (baseline)`.

---

## 2026-04-30T23:42 — pre-edit note (controller_sha=d650fa1fc126)

**Problem**: at exp0004 the controller again suggested a degenerate
crossover: primary=elite/2 (depth=9 + warmdown=0.3, val=1.020),
secondary=elite/1 (depth=9, val=1.018). But elite/2's `parent1` is
elite/1's commit — so elite/1 is the direct conceptual ancestor of
elite/2, and the only "idea" elite/1 carries (DEPTH=9) is already in
elite/2. Crossover between a parent and its direct child has no
information content; transplanting any non-redundant property would
either revert the child's mutation or be invented out of thin air
(deviation from suggestion).

The previous (just-installed) `non_baseline_elites < 2` gate doesn't
catch this because we now have 2 non-baseline elites — they just happen
to share the same lineage.

**Planned change**: introduce a "lineage-independent" notion. Two
non-baseline elites A and B are lineage-independent iff neither's
`parent1` commit equals the other's `commit`. Add:

1. A helper `_lineage_independent(a, b)` that does the direct
   parent/child check.
2. In `choose_operator`, replace the `non_baseline_elites < 2` count with
   "exists a pair of lineage-independent non-baseline elites" (using the
   helper). If no such pair, return `mutate`.
3. In `choose_secondary`, also skip secondaries that are not
   lineage-independent of `primary`. Keep the existing description-distance
   filter. The fallback (which ignores filters) stays as a last resort.

This is conservative: it still allows crossover once the search produces
two non-baseline elites from disjoint mutation chains.

**Expected effect**: with the current frontier (elite/0 baseline +
elite/1 depth=9 + elite/2 depth=9+warmdown=0.3, where elite/2 is a
descendant of elite/1), no lineage-independent pair exists, so the
suggestion will become `mutate` from primary (whichever UCB picks).
After a future mutation off elite/0 lands, lineage-independence will
hold and crossover unlocks.

## 2026-04-30T23:43 — post-edit note (controller_sha=80540eb*)

**What changed**: Added helper `_lineage_independent(a, b)` (12 lines) and
two callers: a "has_independent_pair" gate inside `choose_operator` (10
lines) and a `_lineage_independent(primary, row)` skip filter inside
`choose_secondary`'s loop (4 lines). Total +40 lines, no other logic
touched.

**Compile**: `python -m py_compile trace.py` → ok.

**Confirmed behavior change**: re-running `python trace.py suggest` with
the same frontier as before now returns `operator=mutate` instead of the
degenerate `operator=crossover, parent2=elite/1`.

