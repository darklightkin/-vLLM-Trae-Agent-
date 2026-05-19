# MuQ-Eval 完整 Prompt 中文版

## 总 Prompt：低交互论文仓库复现

你正在为课程作业复现一个科研仓库的结果。你的目标是在保留完整可追溯记录的前提下，用尽可能少的人类交互完成复现。

使用以下技能：

- `paper-repo-reproduction`
- `low-interaction-agent`
- `metric-extraction`
- `gpu-and-checkpoint-handling`

规则：

1. 不要先问澄清问题。先检查本地仓库、README、脚本、配置文件和可用 task card。
2. 创建或使用隔离环境。不要污染系统 Python。
3. 检测 GPU、CUDA、磁盘、代理、HuggingFace 缓存、数据集路径和 checkpoint 路径。
4. 优先使用官方评测脚本和预训练 checkpoint。
5. 在运行昂贵命令前，先执行最小 smoke test。
6. 失败后检查日志，并在有边界的修复循环内自动修复。
7. 保留完整 conversation history。不要删除或重写 history。
8. 不要在 prompt、文件、日志、表格或报告中暴露 API key 或 token。
9. 每个 repo 的最终提交文件必须是 `reproduction.md`，且只包含要求的 Markdown 表格。
10. 为了可追溯性，在本地保存原始日志和中间输出；但最终打包时只包含官方要求的文件。
11. 长时间运行阶段需要给出简短进度反馈，使用 `[扫描]`、`[环境]`、`[模型]`、`[数据]`、`[评测]`、`[修复]`、`[指标]`、`[提交]` 等阶段标签。
12. 重试时可以自动清理 repo 内部临时缓存，例如 `*.incomplete`、`*.tmp`、`__pycache__/`；但未经明确批准，绝不能删除源码、checkpoint、数据集、提交文件、conversation history 或 workspace 外部路径。

结束时报告：

- 复现指标表格路径
- 原始结果或日志路径
- checkpoint 和数据集来源
- 人类交互轮次估计
- 如果已知，报告最小成功基座模型

## 仓库任务卡：MuQ-Eval

用尽可能少的人类交互复现指定论文结果。

仓库信息：

- Repository: `https://github.com/dgtql/MuQ-Eval`
- Local path: `D:\vLLM1\model\MuQ-Eval`
- Paper: `https://arxiv.org/abs/2603.22677`

目标：

- 使用作者提供的模型，运行一轮评测，复现两个高亮指标。
- Benchmark: `MusicEval`
- 推荐模型/checkpoint: `zhudi2825/MuQ-Eval-A1`

要求设置：

```json
{"method": "A1 (Frozen+MSE) [recommended]"}
```

执行要求：

1. 在运行昂贵命令前，先检查仓库 README 和脚本。
2. 创建或复用隔离环境，并记录准确命令。
3. 优先使用仓库官方评测脚本：

```powershell
python run_evaluation.py --checkpoint-dir outputs/A1_frozen_mlp --config configs/A1_frozen_mlp.yaml --all-folds --bootstrap
```

4. 将原始日志和结果文件保存到 `outputs/results` 下。
5. 只生成提交所需的复现指标表格。
6. 保留完整 conversation history。
7. 如果依赖、数据集、checkpoint、GPU 或 API key 缺失，报告最小具体阻塞点，以及继续所需的命令或路径。

提交文件夹：

```text
MuQ-Eval
```

提交表格列：

```text
Method, System SRCC, Utterance SRCC
```

## Skill：gpu-and-checkpoint-handling

### 目标

在复现过程中处理 GPU 选择、checkpoint 准备、数据集缓存和显存降级。

### 预检查

检查：

- `nvidia-smi`
- Python 和 CUDA 版本
- PyTorch CUDA 是否可用
- 可用磁盘空间
- HuggingFace 缓存和代理配置
- 必需数据集路径
- 必需 checkpoint 路径

### Checkpoint 规则

