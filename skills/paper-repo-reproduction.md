# paper-repo-reproduction

## Goal

Reproduce a requested result from a research repository with minimal human interaction. The output must be a traceable metric table and a complete conversation history.

## Inputs

- Repository path and URL
- Paper or table target
- Required benchmark, model, checkpoint, and dataset
- Hardware and API constraints
- Required submission table columns

## Workflow

1. Read `README`, scripts, configs, examples, and paper/table clues before asking questions.
2. Identify the official evaluation entrypoint and the smallest target configuration.
3. Inspect environment requirements: Python, CUDA, PyTorch, `flash-attn`, datasets, checkpoints, disk, GPU memory, and proxy.
4. Create an isolated conda or venv environment.
5. Run the smallest smoke test before expensive evaluation.
6. Run only the target benchmark/settings unless the assignment explicitly requires a full sweep.
7. Save raw logs, commands, configs, result JSON/CSV, and any code patches.
8. Extract only the requested metrics.
9. Write a Markdown table with only the required rows and columns.
10. Preserve the complete conversation history without deletion or rewriting.

## Decision Rules

- Prefer official scripts over rewritten evaluation logic.
- Prefer pretrained checkpoints over retraining when the assignment says to use author-provided models.
- Patch code only for compatibility bugs or broken script interfaces, and document the patch in the conversation history.
- If a required dataset/checkpoint/GPU/API key is missing, report the exact blocker and the smallest next action.
- Do not put secrets in prompts, code, committed files, result tables, or reports.

## Completion Criteria

- The metric table can be traced back to logs or result files.
- The submitted Markdown file contains only the target table.
- Conversation history proves setup, execution, failures, repairs, and metric extraction.
