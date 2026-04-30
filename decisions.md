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