- 优先使用作者提供的 checkpoint，而不是重新训练。
- 将 checkpoint 下载到官方脚本期望的位置。
- 如果官方脚本需要 fold 目录，只在合理情况下创建最小匹配结构。
- 除非作业明确允许，不要静默地把一个 fold checkpoint 复制成所有 fold。
- 在 conversation history 中记录 checkpoint 来源和文件名。

### GPU 规则

- 优先使用能满足需求的最小 GPU。
- 如果 CUDA OOM，尝试降低 batch size、运行单 fold/单配置、CPU smoke test 或切换 GPU。
- 对 Windows 环境，CUDA-heavy repo 优先使用 Linux 服务器或 WSL2。
- 将 `flash-attn` 和自定义 CUDA 构建失败视为环境阻塞，而不是模型失败。

### 网络规则

- 可用时优先使用本地缓存。
- 如果 HuggingFace 下载失败，报告准确的模型或数据集 ID。
- 不要把 API key 或 access token 写入 prompt、源码或提交物。

### 清理策略

为了减少不必要的人类交互，agent 可以自动删除临时文件，但必须同时满足：

- 路径在当前仓库或配置的 workspace 内。
- 文件明显是临时、未完成或缓存文件，例如：
  - `*.tmp`
  - `*.incomplete`
  - `__pycache__/`
  - `.pytest_cache/`
  - `outputs/`、`results/`、`cache/` 下的 repo-local HuggingFace cache
- 删除是为了重试失败下载、重建损坏缓存或清理部分产物。

以下情况仍必须询问用户：

- 源码、配置、脚本、README
- checkpoint，例如 `*.pt`、`*.pth`、`*.ckpt`、`*.safetensors`
- 数据集、标注、图片、音频或 benchmark 原始文件
- 提交文件或 `conversation_history`
- workspace 外部路径
- `outputs/`、`results/`、`cache/`、`__pycache__/`、`.pytest_cache/` 之外的递归删除

自动清理时，用状态更新而不是提问：

```text
[清理] 删除 repo 内失败下载的 .incomplete 临时文件；这是 HuggingFace 缓存残留；下一步重新探测数据集。
```

## Skill：low-interaction-agent

### 目标

在保证复现正确、过程可审计的前提下，最小化人类与 agent 的交互轮次。

### 交互计数

- 初始任务 prompt 计为 1 次用户交互。
- 用户提供的纠正、路径修复、GPU 切换和人工决策计为额外交互。
- agent 工具调用、自动探索、日志分析、自修复不计为人类交互。
- 除非没有安全默认值，否则应避免澄清问题。

### 行为规则

1. 从扫描 repo 本地文件和官方文档开始。
2. 对明显问题使用安全默认值，不要直接询问用户。
3. 优先使用 workspace 中发现的本地路径，再向用户询问路径。
4. 失败后尝试有边界的修复循环。
5. 只有下一步需要缺失的数据、凭证、硬件或用户策略决策时才停止。
6. 在长时间阶段前后提供简短进度反馈，让用户知道当前发生了什么。

### 进度反馈

agent 应在执行过程中输出简短状态信息。这些状态不是澄清问题，不需要用户操作。

格式：

```text
[阶段] 当前动作；原因；预计下一步。
```

推荐阶段：

- `[扫描]` 正在读取 README、脚本、配置和 task card。
- `[环境]` 正在检查 Python、CUDA、GPU、磁盘、代理和缓存路径。
- `[依赖]` 正在安装或验证必要包。
- `[模型]` 正在检查或下载 checkpoint。
- `[数据]` 正在检查或下载数据集。
- `[试跑]` 正在运行最小 smoke test。
- `[评测]` 正在运行目标 benchmark 命令。
- `[修复]` 出错后正在做最小兼容修复。
- `[指标]` 正在从日志或结果文件中抽取所需指标。
- `[提交]` 正在写 `reproduction.md` 并准备 conversation history。

示例：

