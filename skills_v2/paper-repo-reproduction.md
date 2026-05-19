# paper-repo-reproduction v2

## Responsibility

Handle the general paper-repository reproduction workflow: read official materials, identify the official entrypoint, build an inventory, choose the smallest verifiable run, and make the process traceable.

This is a sub-skill for `reproduction-master.md`. It handles repository and workflow issues only. It does not handle GPU/checkpoint details or final table formatting.

## Inputs

- GitHub URL or local repo path;
- target result from paper/table/README;
- benchmark, model, checkpoint, dataset;
- submission requirements;
- whether source-code edits in the original repository are allowed.

## Repo Inventory

Build an inventory before running commands:

| Component | What to Find |
| --- | --- |
| README | official reproduction notes, target tables, download links |
| requirements | Python, CUDA, torch, special packages |
| configs | target model and benchmark configs |
| scripts | train/eval/infer entrypoints |
| data loader | dataset name, split, fields |
| checkpoint | author weights, fold structure, filenames |
| output | logs, JSON, CSV, metrics paths |

## Workflow

1. Read README, requirements, configs, scripts, and examples.
2. Identify the official evaluation entrypoint.
3. Identify the smallest target configuration; avoid unrelated full sweeps.
4. Check repository completeness: eval, config, dataset, checkpoint, and metric parser.
5. Run smoke first, then the target command.
6. Save all commands, logs, and result paths.
7. Classify failures and route them to the corresponding sub-skill or master workflow.

## Source-Code Edit Rules

- Do not modify original repository code by default.
- If the prompt explicitly forbids source edits, use wrappers, environment variables, temporary copies, or record blockers.
- If the prompt allows source edits, apply only the smallest compatibility patch and record diff, reason, and validation command.
- Never modify metrics, dataset split, or evaluation protocol just to improve the result.

## Success Criteria

- The official target command is known.
- The actually successful command is known.
- Any deviation from the official workflow is explained.
- Final metrics can be traced to real logs or result files.
