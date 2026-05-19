# Metric Extraction

## Responsibility

Extract final metrics from real local artifacts, enforce provenance, generate `result.md`, and write `run_summary.json`.

## Run Classification

| Run type | Meaning | Can feed result.md |
| --- | --- | --- |
| `smoke` | minimal import/model/data/forward/metric check | No |
| `diagnostic` | schema/split/system/cache/protocol inspection | No |
| `submission` | target benchmark/model/split/fold chosen for final table | Yes |
| `official_table` | README or paper reference value | No |
| `blocked` | no valid local metric produced | No |

## Metric Provenance

Every metric considered for final output must record:

- metric name;
- value;
- source file;
- source key, CSV column, or log line;
- run type;
- split/fold;
- whether it came from local execution;
- command that produced it when available.

Only local `submission` metrics may populate `result.md`.

## result.md Rules

Final filename is fixed:

```text
result.md
```

Content must be only a Markdown table:

```markdown
| Method | Metric |
| --- | --- |
| <method> | <value> |
```

No notes, commands, screenshots, logs, or provenance text.

## run_summary.json

Create:

```text
conversation_history/run_summary.json
```

Schema:

```json
{
  "repo_name": "",
  "success": false,
  "failure": null,
  "human_turns": 1,
  "repair_attempts": 0,
  "smoke_runs": 0,
  "final_metric": null,
  "metric_source": null,
  "smallest_success_model": "unknown",
  "environment_name": null,
  "dataset_downloaded": false,
  "checkpoint_used": false
}
```

## Final Submission Directory

The final directory must contain only:

```text
repo_name/
  result.md
  conversation_history/
```

Keep raw logs, JSON, CSV, checkpoints, caches, wrappers, scripts, and temp files in the work directory, not the final submission directory.

`conversation_history/history.md`, when required, must be the original Codex conversation trace, not a metric provenance report, command summary, or rewritten reproduction narrative. Metric provenance belongs in raw artifacts or work logs unless the benchmark explicitly requires it inside the preserved trace.