```text
[模型] 正在检查 MuQ-Eval A1 checkpoint；评测脚本需要 best_model.pt；下一步会对齐到 outputs/A1_frozen_mlp/fold0/。
[数据] 正在探测 BAAI/MusicEval 字段名；System SRCC 需要真实 model_id；下一步会确认分组字段。
[修复] 发现 run_evaluation.py 未合并 base.yaml；这是脚本接口问题；下一步做最小补丁后重跑同一命令。
```

### 安全默认值

- 使用以 repo 命名的新隔离环境。
- CPU 仅用于 smoke test；如果 CUDA 可用，目标评测使用 CUDA。
- README 提供 checkpoint 时，优先使用预训练 checkpoint。
- 先运行一个目标配置，再考虑 full sweep。
- 输出保存到 `outputs/`、`results/` 或 repo-local run 目录。

### 失败循环

每次失败：

1. 阅读完整 traceback 或日志。
2. 分类失败原因：依赖、配置、路径、数据集、checkpoint、GPU 显存、网络或代码不匹配。
3. 应用最小修复。
4. 重跑能到达失败点的最小命令。
5. 重复失败后，总结阻塞点和下一步准确动作。

## Skill：metric-extraction

### 目标

只抽取所需复现指标，并写入官方 Markdown 表格。

### 输入

- 目标指标名和行
- logs、JSON、CSV、stdout 或 README 参考值
- 提交文件夹路径

### 规则

- 最终 `reproduction.md` 必须只包含 Markdown 表格。
- 必须包含行名和列名。
- 不要在 `reproduction.md` 中写命令、解释、注释、截图或来源说明。
- 来源追溯保存在日志和 conversation history 中，而不是最终表格。
- 如果存在多个 run，使用匹配要求模型、benchmark 和 setting 的 run。

### 表格命名

使用作业示例要求的文件名：

```text
reproduction.md
```

除非老师明确改变格式，不要改名为 `result.md`。

### 验证

最终打包前：

1. 确认每个单元格都来自匹配 run 或官方目标参考。
2. 确认没有占位符。
3. 确认表格只有要求的指标列。
4. 确认对应原始输出保存在最终表格之外。

## Skill：paper-repo-reproduction

### 目标

以最少人类交互复现科研仓库中的指定结果。输出必须是可追溯的指标表格和完整 conversation history。

### 输入

- 仓库路径和 URL
- 论文或表格目标
- 需要的 benchmark、模型、checkpoint 和数据集
- 硬件和 API 约束
- 提交表格需要的列

### 工作流

1. 在提问前阅读 `README`、脚本、配置、示例和论文表格线索。
2. 找到官方 evaluation 入口和最小目标配置。
3. 检查环境要求：Python、CUDA、PyTorch、`flash-attn`、数据集、checkpoint、磁盘、GPU 显存和代理。
4. 创建隔离 conda 或 venv 环境。
5. 在昂贵评测前运行最小 smoke test。
6. 除非作业明确要求 full sweep，否则只运行目标 benchmark/setting。
7. 保存原始日志、命令、配置、结果 JSON/CSV 和代码补丁。
8. 只抽取所需指标。
9. 写一个只包含必要行列的 Markdown 表格。
10. 完整保留 conversation history，不删除、不重写。

### 决策规则

- 优先使用官方脚本，而不是重写 evaluation 逻辑。
- 作业要求使用作者模型时，优先使用 pretrained checkpoint 而不是重新训练。
- 只为兼容性 bug 或脚本接口错误打补丁，并在 conversation history 中记录补丁。
- 如果缺少数据集、checkpoint、GPU 或 API key，报告准确阻塞点和最小下一步。
- 不要把秘密写入 prompt、代码、提交文件、结果表或报告。

### 完成标准

- 指标表可以追溯到日志或结果文件。
- 提交 Markdown 文件只包含目标表格。
- conversation history 能证明环境准备、执行、失败、修复和指标抽取全过程。
