# GPU and Checkpoint Handling

## Responsibility

Protect resource correctness before expensive runs and ensure checkpoints are real, compatible, and traceable.

## Resource Gate

Record:

- Python executable and version;
- environment name/path;
- PyTorch version;
- CUDA availability;
- CUDA/cuDNN version when available;
- `nvidia-smi` output when available;
- GPU memory;
- free disk;
- proxy variables relevant to downloads;
- HuggingFace, torch, pip, and dataset cache paths.

## GPU Policy

- Prefer official precision and device settings.
- For OOM, reduce batch size, sequence length for smoke only, or enable documented eval memory options.
- Do not change final benchmark semantics to fit GPU memory.
- If required hardware is unavailable, record a blocker.

## Checkpoint Gate

Prefer author checkpoints over retraining.

If pretrained checkpoints are available, the agent must prioritize checkpoint-based evaluation reproduction over full training reproduction.

Record:

- checkpoint source;
- local path;
- size;
- hash when practical;
- expected architecture;
- fold layout;
- license or access requirement;
- whether it was downloaded during the run.

Do not copy one checkpoint into multiple folds unless explicitly allowed.

## Checkpoint Compatibility

Before final evaluation:

- verify checkpoint loads into the intended model;
- inspect missing/unexpected keys;
- confirm tokenizer/processor/config compatibility;
- distinguish architecture mismatch from prefix/key naming mismatch;
- use key-prefix adapters only when semantics are unchanged.

## Cache Policy

- Prefer workdir-local caches for new downloads.
- Do not delete global caches broadly.
- Keep downloaded checkpoints out of final submission directory.
- Preserve logs and provenance for checkpoint acquisition.
