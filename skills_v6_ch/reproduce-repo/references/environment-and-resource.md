# Environment and Resource

## 职责

在运行昂贵复现命令前，保护环境完整性和资源正确性。

## Environment gate

安装pytorch环境默认先使用清华源https://pypi.tuna.tsinghua.edu.cn/simple/

记录：

- 仓库要求的 Python version；
- Python executable 和 version；
- 仓库明确要求的 PyTorch/CUDA/package versions；
- fresh conda/venv name 或 path；
- PyTorch version；
- `torch.cuda.is_available()`；
- CUDA/cuDNN version；
- `nvidia-smi`；
- free disk；
- proxy variables；
- HuggingFace cache variables；
- 相关 dataset/checkpoint paths。

不要污染 base/system Python。默认创建新的 repo-specific conda environment；如果 conda 不可用，则创建新的 venv。环境名应包含 repo name 和 timestamp。除非用户明确要求复用，否则不要复用已有 conda/venv environments、`base`、current shell Python 或 old repo environments。

### Verify

Environment creation 后和 dependency installation 后，将本地环境与 repository-specified requirements 对比。

Verify：

- README、`.python-version`、`environment.yml`、`pyproject.toml`、`requirements.txt`、install docs 或 task card 中的 Python version；
- PyTorch、CUDA、torchvision、torchaudio、transformers、vLLM、flash-attn，以及其他明确 pin version 的 packages；
- GPU visibility 和 CUDA compatibility；
- 仓库使用的 command entrypoints；
- model names 和 package-specific model identifiers 必须与官方材料写法完全一致。

在工作记录和 `change_summary.md` 中记录 expected value、actual value、expectation source、match status 和 repair action。

## Environment setup workflow

使用分阶段 setup process，而不是一次性安装所有依赖。

默认顺序：

1. 创建 fresh isolated environment。
2. Verify `python` 和 `pip` 属于该 isolated environment。
3. 检查 GPU driver、disk space、proxy variables 和 cache paths。
4. 先安装 PyTorch/CUDA。
5. Validate PyTorch 和 CUDA。
6. 安装 remaining project dependencies。
7. Verify installed versions against repository requirements。
8. 在昂贵命令前运行 import/help smoke tests。
9. Verify smoke commands 与 repository entrypoints 匹配，或记录 wrapper reason。

PyTorch/CUDA validation 通过前，不要运行昂贵 evaluation commands。

推荐 isolation check：

```bash
which python
python -V
which pip
pip -V
python -c "import sys; print(sys.executable); print('\n'.join(sys.path))"
```

如果 `sys.path` 包含 `/usr/local/lib/python.../dist-packages` 等 system Python paths，则视为环境污染。

---

## PyTorch and CUDA policy

对于 PyTorch-based repositories，在安装完整 project requirements 前，先安装并验证 PyTorch。

不要在 PyTorch/CUDA validation 前盲目运行：

```bash
pip install -r requirements.txt
```

GPU PyTorch 的推荐安装方式：

```bash
conda install -y pytorch torchaudio pytorch-cuda=12.1 \
  -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch \
  -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/nvidia
```

Validation command：

```bash
python -c "import torch; \
print('torch:', torch.__version__); \
print('cuda available:', torch.cuda.is_available()); \
print('torch cuda:', torch.version.cuda); \
print('gpu count:', torch.cuda.device_count())"
```

如果 `nvidia-smi` 可见 GPU，但输出显示：

```text
torch: x.y.z+cpu
cuda available: False
torch cuda: None
gpu count: 0
```

则视为 environment blocker。

---

## Dependency gate

- 只在 isolated environment 中安装 dependencies。
- 修复前尽量保存当前 environment state。
- 如果出现 dependency conflicts，一次只改变一个变量。
- 除非证明不是 blocker，否则将 `flash-attn`、custom CUDA extension 和 vLLM build failures 视为 environment blockers。
- 在安装 remaining dependencies 前，先安装并验证 PyTorch/CUDA。
- 如果 dependency installation 试图替换已验证的 PyTorch/CUDA stack，停止并记录后再继续。
- 不要让 pip 替换可工作的 conda PyTorch installation。

如果 PyTorch 已经验证后，安装 package 时试图拉取新的 `torch`、`torchaudio`、`torchvision` 或 `nvidia-*` CUDA stack，则停止并重试：

```bash
python -m pip install <package-name> --no-deps
```

然后只显式安装缺失的小依赖。

每次 dependency repair 后都重新运行 `Verify`。除非 mismatch 被标记为 `acceptable-drift` 且有 reason 和 source，否则不要带着未记录的 version mismatch 进入 dataset、checkpoint、smoke 或 evaluation 阶段。

## Dataset gate

- 不要默认下载完整 training datasets。
- Dataset download 前先估算 dataset size、检查 free disk，并优先选择必要的最小 subset。
- 优先使用 local/offline resources、official mini/dev/eval subsets、streaming 或 metadata probes。
- 只有在完成 size 和 disk checks 后，才使用 official download scripts 或 HuggingFace `datasets`。
- 有 local/offline resources 时优先使用它们。
- 记录 dataset ID/path、split、sample-count probe 和 schema。
- 记录为什么需要该 dataset，以及为什么更小替代方案足够或不够。
- 如果 system-level metric 需要 system/model IDs，但 dataset 缺少字段，先检查 filenames、metadata、README 或 official analysis scripts，不要自行发明 grouping。
- 如果一次运行混合 train/validation/test，除非 official protocol 如此要求，否则将其归类为 diagnostic。

### Verify

Dataset discovery、download、binding、schema probing 或 preprocessing 后，将本地 dataset 与 repository requirements 对比。

Verify：

- dataset ID/name 和 version 或 commit；
- local path 和 source；
- split names；
- official materials 提供时的 sample counts 或 shard counts；
- schema columns 和 label names；
- preprocessing/tokenizer/feature extraction settings；
- subset selection 以及它是 smoke、diagnostic 还是 submission-valid。

在 `change_summary.md` 中记录 mismatches。不要为了让结果更整洁而重命名 split names、class labels、dataset names 或 model-related identifiers。

## Checkpoint gate

- 优先使用 author checkpoints，而不是 retraining。
- 如果 pretrained checkpoints 或 model weights 可用，在考虑 training 前先用于 evaluation/inference reproduction。
- 记录 checkpoint source、filename、local path、hash 或 size，以及 fold layout。
- 如果只有一个 checkpoint，但论文报告 5-fold CV，记录 limitation。
- 除非任务明确允许，不要把一个 checkpoint 复制到多个 folds。
- 如果 checkpoint access 失败，在 fallback 到 training 前记录失败。

### Verify

Checkpoint 或 pretrained weight discovery/download 后，将本地 checkpoint 与 repository requirements 对比。

Verify：

- official checkpoint source URL 或 model card；
- filename；
- local path；
- size、hash 或其他 integrity signal；
- fold layout 和 checkpoint 数量；
- architecture、model name、method name 和 config compatibility；
- checkpoint 是 author-provided、third-party、locally trained 还是 official-table-only。

在 `change_summary.md` 中记录 expected vs actual values。保留 official model 和 method names 的准确写法。


