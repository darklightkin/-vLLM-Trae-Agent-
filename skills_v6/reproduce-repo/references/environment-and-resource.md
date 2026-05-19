# Environment and Resource

## Responsibility

Protect environment integrity, dependency correctness, dataset identity, and checkpoint identity before expensive runs.

## Environment Gate

Default: create a fresh repo-specific conda env. If conda is unavailable, create a fresh venv. Do not reuse `base`, system Python, current shell Python, or old envs unless the user explicitly requests reuse.

Record:

- repo-required Python/package/CUDA versions;
- actual Python executable and version;
- env name/path;
- PyTorch, CUDA/cuDNN, GPU visibility, `nvidia-smi`;
- disk space, proxy vars, HF/cache vars;
- dataset/checkpoint local paths.

Verify after env creation and dependency installation:

```text
Expected | Actual | Status | Source | Action
```

Treat a polluted env or CPU-only PyTorch on a GPU machine as a blocker unless the task explicitly allows CPU-only smoke testing.

## Setup Order

1. Create fresh env.
2. Verify `python` and `pip` are inside it.
3. Check GPU, disk, proxy, and cache paths.
4. Install and validate PyTorch/CUDA first.
5. Install project dependencies.
6. Run import/help smoke tests.

Useful checks:

```bash
which python
python -V
which pip
pip -V
python -c "import sys; print(sys.executable); print('\n'.join(sys.path))"
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.version.cuda); print(torch.cuda.device_count())"
```

## Dependency Policy

- Install only inside the isolated env.
- Change one variable at a time when repairing.
- Do not let a later `pip install` replace a verified PyTorch/CUDA stack.
- If a package tries to reinstall `torch`, `torchvision`, `torchaudio`, or `nvidia-*`, stop and retry with `--no-deps` when safe.
- Treat `flash-attn`, custom CUDA extension, and vLLM build failures as environment blockers unless proven otherwise.

## Mirror and Fallback Policy

Prefer reliable local/domestic mirrors when direct access is slow:

- PyPI mirrors such as Aliyun or Tsinghua;
- conda mirrors;
- HuggingFace mirrors via `HF_ENDPOINT`;
- GitHub archive/proxy routes for pinned commits.

Mirrors may change transport only. They must not change package identity, model ID, dataset ID, checkpoint, commit, split, metric, or evaluation protocol.

If an official pinned commit/wheel/model/dataset cannot be fetched, retry through mirror/archive routes first. If a nearest installable release is used instead, record it as:

```text
environment fallback
protocol deviation: acceptable-drift / mismatch
reason: network / unavailable source / incompatible build
```

## Dataset Gate

- Prefer local/offline resources when provided.
- Do not default to full training dataset downloads.
- Before downloading, estimate size, check disk, and choose the smallest valid eval subset.
- Use official scripts or HuggingFace `datasets` after size/path checks.
- Preserve official dataset IDs, split names, schema, labels, and preprocessing.

Record and verify:

- dataset name/ID/version/source;
- local path;
- split;
- sample/shard counts;
- schema and labels;
- subset choice and whether it is smoke, diagnostic, or submission-valid.

## Checkpoint Gate

- Prefer author checkpoints or pretrained weights over training.
- Record source/model card, filename, local path, size/hash when available, fold layout, architecture, and model name.
- Do not copy one checkpoint into multiple folds unless the task allows it.
- If checkpoint access fails, record the blocker before considering training.

Verify checkpoint identity against README, configs, scripts, model cards, or task card.

## Cleanup

Allowed without asking:

- workspace-local `*.tmp`, `*.incomplete`;
- `__pycache__/`, `.pytest_cache/`;
- failed partial downloads under repo-local cache.

Ask before deleting:

- source, configs, scripts, README;
- checkpoints, raw datasets;
- submission files, conversation history;
- paths outside the workspace.
