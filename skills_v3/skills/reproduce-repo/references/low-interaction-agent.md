# Low Interaction Reference

## Responsibility

Reduce human turns while preserving correctness and auditability.

## Interaction counting

- Initial task prompt counts as `1`.
- User path fixes, GPU changes, credentials, private data decisions, deletion approvals, and source-edit approvals count as extra turns.
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

## Must ask when blocked

Ask only for:

- API key, account, token, or private dataset permission;
- paid download or license decision;
- machine/GPU/server switch;
- deletion of non-temporary files;
- modifying upstream code when forbidden;
- ambiguous final submission choice that cannot be resolved from task requirements.

## Progress updates

Use short stage updates:

```text
[scan] current action; reason; next step.
[env] current action; reason; next step.
[data] current action; reason; next step.
[model] current action; reason; next step.
[smoke] current action; reason; next step.
[eval] current action; reason; next step.
[repair] current action; reason; next step.
[metrics] current action; reason; next step.
[submit] current action; reason; next step.
```

## Required summary fields

At the end, ensure `run_summary.json` includes:

```json
{
  "human_turns": 1,
  "repair_count": 0,
  "smoke_count": 0,
  "final_success": false,
  "smallest_success_model": null
}
```

`smallest_success_model` means the smallest agent/model tier that successfully completed the reproduction, for example `haiku`, `sonnet`, `opus`, `codex`, or `unknown`.
