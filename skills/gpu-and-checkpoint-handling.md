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
