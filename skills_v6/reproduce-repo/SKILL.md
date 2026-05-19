---
name: reproduce-repo
description: Use when the user provides a GitHub repository URL, local repository path, paper repository, benchmark repository, offline reproduction bundle, or asks to reproduce paper/table/benchmark results with low human interaction. This skill covers repo scanning, target metric identification, fresh isolated conda or venv setup, dependency repair, checkpoint-first evaluation or inference reproduction, minimal dataset handling, stage-by-stage verification against repository-specified versions, smoke testing, bounded failure recovery, metric provenance checks, fixed result.md generation, raw Codex history.md preservation, and standalone change_summary.md documentation of agent edits and verification comparisons. Trigger this skill for failure modes including missing dependencies, broken Python or torch environments, CUDA OOM, checkpoint layout mismatch, dataset schema mismatch, missing metric files, official script interface drift, and ambiguous final submission results.
---

# Reproduce Repo

## Purpose

Reproduce a target paper, table, or benchmark result from a repository with minimal human interaction and complete auditability.

This is the master skill. Keep it as the triggerable entrypoint. Load the reference files only when their stage is needed.

## When to use

Use after the YAML `description` triggers when the user:

- provides a GitHub URL or local repository path;
- points to an offline paper-repo bundle;
- asks to reproduce a benchmark, table, result, model row, or README metric;
- asks for `result.md`, raw `history.md`, or a low-interaction reproduction;
- pastes logs from a failed reproduction and wants the agent to continue.

Do not use this skill for unrelated code review or ordinary feature development unless it is part of a reproduction workflow.

## Inputs

Expected inputs:

- GitHub URL or local repo path;
- paper URL or target table/README row;
- benchmark name;
- model/method/checkpoint target;
- dataset target or offline resource path;
- target metrics and accepted tolerance;
- hardware constraints;
- whether source-code edits are allowed;
- official submission structure.

If inputs are missing, infer from README, configs, scripts, paper links, model cards, issues, local resources, and task cards before asking the user.

## Outputs

Final submission directory must contain only:

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

`result.md` contains only the metric table. `history.md` contains only the raw Codex conversation trace: user messages, assistant replies, tool calls, command execution records, errors, repairs, and metric extraction process. `change_summary.md` contains a concise standalone summary of agent-made changes and every stage verification/comparison against repository-specified versions, configs, datasets, checkpoints, scripts, and model names. Raw logs, JSON, CSV, checkpoints, caches, outputs, and wrapper scripts can stay in the work directory for traceability, but do not include them in the final submission directory unless the user explicitly requires them.

## Low-interaction rules

- User interaction turns are counted by messages the user actively sends to Codex.
- Agent scans, shell commands, progress updates, log analysis, and automatic repairs do not count as human turns.
- Ask the user only for credentials, private data permission, paid downloads, large dataset downloads, long training runs, hardware/machine switch, system-level modifications, dangerous deletion, forbidden source edit approval, or non-temporary deletion.
- Record every unavoidable user question as an `interaction_count_candidate`.
- Prefer safe defaults, act automatically when inferable, and let the raw `history.md` trace show the decision process.

## Execution workflow

Follow this order:

1. repo scan;
2. verify repository-declared versions, required files, official scripts, model names, and target instructions;
3. paper/table/benchmark target identification;
4. verify target row, metric names, split/fold protocol, and accepted tolerance against official repo/paper/task-card requirements;
5. fresh isolated environment creation;
6. dependency installation;
7. verify Python, CUDA, PyTorch, package versions, and command entrypoints against repository requirements;
8. author checkpoint or pretrained weight discovery/download;
9. verify checkpoint source, filename, size/hash, fold layout, and model name against repository requirements;
10. dataset discovery/download or local dataset binding;
11. verify dataset ID/path, version, split, schema, sample count, and preprocessing against repository requirements;
12. checkpoint-based evaluation or inference reproduction planning;
13. minimal evaluation subset or smoke test;
14. verify smoke output, metric keys, and expected artifact locations;
15. benchmark evaluation if the gates pass;
16. verify final evaluation command, config, checkpoint, dataset, split/fold, model name, and metrics against repository requirements;
17. training only if no checkpoint exists, evaluation cannot be completed, and training is required by the task or official instructions;
18. failure diagnosis and repair;
19. verify every agent-made edit or wrapper against official semantics and record it in `change_summary.md`;
20. metric extraction;
21. verify metric provenance and local-execution status;
22. `result.md` generation;
23. raw Codex `history.md` preservation;
24. `change_summary.md` generation.

Do not start with a full benchmark run, full dataset download, or full training run. Pass the fresh-environment, checkpoint, dataset-size, disk-space, and smoke gates first.

## Stage verification policy

