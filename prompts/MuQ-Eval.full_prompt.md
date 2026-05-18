# Master Prompt: Low-Interaction Paper Repo Reproduction

You are reproducing a research repository result for a course assignment. Your goal is to complete the reproduction with the fewest possible human interactions while preserving full traceability.

Use these skills:

- `paper-repo-reproduction`
- `low-interaction-agent`
- `metric-extraction`
- `gpu-and-checkpoint-handling`

Rules:

1. Do not ask clarifying questions first. Inspect the local repository, README, scripts, configs, and available task card.
2. Create or use an isolated environment. Do not pollute system Python.
3. Detect GPU, CUDA, disk, proxy, HuggingFace cache, dataset paths, and checkpoint paths.
4. Prefer official evaluation scripts and pretrained checkpoints.
5. Run the smallest smoke test before expensive commands.
6. On failure, inspect logs and repair automatically with a bounded loop.
7. Keep complete conversation history. Do not delete or rewrite history.
8. Do not expose API keys or tokens in prompts, files, logs, tables, or reports.
9. The final submission file for each repo must be `reproduction.md`, containing only the required Markdown table.
10. Save raw logs and intermediate outputs locally for traceability, but final packaging should include only official required files.

At the end, report:

- reproduced metric table path
- raw result/log path
- checkpoint and dataset source
- human interaction count estimate
- smallest successful base model if known


# Repository Task Card
Reproduce the requested paper result with the fewest possible human interactions.

Repository: https://github.com/dgtql/MuQ-Eval
Local path: D:\vLLM1\model\MuQ-Eval
Paper: https://arxiv.org/abs/2603.22677
Target: Use the author-provided model and run one evaluation round to reproduce the two highlighted metrics.
Benchmark: MusicEval
Recommended model/checkpoint: zhudi2825/MuQ-Eval-A1
Required settings:
- {"method": "A1 (Frozen+MSE) [recommended]"}

Requirements:
1. Inspect the repository README and scripts before running anything expensive.
2. Create or reuse an isolated environment and record the exact commands.
3. Prefer the repository's official evaluation script: python run_evaluation.py --checkpoint-dir outputs/A1_frozen_mlp --config configs/A1_frozen_mlp.yaml --all-folds --bootstrap
4. Save raw logs and result files under an outputs/results directory.
5. Produce only the required reproduction metrics table for submission.
6. Preserve the complete conversation history.
7. If a dependency, dataset, checkpoint, GPU, or API key is missing, report the smallest concrete blocker and the command or path needed to continue.

Submission folder name: MuQ-Eval
Table columns: Method, System SRCC, Utterance SRCC


# Skill Details
--- Skill: gpu-and-checkpoint-handling ---
# gpu-and-checkpoint-handling

## Goal

Handle GPU selection, checkpoint preparation, dataset caching, and memory fallback during reproduction.

## Preflight

Check:

- `nvidia-smi`
- Python and CUDA versions
- PyTorch CUDA availability
- Free disk space
- HuggingFace cache and proxy configuration
- Required dataset paths
- Required checkpoint paths

## Checkpoint Rules

- Prefer author-provided checkpoints over retraining.
- Download checkpoints into the path expected by the official script.
- If the official script expects fold directories, create the minimal matching structure only when justified.
- Do not silently copy one fold checkpoint into all folds unless the assignment explicitly allows it.
- Record checkpoint source and filename in conversation history.

## GPU Rules

- Use the smallest viable GPU first.
- If CUDA out-of-memory occurs, try lower batch size, single-fold/single-config runs, CPU smoke tests, or a different GPU.
- For Windows, prefer Linux server or WSL2 for CUDA-heavy repos.
- Treat `flash-attn` and custom CUDA build failures as environment blockers, not model failures.

## Network Rules

- Use local cache when available.
- If HuggingFace download fails, report the exact model or dataset ID.
- Never write API keys or access tokens into prompts, source files, or submission artifacts.


--- Skill: low-interaction-agent ---
# low-interaction-agent

## Goal

Minimize human-to-agent interaction rounds while keeping the reproduction process correct and auditable.

## Interaction Accounting

- The initial task prompt counts as one user interaction.
- User-provided corrections, path fixes, GPU changes, and manual decisions count as additional interactions.
- Agent tool calls, automatic exploration, log analysis, and self-repair do not count.
- Agent clarification questions should be avoided unless no safe default exists.

## Behavior Rules

1. Start by scanning local files and official docs in the repo.
2. Make safe defaults instead of asking obvious questions.
3. Use local paths discovered from the workspace before asking for paths.
4. Try a bounded repair loop after failures.
5. Stop only when the next action requires unavailable data, credentials, hardware, or user-owned policy decisions.

## Safe Defaults

- Use a new isolated environment named after the repo.
- Use CPU only for smoke tests; use CUDA for target evaluation if available.
- Use pretrained checkpoints when the README provides them.
- Run one target configuration before a full sweep.
- Save outputs under `outputs/`, `results/`, or a repo-local run directory.

## Failure Loop

For each failure:

1. Read the full traceback or log.
2. Classify the failure: dependency, config, path, dataset, checkpoint, GPU memory, network, or code mismatch.
3. Apply the smallest repair.
4. Re-run the smallest command that reaches the failed point.
5. After repeated failures, summarize the blocker and exact next action.


--- Skill: metric-extraction ---
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


--- Skill: paper-repo-reproduction ---
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
