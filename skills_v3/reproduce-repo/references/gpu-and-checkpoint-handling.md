# GPU and Checkpoint Handling Reference

## Responsibility

Protect environment integrity and resource correctness before running expensive reproduction commands.

## Environment gate

Record:

- Python executable and version;
- conda/venv name or path;
- PyTorch version;
- `torch.cuda.is_available()`;
- CUDA/cuDNN version;
- `nvidia-smi`;
- free disk;
- proxy variables;
- HuggingFace cache variables;
- relevant dataset/checkpoint paths.

Do not pollute base/system Python. Use a repo-specific conda or venv, or a verified existing isolated environment.

## Dependency gate

- Install dependencies only inside the isolated environment.
- Save current environment state before repairs when practical.
- If dependency conflicts occur, change one variable at a time.
- Treat `flash-attn`, custom CUDA extension, and vLLM build failures as environment blockers unless proven otherwise.

## Dataset gate

- Prefer official download scripts or HuggingFace `datasets`.
- Prefer local/offline resources when provided.
- Record dataset ID/path, split, sample-count probe, and schema.
- If a system-level metric needs system/model IDs but the dataset lacks a field, inspect filenames, metadata, README, or official analysis scripts before inventing grouping.
- If a run combines train/validation/test, classify it as diagnostic unless official protocol says otherwise.

## Checkpoint gate

- Prefer author checkpoints over retraining.
- Record checkpoint source, filename, local path, hash or size, and fold layout.
- If only one checkpoint exists but the paper reports 5-fold CV, record the limitation.
- Do not copy one checkpoint into multiple folds unless explicitly allowed by the task.

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
