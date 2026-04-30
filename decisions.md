# Decision log — apr30-h100long-opus47-trace-selfmod-2

Each loop iteration appends one line in the format:

`<ISO timestamp> | next exp_id: <id> | choice: <experiment|controller> | reason: <one sentence>`

2026-04-30T23:14:45Z | next exp_id: exp0001 | choice: experiment | reason: did not consider controller step — this is the first experiment, need baseline signal first
2026-04-30T23:24:59Z | next exp_id: exp0002 | choice: experiment | reason: did not consider controller step — only 1 experiment so far, need baseline signal first
