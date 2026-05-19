# Low Interaction Reference

## Responsibility

Reduce human turns while preserving correctness and auditability.

## Interaction counting

- Count human turns by messages the user actively sends to Codex.
- User path fixes, GPU changes, credentials, private data decisions, large-download approvals, long-training approvals, deletion approvals, system-level modification approvals, and source-edit approvals count as extra turns.
- Agent scans, shell commands, progress updates, automatic repairs, and log analysis do not count.
- Every unavoidable question must be recorded as an `interaction_count_candidate`.

## Do not ask when inferable

Do not ask the user for information that can be inferred from:

- README, configs, scripts, paper, model card, or issue text;
- local directory names and `git remote -v`;
- offline resource README files;
- environment probes;
- dataset/checkpoint inspection;
- logs and tracebacks.

Choose the safest benchmark-preserving default and record it.

Default environment decision: create a new isolated environment. Do not ask whether to reuse an environment, and do not silently reuse `base`, the current shell environment, a previous conda env, or an old venv. Reuse is allowed only when the user explicitly requests reuse.

Default reproduction decision: look for author checkpoints or pretrained weights first, then run checkpoint-based evaluation/inference with the smallest valid evaluation subset or smoke test. Do not ask whether to start full training unless the training gate is met.

Default verification decision: after each major stage, compare local state against repository-specified versions, configs, model names, dataset versions, checkpoint identities, script entrypoints, split/fold protocols, and metric keys. Do not ask the user to verify information that can be checked from repository files, papers, task cards, model cards, logs, or local command outputs.

## Must ask when blocked

Ask only for:

- API key, account, token, or private dataset permission;
- paid download or license decision;
- large dataset download or unknown-size dataset download;
- long training or full training run;
- machine/GPU/server switch;
- system-level modification;
- deletion of non-temporary files;
- modifying upstream code when forbidden;
- ambiguous final submission choice that cannot be resolved from task requirements.

## Progress updates

Use short stage updates:

```text
[scan] current action; reason; next step.
[verify] current comparison; expected vs actual; next action.
[env] current action; reason; next step.
[data] current action; reason; next step.
[model] current action; reason; next step.
[smoke] current action; reason; next step.
[eval] current action; reason; next step.
[repair] current action; reason; next step.
[metrics] current action; reason; next step.
[submit] current action; reason; next step.
```

Use `[verify]` after environment setup, dataset handling, checkpoint handling, smoke tests, evaluation, metric extraction, agent-made edits, and final packaging.

## Interaction stats

Track interaction stats in internal notes or the raw history trace when useful:

- `human_turns`: count user-sent messages, not agent tool calls or automatic repairs;
- `repair_count`: count bounded automatic repair attempts;
- `smoke_count`: count smoke commands run before final evaluation;
- `final_success`: true only if `result.md` was generated from a valid submission run.
- `verify_count`: count stage verification checks completed;
- `mismatch_count`: count verification mismatches that required repair, accepted drift, or blocker recording;
- `change_summary_ready`: true only if `change_summary.md` records agent edits and stage verification comparisons.

Do not add stats files to the final submission directory unless the user explicitly asks for them.
