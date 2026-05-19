# metric-extraction

## Goal

Extract only the requested reproduction metrics and write the official Markdown table.

## Inputs

- Target metric names and rows
- Logs, JSON, CSV, stdout, or README reference values
- Submission folder path

## Rules

- The final `reproduction.md` must contain only a Markdown table.
- Include row names and column names.
- Do not include commands, explanations, notes, screenshots, or provenance text in `reproduction.md`.
- Keep provenance in logs and conversation history, not in the final table.
- If multiple runs exist, use the run that matches the required model, benchmark, and setting.

## Table Naming

Use the file name required by the assignment examples:

```text
reproduction.md
```

Do not rename it to `result.md` unless the instructor explicitly changes the format.

## Validation

Before final packaging:

1. Confirm every cell is from a matching run or official target reference.
2. Confirm no placeholder remains.
3. Confirm the table has only the required metric columns.
4. Confirm the corresponding raw output is saved outside the final table.
