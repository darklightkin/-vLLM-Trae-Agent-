# reproduction-master v2

## 目标

作为总控 skill，负责把论文仓库复现任务拆成可审计的阶段，并按需调用小 skill。核心目标是：少交互、可追溯、不伪造、不污染原仓库。

## 总分结构

总控只负责阶段、门禁、路由和最终验收；细节交给外接小 skill：

- `paper-repo-reproduction.md`：仓库扫描、官方入口识别、复现流程。
- `low-interaction-agent.md`：低交互策略、进度反馈、停止条件。
- `gpu-and-checkpoint-handling.md`：GPU、CUDA、checkpoint、HuggingFace cache 和数据下载。
- `metric-extraction.md`：指标抽取、表格生成、结果校验。

执行时先读本文件，再按阶段读取对应小 skill。不要一次性把所有 skill 全部展开进 prompt。

## 阶段流程

### 1. Intake

默认 prompt 会提供 GitHub URL。把 GitHub URL 作为任务主入口，本地路径只作为可选覆盖项。

确认任务卡中至少包含：

- GitHub URL；
- 目标 benchmark；
- 目标模型或方法；
- 目标指标；
- 提交文件格式；
- 是否允许修改原仓库代码。

如果没有提供本地路径，根据 GitHub repo 名自动推导：

```text
<workspace>/model/<repo-name>
```

例如：

```text
GitHub: https://github.com/dgtql/MuQ-Eval
Local repo: D:\vLLM1\model\MuQ-Eval
```

如果缺 benchmark、dataset、checkpoint、metric 等字段，先 clone/读取 README、paper、configs、scripts 自动补齐；只有无法推断时才问用户。

### 2. Repo Bootstrap

从 GitHub URL 启动仓库准备：

1. 解析 repo 名，推导默认本地目录。
2. 如果本地目录不存在，从 GitHub clone。
3. 如果本地目录已存在，先检查 `git status --short` 和 `git remote -v`。
4. 如果 existing remote 与 prompt 的 GitHub URL 不匹配，不要覆盖，停止并报告路径冲突。
5. 如果 remote 匹配，继续使用已有目录，不要删除用户改动。

推荐命令形态：

```powershell
cd <workspace>
New-Item -ItemType Directory -Force model | Out-Null
git clone <github-url> model\<repo-name>
cd model\<repo-name>
```

已存在时：

```powershell
cd <workspace>\model\<repo-name>
git status --short
git remote -v
```

输出一个最小 inventory：

| Component | Path / Source | Status |
| --- | --- | --- |
| README |  |  |
| requirements |  |  |
| configs |  |  |
| eval entrypoint |  |  |
| dataset |  |  |
| checkpoint |  |  |
| result output |  |  |

### 3. Environment Gate

先做环境门禁，再跑重任务：

- Python/conda/venv 是否可用；
- PyTorch 和 CUDA 是否可用；
- `nvidia-smi` 是否可用；
- 磁盘空间是否足够；
- 代理和 HuggingFace cache 是否合理；
- 关键包能否 import。

环境不满足时，先给出最小修复动作；不要直接开始长评测。

### 4. Resource Gate

确认数据集和 checkpoint 来自官方 README、paper 或模型仓库。优先使用 HuggingFace 官方接口和 repo-local cache。

必须记录：

- dataset ID、split、cache 路径；
- checkpoint repo/file、下载路径；
- encoder/backbone 模型 ID；
- 是否需要 token；
- 是否只拿到部分 fold。

### 5. Smoke Gate

先做最小可验证运行，再做目标复现。smoke 可以是：

- import test；
- dataset `train[:1]` 或最小 split 加载；
- checkpoint load；
- 单 batch forward；
- 单 fold 或单配置评测。

smoke 失败时，进入 bounded repair loop；不要跳到全量运行。

### 6. Target Run

优先运行官方 README 推荐命令。只跑任务要求的模型、benchmark 和 setting，除非任务明确要求 full sweep。

如果完整复现缺硬件、数据、checkpoint 或权限，执行最小可验证替代 run，并在 history 中明确写阻塞原因和缺失项。

### 7. Metric Extraction

从真实 logs、JSON、CSV 或 stdout 抽取指标。最终提交表只保留指定表格，不写过程说明。

严禁把 README 参考值伪装成本地运行结果。

### 8. History and Acceptance

必须生成 conversation history，并包含：

- 初始 prompt 和 task card；
- 读取过的 skill；
- 实际命令和关键输出；
- 数据集/checkpoint 来源；
- 错误、修复和重跑过程；
- 是否修改原仓库代码；
- 最终指标和原始结果路径；
- 人类交互轮次估计。

## 关键门禁

进入下一阶段前必须满足：

- Repo bootstrap 完成后：知道官方 eval entrypoint。
- Environment gate 完成后：知道用 CPU 还是 CUDA。
- Resource gate 完成后：知道数据和 checkpoint 是否完整。
- Smoke gate 完成后：至少有一个最小命令跑通。
- Target run 完成后：指标可追溯到原始文件。

## 安全策略

- 默认不修改原仓库代码；若任务允许修改，也必须记录 diff 和原因。
- 禁止删除源码、配置、checkpoint、原始数据、提交文件和 conversation history。
- 可以自动清理 workspace 内明显临时的失败缓存，例如 `*.tmp`、`*.incomplete`、`__pycache__/`。
- 不把 API key、HF token、私有路径凭证写入 prompt、代码、表格或 history。

## 输出

每个 repo 最终至少生成：

```text
submission/<repo>/reproduction.md
submission/<repo>/conversation_history/history.md
```

`reproduction.md` 只放表格；过程、命令、错误、来源全部写入 history。
