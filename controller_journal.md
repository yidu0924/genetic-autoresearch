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

