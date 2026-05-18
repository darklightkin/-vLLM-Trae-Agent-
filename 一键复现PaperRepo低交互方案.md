# 一键复现 Paper Repo 的低交互方案

## Summary

目标从“环境自动配置工作流”调整为：用 vibe coding 工具在尽量少的人类交互轮次下，完成 3 个指定 repo 的指标复现，并提交每仓库 2 类文件：精度表格 md 和完整 conversation_history。

默认路线：使用成熟 Claude/Codex 类 agent 工具，配一套自定义 prompt + skill pack；模型策略采用“小到大阶梯”：先用便宜/小模型跑，失败再升到 Sonnet/Opus，并在 PPT 中记录“最小成功基座模型”。内部保留日志、脚本、指标解析器和 case notes，但最终按官方要求只提交表格和 conversation history。

方案借鉴 MLEvolve 的 experience-driven memory 思想：不是把所有运行日志无限追加进 prompt，而是让 agent 在真实执行、失败修复和指标验证中产生经验摘要，再由人工审核压缩为最终版 final skill pack。最终 skill 是“人工先验 + agent 执行反馈 + 日志复盘 + 经验压缩”的结果，用来帮助后续 repo 复现减少交互轮次。

## Key Design

核心评价口径：

- `a=1` 表示精度/速度复现达标；
- 交互轮次按“用户主动消息数”统计；
- 初始总 prompt 计 1；
- 后续人工纠偏、换 GPU、补路径计新增轮次；
- agent 主动探索、工具调用、自动修复不计。

方案不再突出多 Agent 系统，而是突出“一条强 prompt + 可复用 skill + 自动闭环执行”：

- repo scan；
- 环境创建；
- 数据/模型准备；
- 运行复现；
- 指标解析；
- 失败修复；
- 生成提交表格。

其他设计原则：

- 不把敏感 API key 写入 prompt、repo、history 清单或报告正文；
- API key 只使用本地环境变量或工具配置；
- conversation history 必须完整保存，不做删改；
- 内部辅助日志可以留在工作目录；
- 最终 zip 只放官方要求文件；
- 参考 SUPER 和 ML-Bench 的动机：科研 repo 复现难点在 setup + execution；
- 参考 MLEvolve 的动机：通过执行反馈不断演化方案，并把高价值经验沉淀为 memory；
- 本方案优化点是低交互、低模型成本、可复用 prompt/skill。

## Overall Workflow

整体流程分为 6 步：

```text
v0 skill + task card
  -> agent 一键复现 repo
  -> 保存 conversation history 和原始日志
  -> agent 总结 case note
  -> 人工审核并压缩经验
  -> 更新 final skill pack
```

每个 repo 的执行闭环：

```text
Repo Scan
  -> Task / Env Hypothesis
  -> Install & Run Plan
  -> Execute
  -> Verify Metrics
  -> Reflect / Repair
  -> Case Note
  -> Skill Update
```

其中：

- `conversation_history` 是原始证据，必须完整保存，不删改；
- `case note` 是每个 repo 的复现经验摘要，内部使用；
- `final skill pack` 是从多个 case note 中压缩出的通用经验，最终用于展示和复用。

经验进入 final skill 的标准：

- 在多个 repo 中重复出现；
- 能显著减少人类交互；
- 能避免高成本失败，例如 OOM、重复下载、模型路径错误；
- 会影响最终提交格式或指标可追溯性；
- 是下次 agent 必须默认知道的规则。

不进入 final skill 的内容：

- 某次运行的临时目录；
- 某一块 GPU 的临时编号；
- 偶发网络慢或一次性路径错误；
- 已被后续规则覆盖的低价值失败记录。

## Prompt And Skills

准备一个通用总 prompt，要求 agent：

1. 不要先问用户，先自动阅读 README、scripts、configs、issues、paper/table 线索；
2. 必须创建隔离 conda/venv 环境；
3. 自动检测 GPU、CUDA、磁盘、代理、HuggingFace 缓存；
4. 优先执行最小可验证复现，再跑目标指标；
5. 失败时根据日志自修复，最多连续修复 N 轮；
6. 最后只生成官方要求的精度表格 md，并整理 conversation history。

准备 4 个 skill：

- `paper-repo-reproduction`：科研仓库复现流程；
- `low-interaction-agent`：减少澄清问题，安全默认决策；
- `metric-extraction`：从 logs/json/csv 中抽取红圈指标并生成只含表格的 md；
- `gpu-and-checkpoint-handling`：GPU 选择、模型下载、断点缓存、显存不足降级。

### Skill 能力描述

这些 skill 的目的不是增加系统复杂度，而是把人工复现经验提前固化为 agent 的默认行为，减少后续人类提醒和交互轮次。

