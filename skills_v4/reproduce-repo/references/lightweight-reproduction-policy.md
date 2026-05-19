# Lightweight Reproduction Policy

## Responsibility

Prefer the smallest faithful reproduction path: checkpoint-based evaluation, inference-only runs, benchmark scripts, smoke tests, and minimal subsets before any full training.

## Default Priority

Choose reproduction strategies in this order:

1. Locate or download author-provided checkpoints or pretrained weights.
2. Run checkpoint-based official evaluation.
3. Run checkpoint-based official inference plus official metric script.
4. Use the minimal benchmark evaluation subset that exercises the target metric.
5. Run smoke tests proving imports, model load, data load, forward pass, and metric code.
6. Train only when a training gate is satisfied.

If pretrained checkpoints are available, the agent must prioritize checkpoint-based evaluation reproduction over full training reproduction.

## Forbidden Defaults

Do not default to:

- full training;
- full dataset download;
- multi-day or open-ended runs;
- all-fold training;
- hyperparameter sweeps;
- automatic large model downloads when a smaller official checkpoint can validate the workflow.
- dataset downloads before checkpoint/pretrained-weight discovery.

## Training Gate

Training is allowed only when one of these is true:

- README or official benchmark instructions explicitly require training to reproduce the requested result.
- The user explicitly asks for full training reproduction.
- No checkpoint exists and evaluation cannot be completed without training.

When training is allowed, still start with a short smoke train run, such as one batch or one epoch on a minimal subset, before launching longer work.

If a checkpoint exists but is temporarily unreachable, classify the blocker as checkpoint access/download failure before falling back to training. Do not train merely to avoid resolving checkpoint access unless the user explicitly asks for that path.

## Checkpoint-First Rules

- Search README, releases, model cards, HuggingFace links, Google Drive links, scripts, config defaults, and issues for checkpoints.
- Prefer official checkpoints over retraining in every default benchmark reproduction path.
- Download or locate the checkpoint before downloading full datasets, unless checkpoint discovery itself requires a small metadata-only dataset probe.
- Record checkpoint source, local path, file size, hash when practical, expected architecture, and fold layout.
- Do not duplicate one checkpoint across folds unless official instructions explicitly allow it.

## Smoke Test Rules

A smoke test should verify the pipeline without claiming final metrics:

- environment imports;
- model/checkpoint load;
- dataset schema or sample loading;
- one forward pass or one tiny evaluation batch;
- metric function execution.

Classify smoke metrics as `smoke`. They cannot populate `result.md`.

## Evaluation-First Rules

- Prefer `eval.py`, `evaluate.py`, `test.py`, `benchmark.py`, official shell scripts, or documented commands.
- Prefer official metric scripts over reimplementing metrics.
- Use smaller batch sizes or sample limits only when they do not change benchmark semantics, or classify the run as smoke/diagnostic.
- Treat README/paper numbers as references, not reproduced local results.

## Failure Handling

Attempt bounded repairs that preserve benchmark semantics:

- dependency pins;
- path fixes;
- cache path changes;
- batch size reductions for OOM;
- checkpoint key adaptation when architecture is unchanged;
- wrapper scripts that call official code.

Do not silently change labels, splits, metric formulas, or model architecture.
