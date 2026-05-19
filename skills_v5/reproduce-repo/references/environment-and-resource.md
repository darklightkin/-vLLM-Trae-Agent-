# Environment and Resource

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

## Environment setup workflow

Use a staged setup process instead of installing all dependencies at once.

Default order:

1. Create a fresh isolated environment.
2. Verify that `python` and `pip` belong to the isolated environment.
3. Check GPU driver, disk space, proxy variables, and cache paths.
4. Install PyTorch/CUDA first.
5. Validate PyTorch and CUDA.
6. Install remaining project dependencies.
7. Run import/help smoke tests before expensive commands.

Do not run expensive evaluation commands before PyTorch/CUDA validation passes.

Recommended isolation check:

```bash
which python
python -V
which pip
pip -V
python -c "import sys; print(sys.executable); print('\n'.join(sys.path))"
```

If `sys.path` contains system Python paths such as `/usr/local/lib/python.../dist-packages`, treat the environment as polluted.

---

## PyTorch and CUDA policy

For PyTorch-based repositories, install and validate PyTorch before installing the full project requirements.

Do not blindly run:

```bash
pip install -r requirements.txt
```

before PyTorch/CUDA validation.

Preferred installation for GPU PyTorch:

```bash
conda install -y pytorch torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```

Validation command:

```bash
python -c "import torch; \
print('torch:', torch.__version__); \
print('cuda available:', torch.cuda.is_available()); \
print('torch cuda:', torch.version.cuda); \
print('gpu count:', torch.cuda.device_count())"
```

If GPUs are visible in `nvidia-smi` but the output shows:

```text
torch: x.y.z+cpu
cuda available: False
torch cuda: None
gpu count: 0
```

treat it as an environment blocker.

---

## Dependency gate

- Install dependencies only inside the isolated environment.
- Save current environment state before repairs when practical.
- If dependency conflicts occur, change one variable at a time.
- Treat `flash-attn`, custom CUDA extension, and vLLM build failures as environment blockers unless proven otherwise.
- Install and validate PyTorch/CUDA before installing the remaining dependencies.
- If a dependency installation tries to replace a verified PyTorch/CUDA stack, stop and record it before continuing.
- Do not let pip replace a working conda PyTorch installation.

If installing a package tries to pull a new `torch`, `torchaudio`, `torchvision`, or `nvidia-*` CUDA stack after PyTorch has already been validated, stop and retry with:

```bash
python -m pip install <package-name> --no-deps
```

Then install only missing small dependencies explicitly.


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
