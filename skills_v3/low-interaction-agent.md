# low-interaction-agent v2.1

## Responsibility

Minimize human interaction rounds while keeping the reproduction correct and auditable. This skill does not replace the reproduction workflow; it defines when the agent should decide automatically and when it must stop to ask the user.

## Interaction Counting

- The initial prompt counts as one user interaction.
- User-provided path fixes, GPU changes, credential decisions, and source-edit decisions count as additional interactions.
- Agent scans, tool calls, log analysis, self-repair, and progress updates do not count.
- Every unavoidable question to the user must be recorded as an `interaction count candidate` in the clean history.

## Default Behavior

Do these automatically first:

- read README, configs, scripts, and task card;
- inspect Python, conda/venv, CUDA, GPU, disk, and proxy;
- inspect local data, checkpoints, and cache;
- run the smallest smoke test;
- analyze errors and run a bounded repair loop.

Do not ask the user merely because the answer could be clearer. Ask only when there is no safe default.

## Must Ask The User When

- a token, account, or private data permission is required;
- a machine, GPU, or remote server must be changed;
- a non-temporary file must be deleted;
- source-code edits are required but forbidden by the prompt;
- multiple results could be submitted and the task requirements cannot disambiguate them.

## Do Not Ask The User When

Do not ask for information that can be inferred from:

- README, paper, model cards, issues, configs, or scripts;
- `git remote -v`, local directory names, and task card fields;
- environment probes such as Python, CUDA, GPU, disk, proxy, and cache;
- local checkpoint and dataset file inspection;
- raw logs or traceback text.

If multiple safe options exist, choose the lowest-cost option that preserves the benchmark semantics and record the choice in history.

## Progress Updates

For long tasks, emit status updates:

```text
[stage] current action; reason; next step.
```

Recommended stages:

- `[scan]` repository, README, configs, scripts;
- `[env]` Python, CUDA, GPU, disk, proxy;
- `[deps]` package import or installation;
- `[model]` checkpoint and encoder;
- `[data]` dataset, split, cache;
- `[smoke]` smoke or minimal run;
- `[eval]` target benchmark;
- `[repair]` error classification and smallest repair;
- `[metrics]` parse JSON/log/csv;
- `[submit]` write table and history.

## Bounded Repair Loop

For each failure:

1. Read the full traceback or log.
2. Classify the failure: dependency, config, path, data, checkpoint, memory, network, code interface, or permission.
3. Apply the smallest repair or workaround.
4. Re-run the smallest command that reaches the failure point.
5. After two repeated failures of the same class, stop and write a clear blocker.

For every repair attempt, append a structured entry to the future history table:

| Field | Meaning |
| --- | --- |
| Failure | exact symptom or command failure |
| Cause | evidence-backed cause or hypothesis |
| Action | one change made in this attempt |
| Result | pass/fail and next step |

Do not change multiple variables in one repair attempt unless they are inseparable.

## Safe Defaults

- Use an isolated environment; do not pollute system Python.
- Prefer official scripts and author checkpoints.
- Run target heavy jobs only when CUDA is available; use CPU for smoke or single-fold runs.
- Save outputs under repo-local `outputs/results` or workspace `outputs/<repo>`.
- Put actual choices in history, not in the final result table.
- Prefer wrapper scripts over original source edits when the task forbids modifying the upstream repository.
- If a wrapper is added, record `Original repo modified: No` and `Wrapper script added: Yes` in history.
