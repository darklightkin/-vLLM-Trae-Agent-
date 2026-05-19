---
name: reproduce-repo
description: Use when the user asks to reproduce paper repository results, benchmark tables, evaluation metrics, or scientific GitHub repositories with low human interaction, including repo exploration, isolated environment creation, dependency repair, checkpoint-first reproduction, minimal dataset handling, smoke tests, benchmark evaluation, metric extraction, final result packaging, run summary generation, and complete Codex conversation history preservation.
---

# Reproduce Repo

## Purpose

Reproduce a paper repository, benchmark row, table result, or evaluation metric with minimal human interaction, isolated execution, and auditable outputs.

This is the only triggerable skill entrypoint. Load reference files only when their stage is active.

## Core Defaults

- Create a fresh isolated environment for each repo by default.
- Prefer conda. Fall back to venv only when conda is unavailable.
- Do not reuse existing conda/venv environments unless the user explicitly requests reuse.
- Do not use `base`, system Python, a previous repo environment, or any discovered old environment as an automatic fallback.
- Prefer author checkpoints, inference-only reproduction, minimal evaluation subsets, smoke tests, and benchmark evaluation.
- If pretrained checkpoints are available, the agent must prioritize checkpoint-based evaluation reproduction over full training reproduction.
- Do not default to full training, TB-scale downloads, or long-running training.
- Ask fewer questions: infer from repository artifacts whenever safe.
- Preserve the real Codex conversation history. `history.md` must preserve the original Codex conversation trace instead of a rewritten or summarized reproduction report.
- Final submission directory contains only `result.md` and `conversation_history/`.

## Required Workflow

1. Scan repository docs, scripts, configs, model cards, examples, issues, and papers before heavy execution.
2. Identify the target benchmark, method, checkpoint, split, metric, and official entrypoint.
3. Create a fresh repo-specific isolated environment.
4. Install dependencies inside that environment only.
5. Find and prepare author checkpoints or pretrained weights before dataset-heavy work.
6. Prefer checkpoint-based inference/evaluation reproduction.
7. Estimate dataset size and check disk before any dataset download; use the smallest valid evaluation subset first.
8. Run smoke tests before benchmark evaluation.
9. Run benchmark evaluation only after the environment, checkpoint, and data gates pass.
10. Train only when no checkpoint is available and evaluation cannot be completed, or when official instructions/user request explicitly require training.
11. Attempt bounded automatic repairs without changing benchmark semantics.
12. Extract metrics only from real local artifacts with provenance.
13. Create the final submission directory with only:

```text
repo_name/
  result.md
  conversation_history/
```

`conversation_history/` must contain the complete raw Codex conversation export plus `run_summary.json`.
`conversation_history/history.md` must be the original Codex conversation trace, including user messages, assistant messages, tool calls, repair attempts, and execution traces. It must not be a cleaned-up narrative report.

## Reference Loading

Load these references when needed:

- `references/isolated-environment-management.md`: fresh environment policy, conda/venv creation, Python inference, CUDA compatibility, naming, cleanup, cache handling.
- `references/lightweight-reproduction-policy.md`: checkpoint-first, evaluation-first, inference-only, smoke-test-first workflow and training gate.
- `references/dataset-download-policy.md`: dataset size estimation, disk checks, subset strategy, download thresholds, and stop conditions.
- `references/codex-history-preservation.md`: raw conversation history requirements, forbidden pseudo-history summaries, final directory rules.
- `references/interaction-minimization-policy.md`: necessary vs unnecessary clarification, automatic default decisions, human-turn minimization.
- `references/paper-repo-reproduction.md`: repo scan, target identification, official entrypoint, wrapper policy.
- `references/gpu-and-checkpoint-handling.md`: GPU, CUDA, checkpoint, cache, and resource checks.
- `references/metric-extraction.md`: metric provenance, `result.md`, and `run_summary.json`.

## Environment Gate

Before installing dependencies or running repo code:

- infer Python version from docs, metadata, lockfiles, classifiers, Dockerfiles, CI, or import syntax;
- create an environment named with repo name and timestamp, for example `repo-20260519-153012`;
- record environment manager, environment name/path, Python version, torch/CUDA versions, and GPU availability;
- treat any existing environment as read-only context, not an execution target, unless the user explicitly says to reuse that exact environment.

See `references/isolated-environment-management.md`.

## Data and Training Gate

Before any dataset download or training:

- search for author checkpoints and official pretrained weights;
- download or locate pretrained checkpoints before considering full training;
- identify inference or evaluation scripts;
- identify a minimal valid evaluation subset or smoke mode;
- estimate download size and available disk;
- stop before large downloads or full training unless a training gate is satisfied.

Training is allowed only when:

- README or official instructions explicitly require training for reproduction;
- the user explicitly requests full training reproduction;
- no checkpoint exists and evaluation cannot be completed otherwise.

See `references/lightweight-reproduction-policy.md` and `references/dataset-download-policy.md`.

## Safety Rules

Never use:

- Docker;
- sudo;
- `rm -rf`;
- system Python overwrite;
- system CUDA modification;
- broad or violent cache deletion.

Always:

- use an isolated environment;
- operate inside a workspace;
- preserve logs and metric provenance;
- avoid modifying upstream source unless needed and allowed;
- keep checkpoints, datasets, logs, scripts, and temp files out of the final submission directory.

## Run Summary

Create `conversation_history/run_summary.json` with:

```json
{
  "repo_name": "",
  "success": false,
  "failure": null,
  "human_turns": 1,
  "repair_attempts": 0,
  "smoke_runs": 0,
  "final_metric": null,
  "metric_source": null,
  "smallest_success_model": "unknown",
  "environment_name": null,
  "dataset_downloaded": false,
  "checkpoint_used": false
}
```

Definitions:

- `success`: true only when `result.md` is generated from valid local benchmark/evaluation artifacts.
- `failure`: null on success, otherwise a short blocker category.
- `final_metric`: final submitted metric value or object.
- `metric_source`: file path plus key, column, or log line used for the final metric.
- `dataset_downloaded`: true when new dataset bytes were downloaded during the run.
- `checkpoint_used`: true when a local or downloaded checkpoint was used.

## Final Output Contract

The final submission directory must contain only:

```text
repo_name/
  result.md
  conversation_history/
```

Do not place logs, checkpoints, outputs, scripts, caches, raw datasets, or temp files in the final submission directory. Keep them in the work directory for traceability.
