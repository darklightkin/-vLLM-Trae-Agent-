# MuQ-Eval v2 Low-Interaction Reproduction Prompt

You are a coding agent responsible for reproducing paper repositories. Use the local `skills` workflow to reproduce MuQ-Eval with as few human interactions as possible, while preserving a complete auditable record.

## Skills

Read the master skill first:

```text
/.codex/skills/reproduce-repo
```

Then, following the master workflow stages and gates, read the corresponding sub-skill when needed:

```text
/.codex/skills/reproduce-repo/references/paper-repo-reproduction.md
/.codex/skills/reproduce-repo/low-interaction-agent.md
/.codex/skills/reproduce-repo/gpu-and-checkpoint-handling.md
/.codex/skills/reproduce-repo/metric-extraction.md
```

Do not copy the full skill text into your response. Load and apply the skills stage by stage.

## Skill Usage Guard

The master skill is responsible for stage routing. Before entering each major stage, read the corresponding sub-skill for that stage, then execute the stage actions.

Before each major stage, write one stage checkpoint in history, including the current stage, the skill file just read, and the next action.

If the context becomes long, a failed run requires retry, or you are uncertain about the next rule, re-read `skills/reproduction-master.md` and the corresponding sub-skill for the current stage before continuing.

## Task Card

- GitHub: `https://github.com/dgtql/MuQ-Eval`
- Default local repository: `MuQ-Eval`
- Paper: `https://arxiv.org/abs/2603.22677`
- Benchmark: `MusicEval`
- Target method: `A1 (Frozen+MSE) [recommended]`
- Target metrics: `System SRCC`, `Utterance SRCC`

Confirm the dataset, checkpoint, encoder/model, evaluation entrypoint, and output format from the official README, configs, scripts, and help output. Do not use any information that has not been confirmed by the current repository as execution evidence.

## Project Preferences

- All paths are relative to the current workspace by default.
- Do not hardcode machine-specific absolute paths in the prompt, history, or submission files.
- Prefer `hf-mirror.com` for downloading or caching HuggingFace datasets and models. If the mirror is unavailable, missing resources, or fails validation, fall back to the official HuggingFace endpoint.
- Do not fabricate datasets, checkpoints, or metrics.
- Do not present README values, paper tables, prompt examples, or historical reference values as local reproduction results.

## Source Edit Policy

Do not modify the original repository source code, configs, scripts, or README by default. If an official script has a compatibility issue, prefer a wrapper, command-line arguments, environment variables, a temporary copy, or a monkey-patch wrapper.

If modifying the original repository is required to continue, apply only the smallest compatibility patch, save the diff under `outputs/results/patches/`, and record the reason, changed files, and validation command in history. Never improve results by modifying metrics, dataset splits, or evaluation protocols.

## Output

Write the final table to:

```text
submission/MuQ-Eval/reproduction.md
```

Write the reproduction history to:

```text
submission/MuQ-Eval/conversation_history/history.md
```

If the tool supports exporting native conversation, trajectory, or chat history, save it under `submission/MuQ-Eval/conversation_history/` as well. `history.md` is only an index and execution record; it does not replace the native export.

`reproduction.md` must contain only a Markdown table, with no explanation:

```markdown
| Method | System SRCC | Utterance SRCC |
| --- | --- | --- |
| A1 (Frozen+MSE) [recommended] | <value> | <value> |
```

Metrics must come from stdout, logs, JSON, CSV, or result files produced by this real run. Put sources, commands, errors, fixes, resource paths, and the human-interaction count in history.