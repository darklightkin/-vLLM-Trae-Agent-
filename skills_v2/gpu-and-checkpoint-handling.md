# gpu-and-checkpoint-handling v2

## Responsibility

Handle hardware, CUDA, checkpoints, HuggingFace cache, dataset download, and memory fallback. This skill does not decide final metrics; it ensures resource preparation is reliable and traceable.

## Environment Preflight

Record:

- `nvidia-smi`;
- Python path and version;
- PyTorch version;
- `torch.cuda.is_available()`;
- CUDA/cuDNN version;
- free disk space;
- proxy variables;
- HuggingFace cache paths.

If CUDA is unavailable, explicitly record `CUDA unavailable`, and use CPU only for smoke, single-fold, or small-sample evaluation.

## Checkpoint Rules

- Prefer author-provided checkpoints over retraining.
- Confirm the source from README, paper, HF model card, or release.
- Download to the path expected by the official script, or use a wrapper to point to the actual path.
- If the official script expects fold directories, create only folds that truly exist.
- Do not copy one fold checkpoint into multiple folds unless the task explicitly allows it.
- Record checkpoint repo, filename, local path, hash or size in history.

## Dataset Rules

- Prefer HuggingFace `datasets` or the official download script.
- Record dataset ID, split, sample-count probe, and cache path.
- Do not fabricate samples or use random data as a benchmark.
- If data is gated/private, stop and report the required permission or token.

## HuggingFace Cache

Prefer workspace or repo-local cache, for example:

```powershell
$env:HF_HOME='<workspace>\outputs\hf_cache'
$env:HF_HUB_CACHE='<workspace>\outputs\hf_cache\hub'
$env:HF_DATASETS_CACHE='<workspace>\outputs\hf_cache\datasets'
```

Do not write tokens into files, prompts, tables, or history.

## OOM and Fallback

When CUDA OOM occurs, try in order:

1. lower batch size;
2. single fold / single config;
3. fp16/bf16 or disabling non-essential acceleration;
4. CPU smoke;
5. report that a larger GPU is required.

Treat `flash-attn` and custom CUDA extension build failures as environment blockers, not model failures.

## Cleanup Policy

The agent may automatically delete obvious temporary failed-cache files inside the workspace:

- `*.tmp`
- `*.incomplete`
- `__pycache__/`
- `.pytest_cache/`
- partial download residue under repo-local cache

Ask before deleting:

- source code, configs, scripts, README;
- checkpoints: `*.pt`, `*.pth`, `*.ckpt`, `*.safetensors`;
- raw dataset files, annotations, images, audio;
- submission or conversation history;
- paths outside the workspace.

When automatic cleanup is used, report only a status update:

```text
[cleanup] Removed failed temporary files from repo-local cache; next step is to probe the dataset again.
```
