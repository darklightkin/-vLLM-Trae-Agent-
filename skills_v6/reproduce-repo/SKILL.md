---
name: reproduce-repo
description: Use when the user provides a GitHub repository URL, local repository path, paper repository, benchmark repository, offline reproduction bundle, or asks to reproduce paper/table/benchmark results with low human interaction. Covers repo scanning, fresh isolated environment setup, checkpoint-first evaluation, minimal dataset handling, smoke testing, bounded repair, metric provenance, result.md generation, raw history.md preservation, and change_summary.md.
---

# Reproduce Repo

## Purpose

Reproduce a target paper, table, or benchmark result from a repository with minimal human interaction and complete auditability.

This file is the entrypoint and dispatcher. Load reference files only when their stage is needed.

## Trigger

Use this skill when the task includes a repo URL/path, paper benchmark, target table/row, pretrained model, reproduction failure log, or required files such as `result.md` and `history.md`.

## Required Inputs

Infer missing values from README, configs, scripts, paper, model cards, task card, local files, and logs before asking the user.

Expected inputs:

- repo URL or local repo path;
- target benchmark/table/row/model/method;
- target dataset, checkpoint, split/fold, metric, and tolerance;
- hardware constraints and submission layout;
- whether upstream source edits are allowed.

## Final Outputs

Final submission directory:

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

- `result.md`: Markdown table only, from valid local submission metrics.
- `history.md`: raw Codex conversation trace only. Do not summarize or fabricate it.
- `change_summary.md`: concise summary of agent edits, environment deviations, wrappers, command changes, and verification comparisons.

Keep raw logs, caches, checkpoints, datasets, scripts, and JSON/CSV outputs outside the final submission directory unless the user explicitly requires them.

## Core Rules

- Create a fresh isolated conda/venv by default. Do not use `base`, system Python, or an old env unless the user explicitly asks.
- Prefer author checkpoints or pretrained weights. Do not train by default.
- Do not download full training datasets by default. Prefer local/offline resources and the smallest valid eval subset.
- Do not start full evaluation before repo scan, environment, resource, and smoke gates pass.
- Do not modify upstream code unless explicitly allowed. Prefer wrappers, env vars, local config copies, and command-line args.
- Do not fabricate datasets, checkpoints, metrics, history, or official values.
- Do not present README/paper numbers as reproduced results.
- Do not write API keys or private tokens into prompts, files, logs, result tables, or history.
- Do not delete source, configs, scripts, checkpoints, datasets, submission files, or conversation history without approval.

## Workflow

Proceed through these gates:

1. `SCAN`: read README, requirements, configs, scripts, paper/task card, local resources.
2. `TARGET`: lock target row, model/method, dataset, split/fold, metrics, tolerance.
3. `ENV`: create fresh env, install PyTorch/CUDA first, then remaining deps.
4. `RESOURCE`: bind/download checkpoint and dataset with size/path/schema checks.
5. `SMOKE`: run import/help/model/data/minimal metric checks.
6. `EVAL`: run target submission evaluation only after gates pass.
7. `REPAIR`: bounded fixes for dependency, CUDA, path, schema, or interface errors.
8. `METRIC`: extract only local submission-run metrics with provenance.
9. `SUBMIT`: write `result.md`, preserve raw `history.md`, write `change_summary.md`.

Each gate must end with a short verification:

```text
Expected | Actual | Status(match/acceptable-drift/mismatch/unknown) | Source | Action
```

Record verification results in `change_summary.md` or working notes.

## Low Interaction

Ask only for credentials, private data permission, paid access, large/unknown-size downloads, long/full training, hardware switch, system-level changes, dangerous deletion, or forbidden source edits.

Everything else should be inferred, attempted safely, and recorded.

## References

Load only as needed:

- `references/paper-repo-reproduction.md`: repo scan, target lock, official entrypoint, wrapper policy.
- `references/environment-and-resource.md`: fresh env, CUDA/PyTorch, mirrors/fallbacks, datasets, checkpoints.
- `references/low-interaction-agent.md`: when to ask, progress updates, interaction counting.
- `references/metric-extraction.md`: metric provenance, `result.md`, raw `history.md`, `change_summary.md`.

## Metric Gate

Every final metric must have:

- exact metric name;
- numeric value;
- source file and key/line/column;
- run type: `submission`, `smoke`, `diagnostic`, `official_table`, or `blocked`;
- split/fold;
- local execution status.

Only `submission` metrics from local execution can populate `result.md`.

## Stop Conditions

Stop and report instead of guessing when:

- required private/gated resources need user action;
- no valid checkpoint/eval path exists and training is not authorized;
- the official protocol is ambiguous and cannot be resolved from repo/paper/task card;
- repeated repairs would change benchmark semantics;
- raw history export is impossible.