#### `paper-repo-reproduction`

能力描述：

- 指导 agent 按科研 repo 的标准流程做复现；
- 先阅读 README、依赖文件、scripts、configs、examples、eval、tests；
- 推断 Python/CUDA/PyTorch 版本、模型 checkpoint、数据集、评测入口和目标指标；
- 先跑 import check、help check、smoke test，再跑完整目标评测；
- 保留安装命令、运行命令、日志和结果文件路径。

目的：

- 防止 agent 一上来直接 `pip install` 或盲目运行 demo；
- 让 agent 先理解仓库结构和目标任务；
- 提高第一次 prompt 跑通复现的概率。

#### `low-interaction-agent`

能力描述：

- 规定 agent 默认自主决策，不轻易向用户提问；
- 只有遇到缺少私有凭据、缺少私有数据/模型路径、GPU 分配冲突、危险命令或系统级操作时才问用户；
- 普通依赖报错、脚本参数错误、路径错误、CUDA/PyTorch 不匹配等问题，先读日志并自动修复；
- 每次失败后先分类原因，再修改安装或运行方案继续尝试。

目的：

- 直接服务评分中的“人和 vibe coding 交互轮次越少越好”；
- 避免 agent 反复询问“是否继续安装”“是否修复报错”“是否生成结果文件”；
- 把用户介入限制在真正需要人工确认的场景。

#### `metric-extraction`

能力描述：

- 从 stdout、log、json、csv、txt 或评测输出目录中定位目标指标；
- 只抽取任务要求的红圈指标；
- 保留原始行名和列名；
- 生成只包含 Markdown 表格的 `result.md`；
- 将命令、日志路径、解释说明放在内部记录中，不写入 `result.md`。

目的：

- 避免最终提交文件格式错误；
- 保证 `result.md` 满足“只包含表格”的要求；
- 让指标可以从原始日志或结果文件追溯，方便答辩和核查。

#### `gpu-and-checkpoint-handling`

能力描述：

- 运行重任务前检查 `nvidia-smi`、CUDA 版本、显存、磁盘空间和当前进程占用；
- 检查 HuggingFace 缓存、模型路径、数据路径和 checkpoint 完整性；
- 对大文件下载使用可恢复策略，避免重复下载；
- 遇到 OOM 时优先尝试换 GPU、降低 batch size、缩小样本数或使用官方轻量配置；
- 记录模型、数据和 checkpoint 的来源、路径和校验状态。

目的：

- 针对 GUI-KV、GUI-Actor 这类大模型 GUI benchmark 的主要风险；
- 减少因为 GPU、checkpoint、数据路径导致的无效运行；
- 降低 API token 和服务器时间浪费。

每个 repo 使用同一个总 prompt，加一个 repo task card，避免每次重新解释规则。

## Task Card And Prompt Usage

`skill` 和 `task card` 分工不同：

- `skill` 说明怎么复现：工作流、安全边界、失败修复、指标抽取；
- `task card` 说明复现什么：repo、目标表格、模型、数据集、指标、误差和输出格式；
- `prompt` 负责调用 skill，并附上当前 repo 的 task card。

支持本地 skill 机制的工具中，prompt 只需要引用 skill 名称并附 task card：

```text
Use these skills:
- paper-repo-reproduction
- low-interaction-agent
- metric-extraction
- gpu-and-checkpoint-handling

Task card:
Repository: ...
Goal: ...
Metrics: ...
Output: result.md only contains the metric table.
```

不支持本地 skill 的工具中，把 skill 核心规则以内联 prompt 形式提供，保证不同 vibe coding 工具都能使用同一套方法。

task card 模板：

```text
Repository:
<repo url>

Goal:
<which table / benchmark / setting to reproduce>

Model:
<required model or author-provided model>

Dataset / Benchmark:
<dataset or benchmark name>

Required Metrics:
<metric names>

Tolerance:
<allowed deviation or recording rule>

Output:
Create result.md with only the final Markdown table.
Keep original row names and column names when possible.
```

## Execution Phases

### Phase 0: 准备 v0 skill 和 task card

先人工写一版粗 skill，不追求完美，只固化最重要的默认行为：

- 少问人；
- 建隔离环境；
- 先读仓库再安装；
- 失败先读日志并修复；
- 最终只输出指标表格。

同时准备 3 个 task card：

```text
tasks/MuQ-Eval.md
tasks/GUI-Actor.md
tasks/GUI-KV.md
```

### Phase 1: MuQ-Eval 首次闭环

优先从 MuQ-Eval 开始，因为任务相对轻、指标明确、适合打通完整流程。

