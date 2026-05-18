# Metric Extraction and Submission Reference

## Responsibility

Extract metrics from real artifacts, enforce provenance, write fixed `result.md`, clean `conversation_history/history.md`, and `run_summary.json`.

## Run classification

Every run that produces metrics must be classified:

| Run type | Meaning | Can feed result.md |
| --- | --- | --- |
| `smoke` | minimal import/model/data/forward/metric check | No |
| `diagnostic` | schema/split/system/cache/protocol inspection | No |
| `submission` | target benchmark/model/split/fold chosen for final table | Yes |
| `official_table` | README or paper reference value | No |
| `blocked` | no valid local metric produced | No |

## Metric provenance table

For each final metric, record in `history.md`:

| Metric | Value | Source file | Source key/line | Run type | Split/fold | Local execution? |
| --- | --- | --- | --- | --- | --- | --- |

Allowed split/fold values include:

- `test`;
- `validation`;
- `train`;
- `all`;
- `fold0`, `fold1`, etc.;
- `5-fold`;
- `unknown`.

If the source is `all`, `smoke`, `diagnostic`, or `official_table`, do not use it in `result.md` unless the task explicitly says so.

## result.md rules

Final filename is fixed:

```text
result.md
```

Content must be only a Markdown table:

```markdown
| Method | Metric 1 | Metric 2 |
| --- | --- | --- |
| <method> | <value> | <value> |
```

No notes, commands, provenance, screenshots, or explanations.

## history.md required sections

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
## Metric Provenance
| Metric | Value | Source file | Source key/line | Run type | Split/fold | Local execution? |
| --- | --- | --- | --- | --- | --- | --- |
## Protocol Deviations
## Files Produced
## Interaction Count
```

## run_summary.json schema

Create:

```text
conversation_history/run_summary.json
```

Schema:

```json
{
  "human_turns": 1,
  "repair_count": 0,
  "smoke_count": 0,
  "final_success": true,
  "smallest_success_model": "unknown"
}
```

Definitions:

- `human_turns`: initial user prompt plus unavoidable user decisions.
- `repair_count`: count of bounded automatic repair attempts.
- `smoke_count`: count of smoke commands run before final evaluation.
- `final_success`: true only if `result.md` was generated from a valid submission run.
- `smallest_success_model`: smallest agent/model tier that completed the run, or `unknown`.

## Final submission directory

Final package directory must contain only:

```text
result.md
conversation_history/
```

Do not include raw checkpoints, datasets, cache, raw JSONL, or work logs in the final submission directory unless the official instructions require them.
