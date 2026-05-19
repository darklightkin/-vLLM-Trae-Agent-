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

### Verify

After repo scan, verify the local understanding against the repository's own materials before installing dependencies or running heavy commands.

Compare:

- repository name, commit or release tag when available;
- official Python/package/CUDA requirements;
- official eval/infer/train entrypoints;
- official config files and default arguments;
- model names, method names, benchmark names, dataset names, split names, and metric names;
- required output files and submission layout.

Record expected value, actual local value, source of expectation, match status, and mismatch action in `change_summary.md`.

## Target identification

Confirm:

- target benchmark;
- target method/model/checkpoint;
- target row and columns;
- target split/fold protocol;
- accepted tolerance;
- whether one round, one fold, or full 5-fold is required.

If README and task card disagree, prefer the task card for submission requirements and record the discrepancy in history.

### Verify

After target identification, verify the selected target against the paper, README, config, model card, and task card.

Do not rename or normalize official model names, method names, table row labels, dataset split names, or metric names. If two official sources use different spellings, preserve the spelling required by the submission target and record the discrepancy in `change_summary.md`.

## Workflow

1. Read docs and scripts.
2. Verify repository-declared requirements and official names.
3. Identify official eval command.
4. Verify target command, config, metric names, split/fold, and model names.
5. Search for author-provided pretrained checkpoints or model weights.
6. Verify checkpoint identity and compatibility.
7. Prefer checkpoint-based evaluation or inference reproduction.
8. Identify smallest target-equivalent evaluation subset.
9. Estimate dataset size and check disk before any dataset download.
10. Verify dataset identity, version, split, schema, and preprocessing.
11. Run smoke first.
12. Verify smoke output and artifact paths.
13. Run target submission evaluation.
14. Verify final command, config, dataset, checkpoint, model name, and metrics.
15. Train only if no checkpoint exists, evaluation cannot be completed otherwise, and training is required.
16. Route failures to bounded recovery.
17. Route outputs to metric extraction.
18. Verify metric provenance.
19. Package `result.md`, raw `history.md`, and `change_summary.md`.

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
- Every agent-made edit, wrapper, config copy, command-line change, path adaptation, or environment workaround must be summarized in `change_summary.md`.
- For each change, compare the before/after behavior against the repository's official version or instruction.
- Record:

```text
Original repo modified: Yes/No
Wrapper script added: Yes/No
Wrapper reason: <reason>
Official behavior changed: Yes/No
Verify result: match/acceptable-drift/mismatch/unknown
```

## Lessons from history_v1

The MuQ-Eval v1 run showed good behavior by using an external wrapper instead of patching upstream code. The missing piece was final packaging and history preservation. Future runs must package only `result.md` and raw `history.md`; `history.md` must be the original Codex interaction trace, not a cleaned reproduction report.