第一轮建议用较强模型，例如 `claude-sonnet-4-6-thinking` 或同档模型，目标不是省钱，而是拿到第一条成功路径：

- 环境如何创建；
- 依赖如何安装；
- 作者模型和数据如何准备；
- 评测命令是什么；
- 指标从哪个文件抽取；
- 哪些失败需要写进 skill。

完成后生成：

```text
runs/MuQ-Eval/result.md
runs/MuQ-Eval/conversation_history/
runs/MuQ-Eval/case_note.md
```

### Phase 2: 更新 skill 并迁移到 GUI-Actor

根据 MuQ-Eval 的 conversation history、日志和 case note，更新 skill。

然后用更新后的 skill 跑 GUI-Actor，重点验证：

- 大模型 checkpoint 处理；
- ScreenSpot-Pro 数据和脚本入口；
- GPU 显存和 batch 配置；
- GUI benchmark 指标抽取。

完成后生成 GUI-Actor 的 `result.md`、`conversation_history` 和 `case_note.md`，并再次压缩经验进入 final skill pack。

### Phase 3: 跑 GUI-KV

最后跑 GUI-KV，因为它包含 UI-TARS-1.5-7B、AgentNetBench、`budget=80%` 和 `budget=10%` 两个 setting，复杂度最高。

重点验证：

- `transformers` 指定 commit；
- CUDA 12.x 和 `flash-attn`；
- `AGENTNETBENCH_IMGS` 和 `AGENTNETBENCH_DATA`；
- 两个 budget setting 的结果抽取；
- 精度是否在 15% 浮动范围内。

### Phase 4: 小模型复跑和汇报整理

三仓库跑通后，用 final skill pack 尝试小模型复跑或局部复跑，记录：

- 最小成功模型；
- token / cost；
- 人类主动交互轮次；
- 失败修复次数；
- 是否达到目标指标。

这部分用于 PPT 加分展示，不影响官方最终提交格式。



## Repo Execution Plan

### GUI-KV

目标为 `UI-TARS-1.5-7B`，AgentNetBench，`budget=80%` 和 `budget=10%`，精度允许上下浮动 15%。

优先运行 repo 提供的 `eval/agentnetbench_eval.py` 单配置命令，再扩展到两个 budget。

注意事项：

- `transformers` 指定 commit；
- CUDA 12.x；
- `flash-attn`；
- `AGENTNETBENCH_IMGS`；
- `AGENTNETBENCH_DATA`。

### MuQ-Eval

使用作者提供模型，跑一轮即可。

优先复现 README 中 A1 推荐行，记录：

- `System SRCC`；
- `Utterance SRCC`。

如果全 5-fold 成本太高，先按老师要求的一轮复现输出目标指标，并保留运行依据。

### GUI-Actor

目标为 `ScreenSpot-Pro with GUI-Actor-7B`。

优先使用官方 `eval/screenSpot_pro.py`，模型选择和表格列以实际运行结果记录。

README 中 GUI-Actor-7B 可参考：

- Qwen2-VL：40.7；
- Qwen2.5-VL：44.6。

最终以运行输出生成提交表。

每个 repo 结束后只提交：

```text
gui-kv/result.md + conversation_history/
MuQ-Eval/result.md + conversation_history/
GUI-Actor/result.md + conversation_history/
```

然后三个文件夹统一打 zip。

## Model And Cost Strategy

- 第 0 轮：用小模型生成/检查 task card 和 prompt，不跑重任务。
- 第 1 轮：用 `claude-haiku-4-5-thinking` 或同档小模型尝试一键复现。
- 第 2 轮：若因仓库理解、修复能力不足失败，升级 `claude-sonnet-4-6-thinking`。
- 第 3 轮：只在关键仓库多次失败时用 Opus 级模型救场。

PPT 汇报记录每个 repo：

- 最小成功模型；
- token/cost；
- 交互轮次；
- 是否达标；
- 失败修复次数。

## Test And Acceptance

每个 repo 的 `result.md` 只包含目标指标表格，保留行名和列名，不写过程说明。

每个 conversation history 能证明：

- 初始 prompt；
- agent 自动探索；
- 环境创建；
- 复现运行；
- 失败修复；
- 指标提取全过程完整存在。

内部验收标准：

- 指标表可从原始日志或结果文件追溯；
- 环境不污染系统 Python；
- 没有 Docker/sudo/危险删除；
- 人类主动交互轮次尽量保持 1；
- 最多接受“换 GPU/补路径”这类必要交互。

最终展示时强调：

`Final_score = a * (1+b)` 的核心是先保证 `a=1`，再通过 prompt 和 skill 降低交互轮次；小模型使用作为额外加分项展示，不计入 final_score。
