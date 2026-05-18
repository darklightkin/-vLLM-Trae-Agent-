# low-interaction-agent

## Goal

Minimize human-to-agent interaction rounds while keeping the reproduction process correct and auditable.

## Interaction Accounting

- The initial task prompt counts as one user interaction.
- User-provided corrections, path fixes, GPU changes, and manual decisions count as additional interactions.
- Agent tool calls, automatic exploration, log analysis, and self-repair do not count.
- Agent clarification questions should be avoided unless no safe default exists.

## Behavior Rules

1. Start by scanning local files and official docs in the repo.
2. Make safe defaults instead of asking obvious questions.
3. Use local paths discovered from the workspace before asking for paths.
4. Try a bounded repair loop after failures.
5. Stop only when the next action requires unavailable data, credentials, hardware, or user-owned policy decisions.

## Safe Defaults

- Use a new isolated environment named after the repo.
- Use CPU only for smoke tests; use CUDA for target evaluation if available.
- Use pretrained checkpoints when the README provides them.
- Run one target configuration before a full sweep.
- Save outputs under `outputs/`, `results/`, or a repo-local run directory.

## Failure Loop

For each failure:

1. Read the full traceback or log.
2. Classify the failure: dependency, config, path, dataset, checkpoint, GPU memory, network, or code mismatch.
3. Apply the smallest repair.
4. Re-run the smallest command that reaches the failed point.
5. After repeated failures, summarize the blocker and exact next action.
