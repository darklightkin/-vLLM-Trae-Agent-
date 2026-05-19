# GUI-KV v2 Low-Interaction Reproduction Prompt

You are a coding agent responsible for reproducing paper repositories. Use the local `skills` workflow to reproduce GUI-KV with as few human interactions as possible, while preserving a complete auditable record.

## Skills

Read the master skill first:

```text
/.codex/skills/reproduce-repo/SKILL.md
```

Then, following the master workflow stages and gates, read the corresponding sub-skill when needed:

```text
/.codex/skills/reproduce-repo/references/paper-repo-reproduction.md
/.codex/skills/reproduce-repo/references/low-interaction-agent.md
/.codex/skills/reproduce-repo/references/gpu-and-checkpoint-handling.md
/.codex/skills/reproduce-repo/references/metric-extraction.md
```

Do not copy the full skill text into your response. Load and apply the skills stage by stage.

## Skill Usage Guard

The master skill is responsible for stage routing. Before entering each major stage, read the corresponding sub-skill for that stage, then execute the stage actions.

Before each major stage, write one stage checkpoint in `conversation_history/run_log.md`, including the current stage, the skill file just read, and the next action.

If the context becomes long, a failed run requires retry, or you are uncertain about the next rule, re-read `/.codex/skills/reproduce-repo/SKILL.md` and the corresponding sub-skill for the current stage before continuing.

## Task Card

- GitHub: `https://github.com/SalesforceAIResearch/GUI-KV`
- Default local repository: `GUI-KV`
- Paper: `https://arxiv.org/pdf/2510.00536`
- Benchmark: `AgentNetBench`
- Target table: `Table 1`
- Target method: `GUI-KV`
- Target base model: `UI-TARS-1.5-7B`
- Target setting: pretrained/evaluation-only model, no training unless the repository explicitly requires it for evaluation
- Target budgets: `80%`, `10%`
- Target metric: `Step Accuracy`
- Accepted tolerance: within 15% relative difference from the target table values
- Reference table values for sanity check only:
  - `GUI-KV`, `UI-TARS-1.5-7B`, `AgentNetBench`, budget `80%`: `22.6`
  - `GUI-KV`, `UI-TARS-1.5-7B`, `AgentNetBench`, budget `10%`: `6.1`

Confirm the dataset, pretrained model/checkpoint, evaluation entrypoint, budget argument names, and output format from the official README, configs, scripts, and help output. Do not use any information that has not been confirmed by the current repository as execution evidence.

The reference values `22.6` and `6.1` come from the provided target table and may be used only as sanity checks. Do not present them as local reproduction results unless they are produced by the current run.

## Project Preferences

- All paths are relative to the current workspace by default.
- Do not hardcode machine-specific absolute paths in the prompt, history, or submission files.
- Prefer `hf-mirror.com` for downloading or caching HuggingFace datasets and models. If the mirror is unavailable, missing resources, or fails validation, fall back to the official HuggingFace endpoint.
- Do not fabricate datasets, checkpoints, screenshots, annotations, predictions, or metrics.
- Do not present README values, paper tables, prompt examples, or historical reference values as local reproduction results.
- Use the pretrained `UI-TARS-1.5-7B` model or the official checkpoint confirmed by the repository. Do not train or fine-tune a model by default.
- Download only the AgentNetBench resources required for the target evaluation settings unless the official protocol requires more.

## Source Edit Policy

Do not modify the original repository source code, configs, scripts, or README by default. If an official script has a compatibility issue, prefer a wrapper, command-line arguments, environment variables, a temporary copy, or a monkey-patch wrapper.

If modifying the original repository is required to continue, apply only the smallest compatibility patch, save the diff under `outputs/results/patches/`, and record the reason, changed files, and validation command in history. Never improve results by modifying metrics, dataset splits, benchmark annotations, prediction parsing, cache policy, budget logic, or evaluation protocols.

## Output

Write the final table to:

```text
submission/gui-kv/reproduction.md
```

Write the reproduction history to:

```text
submission/gui-kv/conversation_history/
```

If the tool supports exporting native conversation, trajectory, or chat history, save the native export under `submission/gui-kv/conversation_history/`. Use `history.md` for the native readable transcript when available. Put the agent-maintained execution log in `run_log.md`; do not use it as a substitute for the native conversation export.

`reproduction.md` must contain only a Markdown table, with no explanation:

```markdown
| Method | Base Model | Benchmark | Budget | Step Accuracy |
| --- | --- | --- | --- | --- |
| GUI-KV | UI-TARS-1.5-7B | AgentNetBench | 80% | <value> |
| GUI-KV | UI-TARS-1.5-7B | AgentNetBench | 10% | <value> |
```

Metrics must come from stdout, logs, JSON, CSV, or result files produced by this real run. Put sources, commands, errors, fixes, resource paths, dataset/checkpoint provenance, budget settings, and the human-interaction count in `conversation_history/run_log.md`.
