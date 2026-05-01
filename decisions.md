# Decision log — apr30-h100long-opus47-trace-selfmod-2

Each loop iteration appends one line in the format:

`<ISO timestamp> | next exp_id: <id> | choice: <experiment|controller> | reason: <one sentence>`

2026-04-30T23:14:45Z | next exp_id: exp0001 | choice: experiment | reason: did not consider controller step — this is the first experiment, need baseline signal first
2026-04-30T23:24:59Z | next exp_id: exp0002 | choice: experiment | reason: did not consider controller step — only 1 experiment so far, need baseline signal first
2026-04-30T23:32:48Z | next exp_id: exp0003 | choice: controller | reason: degenerate crossover suggested — primary=elite/1 with secondary forced to baseline elite/0 because no other non-baseline parents exist; choose_operator should gate crossover on >=2 non-baseline elites
2026-04-30T23:34:26Z | next exp_id: exp0003 | choice: experiment | reason: considered controller step; just made one — re-suggest now produces mutate from elite/1 which is non-degenerate, proceed with experiment
2026-04-30T23:42:21Z | next exp_id: exp0004 | choice: controller | reason: degenerate crossover again — primary=elite/2 with secondary=elite/1, but elite/2 was a direct mutate-child of elite/1 (parent1=80d458b), so elite/1 carries no idea elite/2 doesn't already have; choose_operator should require >=2 lineage-independent non-baseline elites
2026-04-30T23:45:19Z | next exp_id: exp0004 | choice: experiment | reason: considered controller step; just made one — re-suggest now produces mutate from elite/2 which is non-degenerate, proceed with experiment
2026-04-30T23:52:23Z | next exp_id: exp0005 | choice: experiment | reason: considered controller step; rejected because the suggestion is non-degenerate (crossover baseline+depth9 yields a novel depth=8+warmdown=0.3 combination) and the search has only been running 4 experiments
2026-05-01T00:02:50Z | next exp_id: exp0006 | choice: controller | reason: suggested crossover(elite/1=depth=9, elite/2=warmdown=0.3) would recreate exp0003 (same depth=9+warmdown=0.3 recipe); root pathology is select_primary keeps picking known-bad elites — add quality-gap penalty
2026-05-01T00:07:14Z | next exp_id: exp0006 | choice: experiment | reason: considered controller step; just made one — re-suggest now produces crossover(elite/0, elite/3) which transplants MATRIX_LR=0.05 onto baseline, a non-duplicate non-degenerate combination
2026-05-01T00:14:11Z | next exp_id: exp0007 | choice: experiment | reason: considered controller step; rejected because exp0006 just produced a new global best (search is now healthy and productive)
2026-05-01T00:21:15Z | next exp_id: exp0008 | choice: experiment | reason: considered controller step; rejected because last 2 experiments produced new global bests and the crossover suggestion is a fresh combination (depth=9 + MATRIX_LR=0.06) not a duplicate
2026-05-01T00:28:20Z | next exp_id: exp0009 | choice: experiment | reason: considered controller step; rejected because the MATRIX_LR sweep on depth=8 has produced 2 new bests (search healthy), continue
2026-05-01T00:35:21Z | next exp_id: exp0010 | choice: experiment | reason: considered controller step; rejected because suggestion is a fresh combination (depth=8 + MATRIX_LR=0.06 + WARMDOWN=0.3), not a duplicate, search still healthy
2026-05-01T00:42:59Z | next exp_id: exp0011 | choice: experiment | reason: considered controller step; rejected because elite/2 is only 0.0026 worse than best (not a known-bad parent) and we have not over-stretched the controller-change budget
2026-05-01T00:49:48Z | next exp_id: exp0012 | choice: experiment | reason: considered controller step; rejected because suggestion is fresh combination (MATRIX_LR=0.07 + EMBEDDING_LR=0.5), not a duplicate
2026-05-01T00:56:47Z | next exp_id: exp0013 | choice: experiment | reason: considered controller step; rejected because elite/3 mutation produces a useful test of EMBEDDING_LR=0.5 in the MATRIX_LR=0.06+warmdown=0.3 region; search still healthy
2026-05-01T01:03:35Z | next exp_id: exp0014 | choice: experiment | reason: considered controller step; rejected because suggestion is exactly the high-priority test from exp0012's reflection (EMBEDDING_LR=0.5 onto MATRIX_LR=0.06 base)
