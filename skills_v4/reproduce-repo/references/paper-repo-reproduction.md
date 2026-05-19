# Paper Repo Reproduction

## Responsibility

Scan the repository, identify the official reproduction target, choose the smallest valid run, and keep the process traceable.

## Repo Scan

Read before heavy execution:

- README and reproduction docs;
- requirements, environment files, and lockfiles;
- configs;
- train, eval, inference, benchmark, and test scripts;
- examples and shell scripts;
- paper/table references;
- releases, model cards, and issues when needed;
- local outputs, logs, and prior history files.

Build an inventory:

| Component | What to find |
| --- | --- |
| README | official instructions and target table |
| environment | Python, torch, CUDA, special packages |
| configs | model, benchmark, split, batch size |
| eval entrypoint | official command and arguments |
| dataset | dataset ID/path, split, schema, estimated size |
| checkpoint | source, files, size, fold layout |
| metrics | JSON/CSV/log outputs and metric names |
| submission | required final directory structure |

## Target Identification

Confirm:

- target benchmark;
- target method/model/checkpoint;
- target row and columns;
- target split/fold protocol;
- accepted tolerance;
- whether inference-only, one fold, all folds, or full benchmark is required.

If README and task card disagree, prefer explicit user/task-card submission requirements and record the discrepancy in work logs.

## Official Entrypoint

Prefer official commands in this order:

1. documented evaluation command;
2. benchmark script;
3. inference script plus official metric script;
4. test script that exercises the official metric;
5. wrapper around official functions.

Do not reimplement metrics unless official metric code is unavailable or broken beyond repair.

Do not choose a training entrypoint while a checkpoint/pretrained-weight evaluation entrypoint is available. Training is a gated fallback, not the default reproduction path.

## Wrapper Policy

Default: avoid upstream source edits.

A wrapper may:

- set paths and cache variables;
- adapt CLI arguments;
- select minimal subsets for smoke tests;
- call official model and metric functions;
- normalize checkpoint paths.

A wrapper must not:

- change labels;
- change splits;
- change metric formulas;
- change model architecture;
- silently skip failed samples for final metrics.

Record whether source was modified and why.
