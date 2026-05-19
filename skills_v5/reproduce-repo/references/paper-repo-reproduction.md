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
| dataset | dataset ID, local path, split, schema, estimated size, minimal subset |
| checkpoint | pretrained/author checkpoint source, file names, size, fold layout |
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
3. Search for author-provided pretrained checkpoints or model weights.
4. Prefer checkpoint-based evaluation or inference reproduction.
5. Identify smallest target-equivalent evaluation subset.
6. Estimate dataset size and check disk before any dataset download.
7. Run smoke first.
8. Run target submission evaluation.
9. Train only if no checkpoint exists, evaluation cannot be completed otherwise, and training is required.
10. Route failures to bounded recovery.
11. Route outputs to metric extraction.
12. Package `result.md` and raw `history.md`.

## Checkpoint-first policy

- If pretrained checkpoints are available, prioritize checkpoint-based evaluation or inference reproduction over full training reproduction.
- Do not choose a training entrypoint while an official checkpoint evaluation path exists.
- Do not download complete training datasets before checkpoint discovery.
- If a checkpoint link is broken or gated, record that blocker before considering training as a fallback.

## Dataset and training policy

- Do not default to full dataset download.
- Do not default to full or long training.
- If dataset download is necessary, estimate size, check free disk, prefer the smallest necessary subset, and make the reason visible in logs or the raw history trace.
- Treat full training as a last resort, not the standard reproduction path.

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

The MuQ-Eval v1 run showed good behavior by using an external wrapper instead of patching upstream code. The missing piece was final packaging and history preservation. Future runs must package only `result.md` and raw `history.md`; `history.md` must be the original Codex interaction trace, not a cleaned reproduction report.
