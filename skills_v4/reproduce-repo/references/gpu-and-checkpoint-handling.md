# GPU and Checkpoint Handling Reference

## Responsibility

Protect environment integrity and resource correctness before running expensive reproduction commands.

## Environment gate

Record:

- Python executable and version;
- fresh conda/venv name or path;
- PyTorch version;
- `torch.cuda.is_available()`;
- CUDA/cuDNN version;
- `nvidia-smi`;
- free disk;
- proxy variables;
- HuggingFace cache variables;
- relevant dataset/checkpoint paths.

Do not pollute base/system Python. Default to a new repo-specific conda environment; if conda is unavailable, create a new venv. The environment name should include the repo name and timestamp. Do not reuse existing conda/venv environments, `base`, current shell Python, or old repo environments unless the user explicitly requests reuse.

## Dependency gate

- Install dependencies only inside the isolated environment.
- Save current environment state before repairs when practical.
- If dependency conflicts occur, change one variable at a time.
- Treat `flash-attn`, custom CUDA extension, and vLLM build failures as environment blockers unless proven otherwise.

## Dataset gate

- Do not default to downloading complete training datasets.
- Before dataset download, estimate dataset size, check free disk, and prefer the smallest necessary subset.
- Prefer local/offline resources, official mini/dev/eval subsets, streaming, or metadata probes when provided.
- Use official download scripts or HuggingFace `datasets` only after size and disk checks.
- Prefer local/offline resources when provided.
- Record dataset ID/path, split, sample-count probe, and schema.
- Record why the dataset was needed and why a smaller alternative was or was not sufficient.
- If a system-level metric needs system/model IDs but the dataset lacks a field, inspect filenames, metadata, README, or official analysis scripts before inventing grouping.
- If a run combines train/validation/test, classify it as diagnostic unless official protocol says otherwise.

## Checkpoint gate

- Prefer author checkpoints over retraining.
- If pretrained checkpoints or model weights are available, use them for evaluation/inference reproduction before considering training.
- Record checkpoint source, filename, local path, hash or size, and fold layout.
- If only one checkpoint exists but the paper reports 5-fold CV, record the limitation.
- Do not copy one checkpoint into multiple folds unless explicitly allowed by the task.
- If checkpoint access fails, record the failure before falling back to training.

## Cleanup policy

Allowed without asking:

- workspace-local `*.tmp`;
- workspace-local `*.incomplete`;
- `__pycache__/`;
- `.pytest_cache/`;
- failed partial downloads under repo-local cache.

Must ask before deleting:

- source code;
- configs;
- scripts;
- README;
- checkpoints;
- raw datasets;
- submission files;
- conversation history;
- paths outside the workspace.
