# MuQ-Eval v2 低交互复现 Prompt

把本文全文交给 Codex、Claude Code、Trae、Cursor 等 agent 工具执行。

## 任务目标

请从零开始复现 MuQ-Eval 的 A1 推荐模型评测结果，并尽量减少人类交互轮次。

- Workspace: `D:\vLLM1`
- GitHub: `https://github.com/dgtql/MuQ-Eval`
- 默认本地仓库: `D:\vLLM1\model\MuQ-Eval`
- Paper: `https://arxiv.org/abs/2603.22677`
- Benchmark: `MusicEval`
- 目标方法: `A1 (Frozen+MSE) [recommended]`
- 目标指标: `System SRCC`、`Utterance SRCC`

假设本地没有代码仓库、没有数据集、没有 checkpoint。你需要自己 clone 仓库，读取官方 README 和配置，确认数据集、模型和 checkpoint 地址，然后自动下载所需资源。

## 必读本地 Skills

执行前先读取这些本地 skill 文件，不要把 skill 全文复制进 prompt：

```text
D:\vLLM1\skills\paper-repo-reproduction.md
D:\vLLM1\skills\low-interaction-agent.md
D:\vLLM1\skills\metric-extraction.md
D:\vLLM1\skills\gpu-and-checkpoint-handling.md
```
paper-repo-reproduction.md：负责科研论文仓库的标准复现流程。
low-interaction-agent.md：负责减少人工提问，让 agent 自动决策和持续反馈。
metric-extraction.md：负责从日志或结果文件中提取指标并生成表格。
gpu-and-checkpoint-handling.md：负责处理 GPU、模型权重、数据下载和缓存问题。


简要用途：

- `paper-repo-reproduction.md`：定义科研 repo 复现流程，包括读 README、找入口、建环境、跑评测、保存日志和输出表格。
- `low-interaction-agent.md`：要求 agent 少问用户，优先自动探索、安全默认决策，并在长任务中输出进度。
- `metric-extraction.md`：从日志、JSON、CSV 或 stdout 中抽取目标指标，生成只含表格的 `reproduction.md`。
- `gpu-and-checkpoint-handling.md`：处理 GPU、CUDA、checkpoint、HuggingFace cache、数据下载、显存不足和临时缓存清理。

## 从零启动命令

先执行以下命令下载官方仓库：


如果 `D:\vLLM1\model\MuQ-Eval` 已存在，不要覆盖，先检查：

进入仓库后，必须先读取官方文档和配置，以当前仓库内容为准：

```powershell
Get-Content README.md
Get-Content requirements.txt
Get-Content configs\A1_frozen_mlp.yaml
Get-Content configs\base.yaml
```

根据当前 README，预期资源通常是：

- Dataset: `BAAI/MusicEval`
- Author checkpoint: `zhudi2825/MuQ-Eval-A1`
- Encoder/model: `OpenMuQ/MuQ-large-msd-iter`

如果 README 或配置中的资源 ID 与上面不同，以官方仓库当前内容为准，并记录到 history。

## 强约束

### 1. 禁止修改原仓库代码

不得修改 `D:\vLLM1\model\MuQ-Eval` 下的源码、配置、脚本或 README，包括但不限于：

```text
run_evaluation.py
train.py
src/*.py
configs/*.yaml
README.md
requirements.txt
```

如果官方脚本有 bug 或接口不兼容，优先使用以下方式处理：

- 在 `D:\vLLM1\scripts\` 下写 wrapper 脚本；
- 使用命令行参数、环境变量、临时 copy、monkey-patch wrapper；
- 将问题和阻塞点写入 history；
- 如果必须修改原仓库才能继续，停止并报告原因，不要自行 patch 原仓库。

### 2. 不要先问用户

先自动完成仓库扫描、环境检查、依赖确认、数据和 checkpoint 探测。只有遇到缺失权限、访问受限数据、必须提供 token、必须切换机器/GPU、或要删除非临时文件时才询问用户。

### 3. 数据和模型下载

使用 HuggingFace 官方接口下载或缓存数据集和模型。不要伪造数据集，不要把 README 参考值当作本地复现结果。

将 HuggingFace cache 放到 workspace 或 repo-local 目录，例如：

```powershell
$env:HF_HOME='D:\vLLM1\model\MuQ-Eval\outputs\results\hf_cache'
$env:HF_HUB_CACHE='D:\vLLM1\model\MuQ-Eval\outputs\results\hf_cache\hub'
$env:HF_DATASETS_CACHE='D:\vLLM1\model\MuQ-Eval\outputs\results\hf_cache\datasets'
```

可以自动清理 repo-local cache 中的 `*.incomplete`、`*.tmp`、`__pycache__/` 等临时文件。不得删除 checkpoint、原始数据集文件、源码、提交文件或 conversation history。

### 4. 运行策略

优先运行官方 README 推荐的 A1 评测命令。

如果 5-fold checkpoint 不完整，先运行最小可验证评测，例如单 fold，并在 history 中说明原因。

如果 CUDA 不可用，可以用 CPU 做 smoke test 或单 fold 评测，但必须记录：

```text
CUDA unavailable
CPU-only evaluation
```

如果 CPU 评测耗时过长，停止长任务，改为最小样本 smoke test，并说明完整复现需要 GPU 或 Linux 环境。

### 5. 及时反馈

长任务过程中输出简短状态，不要让用户不知道当前在做什么。格式示例：

```text
[扫描] 正在读取 README 和配置；需要确认官方入口；下一步检查环境。
[模型] 正在下载作者 checkpoint；评测需要 best_model.pt；下一步对齐官方路径。
[数据] 正在下载 BAAI/MusicEval；指标必须来自真实数据；下一步运行评测。
[指标] 正在解析 JSON 结果；需要生成最终表格；下一步写 reproduction.md。
```

## 输出要求

### 1. 最终表格

写入：

```text
D:\vLLM1\submission\MuQ-Eval\reproduction.md
```

文件只能包含 Markdown 表格，不写解释：

```markdown
| Method | System SRCC | Utterance SRCC |
| --- | --- | --- |
| A1 (Frozen+MSE) [recommended] | <value> | <value> |
```

### 2. 复现历史

写入：

```text
D:\vLLM1\submission\MuQ-Eval\conversation_history\history.md
```

history 必须记录：

- 初始任务和 prompt 来源；
- 读取过的 skill 文件；
- clone 仓库、环境检查、依赖检查命令；
- 数据集下载方式、资源 ID 和 cache 路径；
- checkpoint 来源和本地路径；
- 实际执行的评测命令；
- 原始日志路径和 JSON 结果路径；
- 是否出错、错误原因和处理方式；
- 是否修改原仓库代码，必须明确写 `否`；
- 如果不能完成 5-fold，说明缺少什么；
- 最终提取的 `System SRCC` 和 `Utterance SRCC`；
- 人类交互轮次估计。

