# Paper Repo Reproduction

## Responsibility

Scan official materials, lock the target, find the official entrypoint, choose the smallest valid run, and keep the process traceable.

## Repo Scan

Read before heavy commands:

- README and reproduction docs;
- requirements/environment files;
- configs;
- train/eval/infer scripts and examples;
- paper/table/task-card references;
- model cards or issues when local docs are incomplete.

Inventory:

| Component | Find |
| --- | --- |
| repo | name, commit/tag, official instructions |
| env | Python, CUDA, torch, special packages |
| entrypoint | official eval/infer/train script and args |
| config | target model, benchmark, split, batch size |
| dataset | ID/path, split, schema, size, subset |
| checkpoint | source, filenames, size/hash, fold layout |
| metrics | output file, key/column/log line, metric names |
| submission | required directory and filenames |

Verify the inventory against README/config/scripts/paper/task card before installing or running expensive commands.

## Target Lock

Confirm:

- benchmark;
- method/model/checkpoint;
- table row and metric columns;
- split/fold protocol;
- accepted tolerance;
- whether one round, one fold, or full evaluation is required.

If repo docs and task card disagree, use the task card for submission requirements and record the discrepancy.

Preserve official spellings for model, method, dataset, split, benchmark, row labels, and metric names.

## Execution Policy

- Prefer checkpoint-based evaluation or inference reproduction.
- Do not choose training while an official checkpoint eval path exists.
- Do not download complete training data before checkpoint discovery.
- Run smoke/minimal validation before full evaluation.
- Full training is a last resort and requires task necessity plus user approval for long runs.

## Wrapper Policy

Default: do not modify upstream repo code.

Use external wrappers, env vars, local config copies, or command-line args for:

- local/offline paths;
- cache variables;
- small interface adapters;
- metric extraction helpers.

Wrappers must not change benchmark semantics, labels, splits, metric formulas, or official model/method names.

Record every agent-made edit or wrapper in `change_summary.md`:

```text
Original repo modified: Yes/No
Wrapper added: Yes/No
Reason
Official behavior changed: Yes/No
Verify: match/acceptable-drift/mismatch/unknown
```

## Generalized Lesson

Prefer wrappers and local configuration over source patches. Final packaging must include raw `history.md`; do not replace it with a cleaned report.