Every major stage must end with a `Verify` step before moving forward. Verification compares the local state against the repository, paper, task card, config files, README, model card, or official scripts.

Verify after at least:

- repository scan and target identification;
- environment creation and dependency installation;
- dataset discovery, download, binding, schema inspection, and preprocessing;
- checkpoint or pretrained weight discovery/download;
- smoke test;
- benchmark evaluation;
- metric extraction;
- every agent-made source edit, wrapper, config copy, path adaptation, or command-line change;
- final packaging.

Verification must record:

- expected version or value from official materials;
- actual local version or value;
- match status: `match`, `acceptable-drift`, `mismatch`, or `unknown`;
- source of expectation, such as README line, config key, requirements file, paper table, model card, or task card;
- action taken for mismatch.

Do not rename model names, method names, benchmark names, metric names, dataset split names, or paper row labels. If an official name is ambiguous, preserve the source spelling in `result.md` and record the ambiguity in `change_summary.md`.

## Environment policy

- Default: create a new isolated environment for every repo.
- Prefer `conda create`; if conda is unavailable, create a new venv.
- Environment names should include the repo name and a timestamp.
- Do not default to reusing any existing conda/venv environment.
- Do not use `base` or system Python for reproduction.
- Do not automatically fallback to an old environment after dependency failure.
- Reuse is allowed only when the user explicitly requests reuse of an existing environment.

## Download and training policy

- First look for author-provided pretrained checkpoints or model weights.
- If pretrained checkpoints are available, prioritize checkpoint-based evaluation or inference reproduction over training.
- Prefer the smallest valid evaluation subset or smoke test.
- Do not default to downloading complete training datasets.
- Do not start full or long training by default.
- If dataset download is necessary, estimate dataset size, check disk space, prefer the smallest necessary subset, and leave the reason visible in logs or the raw history trace.
- Training is allowed only when no usable checkpoint exists, evaluation cannot be completed without training, and the task or official instructions require training.

## Safety rules

- Do not write API keys, tokens, private credentials, or paid-access secrets into prompts, source files, logs, result tables, or history.
- Do not use Docker, sudo, or dangerous recursive deletion commands.
- Do not delete source code, configs, checkpoints, raw datasets, submission files, or conversation history.
- Do not modify upstream repository code unless the task explicitly allows it.
- If source edits are forbidden, use wrappers, environment variables, local config copies, or record a blocker.
- Do not fabricate datasets, checkpoints, metrics, or paper values.
- Do not present README or official table values as local reproduction results.
- Do not overwrite system Python, modify system CUDA, or rely on `base`.

## Required references

Load as needed:

- `references/paper-repo-reproduction.md`: repo scan, target identification, official entrypoint, wrapper policy.
- `references/low-interaction-agent.md`: low-interaction rules, interaction counting, progress updates.
- `references/environment-and-resource.md`: fresh environment/resource gate, CUDA, checkpoint, dataset, HF cache, proxy.
- `references/metric-extraction.md`: metric provenance, `result.md`, raw `history.md`, final submission layout.

## Metric provenance gate

Every final metric must record:

- metric name;
- numeric value;
- source file;
- source key, CSV column, or log line;
- run type: `submission`, `smoke`, `diagnostic`, `official_table`, or `blocked`;
- split: `test`, `validation`, `train`, `all`, `fold<N>`, `5-fold`, or `unknown`;
- whether the value is from local execution or an official reference.

Only `submission` local-execution metrics may populate `result.md`. Smoke, diagnostic, and official-table-only values cannot be submitted as reproduced results.

## Change summary

Generate `change_summary.md` separately from `history.md`.

It must include:

- agent-made file edits, wrappers, local config copies, command changes, environment changes, and path adaptations;
- why each change was needed;
- whether each change altered benchmark semantics;
- before/after comparison against official repository requirements;
- every stage `Verify` result;
- unresolved mismatches or accepted drifts.

It must not replace `history.md`, summarize the raw conversation trace, fabricate missing tool calls, or rename official model/method/dataset/metric names.

## Final deliverables

`result.md`:

- fixed filename;
- Markdown table only;
- no notes, commands, explanations, screenshots, or provenance text.

`history.md`:

- fixed filename at `repo_name/history.md`;
- complete raw Codex conversation trace;
- includes user messages, assistant replies, tool calls, command records, errors, repair attempts, and metric extraction process;
- no rewritten report, no summary substitute, no compressed or beautified transcript, no assistant-only transcript, no fabricated conversation history.

`change_summary.md`:

- fixed filename at `repo_name/change_summary.md`;
- standalone summary of agent-made changes and stage-by-stage verification comparisons;
- includes expected vs actual versions, configs, datasets, checkpoints, scripts, model names, and metric keys;
- records mismatch handling and whether benchmark semantics changed;
- no raw logs, no full conversation transcript, no renamed official model names.
