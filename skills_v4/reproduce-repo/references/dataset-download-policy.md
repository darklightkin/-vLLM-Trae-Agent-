# Dataset Download Policy

## Responsibility

Prevent accidental large downloads and prefer minimal, auditable data access for reproduction.

## Default Policy

- Do not download full datasets by default.
- Do not download datasets before checkpoint/pretrained-weight discovery, unless a small metadata-only probe is required to identify the evaluation protocol.
- Prefer existing local datasets, author-provided mini data, benchmark dev splits, validation splits, sample files, or streaming/subset loading.
- Estimate download size before downloading.
- Check free disk before downloading.
- Stop and explain before large downloads.

## Size Estimation

Estimate size using available sources:

- README and dataset cards;
- HuggingFace dataset metadata;
- release asset sizes;
- download script URLs and headers;
- existing local cache sizes;
- file manifests;
- issue discussions or benchmark docs.

If size cannot be estimated and the dataset may be large, stop before download and report the uncertainty.

## Disk Check

Before downloading any dataset, record:

- target cache path;
- available disk on that volume;
- estimated compressed size;
- estimated extracted size when known;
- expected temporary peak usage.

Require enough room for compressed bytes, extracted bytes, and temporary files.

Do this check even for official download scripts. Official scripts are not permission to download large data blindly.

## Thresholds

Use these default stop thresholds unless the user provided different limits:

- Stop before any single dataset download estimated above 20 GB.
- Stop before any workflow estimated above 50 GB total.
- Stop before any TB-scale dataset.
- Stop if free disk is less than 2x the estimated compressed size or less than estimated extracted peak usage.

When stopped, explain the smallest viable alternative: checkpoint evaluation, sample split, streaming, user-provided local path, or official reported metric comparison.

If pretrained checkpoints are available, prefer checkpoint-based evaluation with the smallest valid evaluation subset over full dataset download or training.

## Minimal Subset Strategy

Prefer:

- official validation/dev/test mini split;
- documented sample data;
- first N examples only for smoke tests;
- streaming reads where supported;
- metadata-only probes before media/blob download.

Clearly classify subset metrics:

- `smoke` for pipeline checks;
- `diagnostic` for schema or protocol checks;
- `submission` only when the subset is the official requested evaluation protocol.

## Download Permission Gate

Ask the user only when:

- the dataset requires credentials, license acceptance, paid access, or private data approval;
- the estimated download exceeds thresholds;
- the target path is outside the workspace and may consume shared storage;
- download size cannot be estimated and risk is high.

Otherwise use the safest minimal dataset path and record it.
