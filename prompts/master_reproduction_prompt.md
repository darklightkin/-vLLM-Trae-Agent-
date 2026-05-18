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
