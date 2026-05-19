# Low Interaction

## Responsibility

Reduce human turns while preserving correctness and auditability.

## Counting

Count human turns by user-sent messages. Agent scans, commands, progress updates, automatic repairs, and log analysis do not count.

Record unavoidable questions as `interaction_count_candidate`.

## Do Not Ask When Inferable

Infer from README, configs, scripts, paper, model card, issues, local directories, `git remote -v`, offline resource notes, environment probes, dataset/checkpoint inspection, logs, and tracebacks.

Defaults:

- create a fresh isolated environment;
- search for author checkpoints/pretrained weights first;
- run checkpoint-based evaluation/inference before training;
- use smallest valid smoke/eval subset first;
- verify stage outputs against official materials.

## Must Ask When Blocked

Ask only for:

- API key, account, token, or private dataset permission;
- paid download or license decision;
- large or unknown-size dataset download;
- long/full training;
- machine/GPU/server switch;
- system-level modification;
- deletion of non-temporary files;
- forbidden upstream source edit;
- ambiguous final submission choice that cannot be resolved from task materials.

## Progress Updates

Use short updates:

```text
[scan] current action; reason; next step.
[verify] expected vs actual; status; next action.
[env] current action; reason; next step.
[data] current action; reason; next step.
[model] current action; reason; next step.
[smoke] current action; reason; next step.
[eval] current action; reason; next step.
[repair] current action; reason; next step.
[metrics] current action; reason; next step.
[submit] current action; reason; next step.
```

## Internal Stats

Track when useful:

- `human_turns`;
- `repair_count`;
- `smoke_count`;
- `verify_count`;
- `mismatch_count`;
- `final_success`;
- `change_summary_ready`.

Do not add stats files to the final submission directory unless the user explicitly asks.
