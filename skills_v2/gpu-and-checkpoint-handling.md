# gpu-and-checkpoint-handling v2

## 职责

负责硬件、CUDA、checkpoint、HuggingFace cache、数据下载和显存降级。它不决定最终指标，只保证资源准备过程可靠且可追溯。

## 环境预检查

记录以下结果：

- `nvidia-smi`；
- Python 路径和版本；
- PyTorch 版本；
- `torch.cuda.is_available()`；
- CUDA/cuDNN 版本；
- 可用磁盘空间；
- 代理变量；
- HuggingFace cache 路径。

如果 CUDA 不可用，明确记录 `CUDA unavailable`，并只用 CPU 做 smoke、单 fold 或小样本评测。

## Checkpoint 规则

- 优先作者提供 checkpoint，不重新训练。
- 从 README、paper、HF model card 或 release 中确认来源。
- 下载到官方脚本期望的位置，或用 wrapper 指向实际路径。
- 如果官方脚本需要 fold 目录，只创建真实存在的 fold。
- 不要把一个 fold 的 checkpoint 复制成多个 fold，除非任务明确允许。
- history 记录 checkpoint repo、文件名、本地路径、hash 或大小。

## Dataset 规则

- 优先用 HuggingFace `datasets` 或官方下载脚本。
- 记录 dataset ID、split、样本数探测、cache 路径。
- 不伪造样本，不用随机数据冒充 benchmark。
- 如果数据 gated/private，停止并说明需要的权限或 token。

## HuggingFace Cache

优先使用 workspace 或 repo-local cache，例如：

```powershell
$env:HF_HOME='<workspace>\outputs\hf_cache'
$env:HF_HUB_CACHE='<workspace>\outputs\hf_cache\hub'
$env:HF_DATASETS_CACHE='<workspace>\outputs\hf_cache\datasets'
```

不要把 token 写入文件、prompt、表格或 history。

## OOM 和降级

遇到 CUDA OOM 时按顺序尝试：

1. 降低 batch size；
2. 单 fold / 单配置；
3. fp16/bf16 或关闭非必要加速；
4. CPU smoke；
5. 报告需要更大 GPU。

`flash-attn`、自定义 CUDA extension 编译失败，归类为环境 blocker，不归类为模型失败。

## 清理策略

可以自动删除 workspace 内明显临时的失败缓存：

- `*.tmp`
- `*.incomplete`
- `__pycache__/`
- `.pytest_cache/`
- repo-local cache 下的部分下载残留

必须询问后才能删除：

- 源码、配置、脚本、README；
- checkpoint：`*.pt`、`*.pth`、`*.ckpt`、`*.safetensors`；
- 数据集原始文件、标注、图片、音频；
- submission 或 conversation history；
- workspace 外路径。

自动清理时只发状态：

```text
[清理] 删除 repo-local cache 中失败下载的临时文件；下一步重新探测数据集。
```
