# metric-extraction v2

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

When multiple runs exist, use the one that best matches the task. Do not mix cells from different runs.

## Table Rules

Default final filename:

```text
reproduction.md
```

The file may contain only a Markdown table:

```markdown
| Method | System SRCC | Utterance SRCC |
| --- | --- | --- |
| A1 (Frozen+MSE) [recommended] | <value> | <value> |
```

Do not include commands, explanations, screenshots, provenance notes, or “Note:”. Put those in history.

## Numeric Rules

- Use the decimal precision required by the task; if unspecified, default to 3 decimals.
- Do not round in a misleading way to appear successful.
- If a metric is missing, write a clear blocker instead of guessing.
- If the run is single-fold, subset, CPU smoke, or otherwise incomplete, the table may still report the real value, but history must explain the limitation.

## Acceptance

Before submission, confirm:

- every cell traces back to a raw file;
- no placeholder remains;
- table column names match the task;
- raw logs and JSON/CSV are not embedded in `reproduction.md`;
- history records metric source paths.
