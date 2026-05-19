# Metric Extraction and Submission Reference

## Responsibility

Extract metrics from real artifacts, enforce provenance, write fixed `result.md`, and preserve raw `history.md` as the benchmark interaction trace.

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

For each final metric, record provenance in work logs or raw trace artifacts:

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

## history.md raw trace rule

`history.md` is not a report template. It must be the complete raw Codex conversation record for the run.

It must include, as originally recorded:

- user inputs;
- assistant replies;
- tool calls;
- command execution records;
- errors and repair attempts;
- metric extraction process.

Forbidden:

- replacing history with a summary;
- replacing history with a reproduction report;
- rewriting, compressing, beautifying, filtering, or reorganizing the raw conversation;
- keeping only assistant output;
- fabricating conversation history.

Metric provenance can be visible in the raw trace and internal logs, but do not convert `history.md` into a polished provenance report.

## Final submission directory

Final package directory must contain only:

```text
repo_name/
  result.md
  history.md
```

Do not include raw checkpoints, datasets, cache, raw JSONL, scripts, outputs, logs, or work files in the final submission directory unless the user explicitly requires them.
