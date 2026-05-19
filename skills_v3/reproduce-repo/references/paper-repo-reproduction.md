# Paper Repo Reproduction Reference

## Responsibility

Handle repository-level reproduction: scan official materials, identify targets, find the official entrypoint, build an inventory, choose the smallest valid run, and keep the process traceable.

## Repo scan

Read before running heavy commands:

- README and reproduction docs;
- requirements and environment files;
- configs;
- train/eval/infer scripts;
- examples and shell scripts;
- paper/table references;
- issues or model cards when local docs are incomplete.

Build an inventory:

| Component | What to find |
| --- | --- |
| README | official reproduction instructions and target table |
| requirements | Python, CUDA, torch, flash-attn, vLLM, special packages |
| configs | target model, benchmark, split, batch size |
| eval entrypoint | official evaluation script and arguments |
| dataset | dataset ID, local path, split, schema |
| checkpoint | source, file names, fold layout |
| metrics | JSON/CSV/log output and metric names |
| submission | required file name and directory structure |

## Target identification

Confirm:

- target benchmark;
- target method/model/checkpoint;
- target row and columns;
- target split/fold protocol;
- accepted tolerance;
- whether one round, one fold, or full 5-fold is required.

If README and task card disagree, prefer the task card for submission requirements and record the discrepancy in history.

## Workflow

1. Read docs and scripts.
2. Identify official eval command.
3. Identify smallest target-equivalent configuration.
4. Confirm data/checkpoint availability.
5. Run smoke first.
6. Run target submission evaluation.
7. Route failures to bounded recovery.
8. Route outputs to metric extraction.
9. Package `result.md` and `conversation_history/`.

## Source-code and wrapper policy

- Default: do not modify upstream repo code.
- If official code is incompatible with offline/local resources, prefer an external wrapper.
- A wrapper may adapt paths, merge configs, set cache variables, or call existing model/metric functions.
- A wrapper must not change benchmark semantics, labels, splits, or metric formulas.
- Record:

```text
Original repo modified: Yes/No
Wrapper script added: Yes/No
Wrapper reason: <reason>
```

## Lessons from history_v1

The MuQ-Eval v1 run showed good behavior by using an external wrapper instead of patching upstream code. The missing piece was standardized final packaging. Future runs must always turn raw event logs into a clean `conversation_history/history.md` and `run_summary.json`.
