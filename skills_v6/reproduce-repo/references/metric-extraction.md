# Metric Extraction and Submission Reference

## Responsibility

Extract metrics from real artifacts, enforce provenance, write fixed `result.md`, preserve raw `history.md` as the benchmark interaction trace, and write standalone `change_summary.md` for agent edits and stage-by-stage verification comparisons.

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

## Verify

After metric extraction, verify every metric against the repository, paper, task card, and evaluation script expectations.

Verify:

- metric name spelling exactly matches official materials or the required submission schema;
- value came from a local `submission` run, not from README, paper, smoke, diagnostic, or copied official table values;
- source file and source key/line are recorded;
- split/fold matches the target protocol;
- model name and method name are preserved exactly;
- no metric formula, label mapping, split, or aggregation was changed by an agent wrapper or edit.

Record expected vs actual metric names, split/fold, source artifacts, and any accepted drift in `change_summary.md`.

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

If raw trace cannot be exported from the host system, record `blocked` instead of fabricating `history.md`.

## change_summary.md rules

`change_summary.md` is a separate documentation artifact. It is not a replacement for `history.md`.

It must summarize:

- agent-made source edits, wrappers, config copies, command changes, path adaptations, and environment workarounds;
- why each change was made;
- before/after comparison against repository-specified versions or behavior;
- stage `Verify` results for repo scan, target selection, environment, dataset, checkpoint, smoke, evaluation, metrics, and packaging;
- whether benchmark semantics changed;
- unresolved mismatches, accepted drifts, and blockers.

Suggested table:

```markdown
| Stage | Official requirement | Local actual | Status | Source | Action |
| --- | --- | --- | --- | --- | --- |
| environment | Python 3.10 | Python 3.10.13 | match | README | none |
```

Do not use `change_summary.md` to rename official model names, rewrite the raw conversation, hide failed attempts, or present smoke/diagnostic metrics as final results.

## Final submission directory

Final package directory must contain only:

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

Do not include raw checkpoints, datasets, cache, raw JSONL, scripts, outputs, logs, or work files in the final submission directory unless the user explicitly requires them.
