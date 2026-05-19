# Interaction Minimization Policy

## Responsibility

Minimize human turns while preserving safety, correctness, and benchmark validity.

## Human Turn Definition

Count the initial user task as `1`. Count each additional required user decision as another human turn.

Do not count:

- agent progress updates;
- shell commands;
- automatic repo scans;
- automatic dependency repairs;
- log analysis;
- smoke runs;
- metric extraction.

Record the final count in `run_summary.json.human_turns`.

## Automatic Exploration

Before asking the user, inspect:

- README and docs;
- install files;
- configs and examples;
- train/eval/inference scripts;
- shell scripts;
- paper links and benchmark tables;
- releases and model cards;
- local history files;
- existing outputs and logs;
- issues when network/repo access is available.

## Necessary Clarification

Ask only when blocked by:

- credentials, API keys, private dataset access, or paid/license decisions;
- a machine/GPU/server change;
- permission to modify upstream source when policy or task forbids it;
- deletion of non-temporary files;
- multiple final submission targets that cannot be resolved from task, README, or official benchmark instructions;
- large downloads or long training beyond policy thresholds.

## Non-Necessary Clarification

Do not ask about:

- obvious repo name;
- default Python version when inferable;
- whether to create an environment;
- whether to reuse an existing environment;
- whether to run smoke tests;
- whether to use author checkpoints;
- whether to use the smallest valid evaluation path;
- routine dependency repair inside the isolated environment.

Choose the safest benchmark-preserving default and record the decision.

The default environment decision is always to create a fresh isolated environment. Low interaction must never be interpreted as permission to reuse `base`, the currently active shell environment, or an old repo environment.

## Default Decisions

When choices are ambiguous but safe, prefer:

- official checkpoint over training;
- checkpoint/pretrained-weight download before dataset-heavy work;
- official eval script over custom metric code;
- validation/dev split for smoke if final split is unavailable;
- smaller batch size for OOM without changing metrics;
- wrapper scripts over upstream source edits;
- local/offline resources over downloads;
- conda over venv.

## Reporting

Keep progress updates short and stage-labeled:

```text
[scan] ...
[env] ...
[data] ...
[model] ...
[smoke] ...
[eval] ...
[repair] ...
[metrics] ...
[submit] ...
```

Do not stop after reporting a recoverable problem. Repair within policy and continue.
