# reproduction-master v2.1

## Goal

Act as the master skill for paper-repository reproduction. Split the task into auditable stages and route to sub-skills when needed. The core goals are low interaction, traceability, no fabrication, and no unintended pollution of the original repository.

## Master/Sub-Skill Structure

The master skill owns stages, gates, routing, and final acceptance. Details are delegated to sub-skills:

- `paper-repo-reproduction.md`: repository scanning, official entrypoint discovery, and reproduction workflow.
- `low-interaction-agent.md`: low-interaction strategy, progress updates, and stop conditions.
- `gpu-and-checkpoint-handling.md`: GPU, CUDA, checkpoints, HuggingFace cache, and dataset download.
- `metric-extraction.md`: metric extraction, table generation, and result validation.

Read this file first, then read the corresponding sub-skill at each stage. Do not expand all skills into the prompt at once.

## Lessons from history_v1

The MuQ-Eval v1 run showed that the skills can guide a successful low-interaction reproduction, but the pack must enforce stronger submission discipline:

- raw JSONL logs are not enough; always produce a clean `conversation_history/history.md`;
- failures must be summarized in a structured recovery table;
- the final table filename must come from the task card, not from a hardcoded default;
- smoke, diagnostic, and submission runs must be clearly separated;
- protocol deviations, such as single-fold or test-split-only reproduction, must be explicit;
- wrapper scripts are acceptable when original repo edits are forbidden, but the history must state why the wrapper exists.

## Stages

### 1. Intake

Assume the prompt provides a GitHub URL. Treat the GitHub URL as the primary task entrypoint. A local path is only an optional override.

Confirm that the task card contains at least:

- GitHub URL;
- target benchmark;
- target model or method;
- target metrics;
- submission format;
- final table filename, such as `result.md`, `reproduction.md`, or a task-specific name;
- whether source-code edits in the original repository are allowed.

If no local path is provided, infer it from the GitHub repository name:


If benchmark, dataset, checkpoint, or metrics are missing, clone/read README, paper, configs, and scripts first. Ask the user only when the missing field cannot be inferred safely.

If the final filename is missing, use the assignment or task card convention. Do not silently switch between `result.md`, `reproduction.md`, and `reproduction-result.md`.

### 2. Repo Bootstrap

Prepare the repository from the GitHub URL:

1. Parse the repo name and infer the default local directory.
2. If the local directory does not exist, clone the GitHub repository.
3. If it exists, check `git status --short` and `git remote -v`.
4. If the existing remote does not match the prompt GitHub URL, do not overwrite; stop and report the path conflict.
5. If the remote matches, keep using the existing directory and do not delete user changes.

Recommended command shape:

```powershell
cd <workspace>
New-Item -ItemType Directory -Force model | Out-Null
git clone <github-url> model\<repo-name>
cd model\<repo-name>
```

If the repo already exists:

```powershell
cd <workspace>\model\<repo-name>
git status --short
git remote -v
```

Produce a minimal inventory:

| Component | Path / Source | Status |
| --- | --- | --- |
| README |  |  |
| requirements |  |  |
| configs |  |  |
| eval entrypoint |  |  |
| dataset |  |  |
| checkpoint |  |  |
| result output |  |  |

### 3. Environment Gate

Run environment checks before expensive jobs:

- Python/conda/venv availability;
- PyTorch and CUDA availability;
- `nvidia-smi` availability;
- free disk space;
- proxy and HuggingFace cache sanity;
- import checks for critical packages.

If the environment is not ready, report the smallest repair action first. Do not start a long evaluation immediately.

### 4. Resource Gate

Confirm that dataset and checkpoints come from the official README, paper, release, or model repository. Prefer HuggingFace official APIs and repo-local cache.

Record:

- dataset ID, split, and cache path;
- checkpoint repo/file and local path;
- encoder/backbone model ID;
- whether a token is required;
- whether only part of the fold structure is available.

### 5. Smoke Gate

Run the smallest verifiable command before target reproduction. A smoke run can be:

- import test;
- dataset `train[:1]` or minimal split load;
- checkpoint load;
- single-batch forward;
- single-fold or single-config evaluation.

If smoke fails, enter the bounded repair loop. Do not jump to full evaluation.

Always label smoke outputs as `smoke` in file names or history. Smoke metrics must not be submitted as final metrics.

### 6. Target Run

Prefer the official command recommended by the README. Run only the requested model, benchmark, and setting unless the assignment explicitly requires a full sweep.

If full reproduction is blocked by hardware, data, checkpoint, or permissions, run the smallest valid substitute and clearly record the blocker and missing items in history.

Classify every run as one of:

| Run type | Meaning | Can be submitted? |
| --- | --- | --- |
| `smoke` | minimal import/model/data/forward/metric check | No |
| `diagnostic` | helper run to inspect split coverage, system IDs, cache, or schema | No |
| `submission` | target benchmark/model/setting chosen for final table | Yes |

Never mix cells from different run types in the final table.

### 7. Metric Extraction

Extract metrics from real logs, JSON, CSV, or stdout. The final submission table must contain only the requested table and no process explanation.

Never present README reference values as local reproduction results.

### 8. History and Acceptance

Generate conversation history with:

- initial prompt and task card;
- skill files read;
- actual commands and key outputs;
- dataset/checkpoint sources;
- errors, fixes, and reruns;
- a failure recovery table with `failure`, `cause`, `action`, and `result`;
- whether original repository code was modified;
- whether wrapper scripts were added and why;
- protocol deviations and limitations;
- final metrics and raw result paths;
- estimated human interaction count.

The clean history file must be human-readable Markdown. Keep raw JSONL or terminal logs as internal evidence, not as the only conversation history artifact.

## Gates

Before moving to the next stage:

- After repo bootstrap: know the official eval entrypoint.
- After environment gate: know whether CPU or CUDA will be used.
- After resource gate: know whether dataset and checkpoints are complete.
- After smoke gate: have at least one minimal command passing.
- After target run: metrics are traceable to raw files.
- After history gate: clean `conversation_history/history.md` exists and includes failure recovery, interaction count, raw output paths, and protocol deviations.
- After submission gate: final table filename and directory match the task card.

## Safety

- Do not modify original repository code by default. If the task allows edits, record the diff and reason.
- Do not delete source code, configs, checkpoints, raw data, submission files, or conversation history.
- You may automatically clean obvious temporary failed-cache files inside the workspace, such as `*.tmp`, `*.incomplete`, and `__pycache__/`.
- Do not write API keys, HF tokens, or private credentials into prompts, code, tables, or history.

## Output

Each repo must produce at least the task-card requested table and clean history:

```text
submission/<repo>/<final-table-name>
submission/<repo>/conversation_history/history.md
```

The final table file contains only the table. Process, commands, errors, protocol deviations, and provenance go into history.

## Required history.md template

Use this structure for the cleaned history:

```markdown
# <repo> Reproduction History

## Task
## Skills Used
## Environment
## Resources
## Commands
## Smoke Runs
## Diagnostic Runs
## Submission Run
## Failure Recovery
| Failure | Cause | Action | Result |
| --- | --- | --- | --- |
## Metrics
## Protocol Deviations
## Files Produced
## Interaction Count
```
