# Metric Extraction and Submission

## Responsibility

Extract metrics from real local artifacts, enforce provenance, write `result.md`, preserve raw `history.md`, and write `change_summary.md`.

## Run Classification

| Type | Meaning | Can feed result.md |
| --- | --- | --- |
| `smoke` | import/model/data/minimal metric check | No |
| `diagnostic` | schema/split/cache/protocol inspection | No |
| `submission` | target benchmark/model/split/fold run | Yes |
| `official_table` | README/paper reference value | No |
| `blocked` | no valid local metric | No |

Only local `submission` metrics can populate `result.md`.

## Provenance

For every final metric, record:

| Metric | Value | Source file | Key/line/column | Run type | Split/fold | Local execution? |
| --- | --- | --- | --- | --- | --- | --- |

Allowed split/fold values include `test`, `validation`, `train`, `all`, `fold<N>`, `5-fold`, and `unknown`.

Verify:

- metric name matches official or required submission schema;
- value comes from local submission run;
- source file and key/line/column are known;
- split/fold matches target protocol;
- model/method names are unchanged;
- wrapper/edit did not change formula, labels, split, or aggregation.

## result.md

Fixed filename:

```text
result.md
```

Content: Markdown table only.

```markdown
| Method | Metric 1 | Metric 2 |
| --- | --- | --- |
| <method> | <value> | <value> |
```

No notes, commands, provenance, screenshots, or explanations.

## history.md

`history.md` must be the raw Codex conversation trace.

It should include original user inputs, assistant replies, tool calls, command records, errors, repair attempts, and metric extraction process.

Forbidden:

- summary/report replacement;
- compressed, beautified, filtered, or reordered transcript;
- assistant-only transcript;
- fabricated history.

If raw trace cannot be exported, record the run as blocked instead of fabricating `history.md`.

## change_summary.md

Separate from `history.md`. Summarize:

- source edits, wrappers, config copies, path adaptations, command changes, environment fallbacks;
- reason for each change;
- expected vs actual comparisons;
- stage verification results;
- whether benchmark semantics changed;
- unresolved mismatches, accepted drifts, and blockers.

Suggested table:

```markdown
| Stage | Official requirement | Local actual | Status | Source | Action |
| --- | --- | --- | --- | --- | --- |
```

Do not use `change_summary.md` to hide failed attempts, replace history, rename official identifiers, or present smoke/diagnostic metrics as final results.

## Final Directory

Final package contains only:

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

Do not include raw checkpoints, datasets, cache, scripts, outputs, logs, or work files unless the user explicitly requires them.
