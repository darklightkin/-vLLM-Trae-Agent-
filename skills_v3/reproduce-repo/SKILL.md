---
name: reproduce-repo
description: Use when the user provides a GitHub repository URL, local repository path, paper repository, benchmark repository, offline reproduction bundle, or asks to reproduce paper/table/benchmark results with low human interaction. This skill covers repo scanning, target metric identification, isolated conda or venv setup, dependency repair, dataset and checkpoint preparation, smoke testing, full evaluation, bounded failure recovery, metric provenance checks, fixed result.md generation, complete conversation_history preservation, and run_summary.json creation. Trigger this skill for failure modes including missing dependencies, broken Python or torch environments, CUDA OOM, checkpoint layout mismatch, dataset schema mismatch, missing metric files, official script interface drift, and ambiguous final submission results.
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
- asks for `result.md`, `conversation_history`, or a low-interaction reproduction;
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
result.md
conversation_history/
```

`conversation_history/` must include:

```text
history.md
run_summary.json
```

Raw logs, JSON, CSV, checkpoints, caches, and wrapper scripts can stay in the work directory for traceability, but do not include them in the final submission directory unless the official instructions require them.

## Low-interaction rules

- Initial prompt counts as one human turn.
- Agent scans, shell commands, progress updates, log analysis, and automatic repairs do not count as human turns.
- Ask the user only for credentials, private data permission, paid downloads, hardware/machine switch, forbidden source edit approval, or non-temporary deletion.
- Record every unavoidable user question as an `interaction_count_candidate`.
- Prefer safe defaults and document them in `history.md`.

## Execution workflow

Follow this order:

1. repo scan;
2. paper/table/benchmark target identification;
3. environment creation;
4. dependency installation;
5. dataset/checkpoint preparation;
6. smoke test;
7. full reproduction run;
8. failure diagnosis and repair;
9. metric extraction;
10. `result.md` generation;
11. `conversation_history` preservation;
12. `run_summary.json` creation.

Do not start with a full benchmark run. Pass the environment/resource/smoke gates first.

## Safety rules

- Do not write API keys, tokens, private credentials, or paid-access secrets into prompts, source files, logs, result tables, or history.
- Do not use Docker, sudo, or dangerous recursive deletion commands.
- Do not delete source code, configs, checkpoints, raw datasets, submission files, or conversation history.
- Do not modify upstream repository code unless the task explicitly allows it.
- If source edits are forbidden, use wrappers, environment variables, local config copies, or record a blocker.
- Do not fabricate datasets, checkpoints, metrics, or paper values.
- Do not present README or official table values as local reproduction results.

## Required references

Load as needed:

- `references/paper-repo-reproduction.md`: repo scan, target identification, official entrypoint, wrapper policy.
- `references/low-interaction-agent.md`: low-interaction rules, interaction counting, progress updates.
- `references/gpu-and-checkpoint-handling.md`: environment/resource gate, CUDA, checkpoint, dataset, HF cache, proxy.
- `references/metric-extraction.md`: metric provenance, `result.md`, `history.md`, `run_summary.json`.

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

## Final deliverables

`result.md`:

- fixed filename;
- Markdown table only;
- no notes, commands, explanations, screenshots, or provenance text.

`conversation_history/history.md`:

- task;
- skills and references used;
- environment;
- resources;
- commands;
- smoke runs;
- diagnostic runs;
- submission run;
- failure recovery table;
- metric provenance table;
- protocol deviations;
- files produced;
- interaction count.

`conversation_history/run_summary.json`:

```json
{
  "human_turns": 1,
  "repair_count": 0,
  "smoke_count": 0,
  "final_success": false,
  "smallest_success_model": null
}
```

Update the values from the actual run.
