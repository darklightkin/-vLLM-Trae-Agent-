# metric-extraction v2.1

## Responsibility

Extract target metrics from real run artifacts and generate the submission Markdown table. This skill handles metric trustworthiness and table format only; it does not explain the reproduction process.

## Input Sources

Priority order:

1. JSON/CSV result file from the target run;
2. stdout/stderr log from the target run;
3. official eval script output;
4. README/paper reference value.

README/paper reference values may be labeled as reference values, but must not be presented as local run results.

## Matching Rules

Before extracting metrics, confirm the run matches:

- repository;
- benchmark;
- model/method;
- checkpoint;
- fold or setting;
- dataset split;
- run time and output path.
- run type: `submission`, not `smoke` or `diagnostic`.

When multiple runs exist, use the one that best matches the task. Do not mix cells from different runs.

If the only successful run is a smoke or diagnostic run, do not generate a submit-ready final table. Write a blocker in history instead.

## Table Rules

Final filename priority:

1. task card explicit filename;
2. assignment official filename;
3. repo prompt explicit filename;
4. fallback `result.md`.

Do not hardcode `reproduction.md` when the task card asks for another name.

Fallback final filename:

```text
result.md
```

The file may contain only a Markdown table:

```markdown
| Method | System SRCC | Utterance SRCC |
| --- | --- | --- |
| A1 (Frozen+MSE) [recommended] | <value> | <value> |
```

Do not include commands, explanations, screenshots, provenance notes, or “Note:”. Put those in history.

If the task requires a different table shape, preserve the task-card row names and column names exactly.

## Numeric Rules

- Use the decimal precision required by the task; if unspecified, default to 3 decimals.
- Do not round in a misleading way to appear successful.
- If a metric is missing, write a clear blocker instead of guessing.
- If the run is single-fold, subset, CPU smoke, or otherwise incomplete, the table may still report the real value, but history must explain the limitation.
- If the run is smoke-only, do not write it as a final submission metric.
- If a diagnostic run uses train+validation+test or otherwise changes benchmark semantics, do not submit it.

## Acceptance

Before submission, confirm:

- every cell traces back to a raw file;
- no placeholder remains;
- table column names match the task;
- raw logs and JSON/CSV are not embedded in the final table file;
- history records metric source paths.
- history records whether the selected run is full, single-fold, subset, test-only, or otherwise limited.

## Metric Source Record

Add this record to history for every final metric:

| Field | Value |
| --- | --- |
| Metric name | exact column name |
| Value | extracted numeric value |
| Source file | JSON/CSV/log/stdout path |
| Source key or line | JSON key, CSV column, or log line |
| Run type | `submission` |
| Limitation | none or explicit deviation |
