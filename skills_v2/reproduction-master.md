# reproduction-master v2

## Goal

Act as the master skill for paper-repository reproduction. Split the task into auditable stages and route to sub-skills when needed. The core goals are low interaction, traceability, no fabrication, and no unintended pollution of the original repository.

## Master/Sub-Skill Structure

The master skill owns stages, gates, routing, and final acceptance. Details are delegated to sub-skills:

- `paper-repo-reproduction.md`: repository scanning, official entrypoint discovery, and reproduction workflow.
- `low-interaction-agent.md`: low-interaction strategy, progress updates, and stop conditions.
- `gpu-and-checkpoint-handling.md`: GPU, CUDA, checkpoints, HuggingFace cache, and dataset download.
- `metric-extraction.md`: metric extraction, table generation, and result validation.

Read this file first, then read the corresponding sub-skill at each stage. Do not expand all skills into the prompt at once.

## Stages

### 1. Intake

Assume the prompt provides a GitHub URL. Treat the GitHub URL as the primary task entrypoint. A local path is only an optional override.

Confirm that the task card contains at least:

- GitHub URL;
- target benchmark;
- target model or method;
- target metrics;
- submission format;
- whether source-code edits in the original repository are allowed.

If no local path is provided, infer it from the GitHub repository name:

```text
<workspace>/model/<repo-name>
```

Example:

```text
GitHub: https://github.com/dgtql/MuQ-Eval
Local repo: D:\vLLM1\model\MuQ-Eval
```

If benchmark, dataset, checkpoint, or metrics are missing, clone/read README, paper, configs, and scripts first. Ask the user only when the missing field cannot be inferred safely.

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

### 6. Target Run

Prefer the official command recommended by the README. Run only the requested model, benchmark, and setting unless the assignment explicitly requires a full sweep.

If full reproduction is blocked by hardware, data, checkpoint, or permissions, run the smallest valid substitute and clearly record the blocker and missing items in history.

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
- whether original repository code was modified;
- final metrics and raw result paths;
- estimated human interaction count.

## Gates

Before moving to the next stage:

- After repo bootstrap: know the official eval entrypoint.
- After environment gate: know whether CPU or CUDA will be used.
- After resource gate: know whether dataset and checkpoints are complete.
- After smoke gate: have at least one minimal command passing.
- After target run: metrics are traceable to raw files.

## Safety

- Do not modify original repository code by default. If the task allows edits, record the diff and reason.
- Do not delete source code, configs, checkpoints, raw data, submission files, or conversation history.
- You may automatically clean obvious temporary failed-cache files inside the workspace, such as `*.tmp`, `*.incomplete`, and `__pycache__/`.
- Do not write API keys, HF tokens, or private credentials into prompts, code, tables, or history.

## Output

Each repo must produce at least:

```text
submission/<repo>/reproduction.md
submission/<repo>/conversation_history/history.md
```

`reproduction.md` contains only the table. Process, commands, errors, and provenance go into history.
