---
name: reproduce-repo
description: 当用户提供 GitHub 仓库 URL、本地仓库路径、论文仓库、benchmark 仓库、离线复现包，或要求以低交互方式复现论文、表格、benchmark 结果时使用。本 skill 覆盖仓库扫描、目标指标识别、全新隔离 conda 或 venv 环境搭建、依赖修复、checkpoint 优先的评估或推理复现、最小 dataset 处理、按阶段与仓库规定版本进行 Verify、smoke test、有边界的失败恢复、指标来源校验、固定 result.md 生成、原始 Codex history.md 保存，以及用独立 change_summary.md 记录智能体改动和 Verify 对比过程。遇到缺失依赖、Python 或 torch 环境损坏、CUDA OOM、checkpoint 布局不匹配、dataset schema 不匹配、缺少指标文件、官方脚本接口漂移、最终提交结果不明确等失败模式时触发本 skill。
---

# Reproduce Repo

## 目的

以最少的人类交互和完整可审计性，从一个仓库中复现目标论文、表格或 benchmark 结果。

这是主 skill。它应保持为可触发的入口。只有在对应阶段需要时，才加载 reference 文件。

## 何时使用

当 YAML `description` 触发后，如果用户：

- 提供 GitHub URL 或本地仓库路径；
- 指向一个离线论文仓库包；
- 要求复现 benchmark、表格、结果、模型行或 README 指标；
- 要求生成 `result.md`、原始 `history.md`，或进行低交互复现；
- 粘贴失败复现日志并希望智能体继续处理；

则使用本 skill。

除非属于复现工作流的一部分，否则不要把本 skill 用于无关代码审查或普通功能开发。

## 输入

预期输入：

- GitHub URL 或本地 repo 路径；
- 论文 URL 或目标表格/README 行；
- benchmark 名称；
- model/method/checkpoint 目标；
- dataset 目标或离线资源路径；
- 目标指标和可接受误差；
- 硬件约束；
- 是否允许修改源代码；
- 官方提交结构。

如果输入缺失，先从 README、configs、scripts、paper links、model cards、issues、本地资源和 task cards 中推断，再向用户提问。

## 输出

最终提交目录只能包含：

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

`result.md` 只包含指标表。`history.md` 只包含原始 Codex 对话轨迹：用户消息、assistant 回复、工具调用、命令执行记录、错误、修复和指标提取过程。`change_summary.md` 是独立的简洁总结，记录智能体所做改动，以及每一阶段与仓库规定版本、配置、datasets、checkpoints、scripts、model names 的 Verify/对比过程。原始 logs、JSON、CSV、checkpoints、caches、outputs 和 wrapper scripts 可以留在工作目录用于追踪，但除非用户明确要求，不要放进最终提交目录。

## 低交互规则

- 用户交互轮数按用户主动发送给 Codex 的消息计算。
- 智能体扫描、shell 命令、进度更新、日志分析和自动修复不计为人类交互轮数。
- 只有在需要 credentials、私有数据许可、付费下载、大 dataset 下载、长时间训练、硬件/机器切换、系统级修改、危险删除、被禁止的源码编辑批准，或非临时删除时，才询问用户。
- 每个不可避免的问题都记录为 `interaction_count_candidate`。
- 优先采用安全默认值；能推断时自动执行；让原始 `history.md` 轨迹展示决策过程。

## 执行工作流

按以下顺序执行：

1. repo scan；
2. Verify 仓库声明的版本、必需文件、官方脚本、model names 和目标说明；
3. 识别 paper/table/benchmark 目标；
4. 按官方 repo/paper/task-card 要求 Verify 目标行、metric names、split/fold 协议和可接受误差；
5. 创建全新隔离环境；
6. 安装依赖；
7. 按仓库要求 Verify Python、CUDA、PyTorch、package versions 和 command entrypoints；
8. 发现/下载作者 checkpoint 或 pretrained weight；
9. 按仓库要求 Verify checkpoint source、filename、size/hash、fold layout 和 model name；
10. dataset discovery/download 或绑定本地 dataset；
11. 按仓库要求 Verify dataset ID/path、version、split、schema、sample count 和 preprocessing；
12. 规划 checkpoint-based evaluation 或 inference reproduction；
13. 运行最小 evaluation subset 或 smoke test；
14. Verify smoke output、metric keys 和预期 artifact locations；
15. 如果 gates 通过，运行 benchmark evaluation；
16. 按仓库要求 Verify 最终 evaluation command、config、checkpoint、dataset、split/fold、model name 和 metrics；
17. 只有在没有可用 checkpoint、无法不训练完成 evaluation，且任务或官方说明要求训练时，才进行 training；
18. 失败诊断与修复；
19. 按官方语义 Verify 每个智能体改动或 wrapper，并记录到 `change_summary.md`；
20. metric extraction；
21. Verify metric provenance 和 local-execution 状态；
22. 生成 `result.md`；
23. 保存原始 Codex `history.md`；
24. 生成 `change_summary.md`。

不要从 full benchmark run、full dataset download 或 full training run 开始。必须先通过 fresh-environment、checkpoint、dataset-size、disk-space 和 smoke gates。

## 阶段 Verify 策略

每个主要阶段在继续前都必须以一个 `Verify` 步骤结束。Verify 要把本地状态与 repository、paper、task card、config files、README、model card 或 official scripts 进行对比。

至少在以下阶段后 Verify：

- repository scan 和 target identification；
- environment creation 和 dependency installation；
- dataset discovery、download、binding、schema inspection 和 preprocessing；
- checkpoint 或 pretrained weight discovery/download；
- smoke test；
- benchmark evaluation；
- metric extraction；
- 每个智能体做的 source edit、wrapper、config copy、path adaptation 或 command-line change；
- final packaging。

Verify 必须记录：

- official materials 中的 expected version 或 value；
- actual local version 或 value；
- match status：`match`、`acceptable-drift`、`mismatch` 或 `unknown`；
- expectation source，例如 README line、config key、requirements file、paper table、model card 或 task card；
- mismatch 的处理动作。

不要重命名 model names、method names、benchmark names、metric names、dataset split names 或 paper row labels。如果官方名称有歧义，在 `result.md` 中保留来源写法，并在 `change_summary.md` 中记录歧义。

## 环境策略

- 默认：为每个 repo 创建新的隔离环境。
- 优先使用 `conda create`；如果 conda 不可用，则创建新的 venv。
- 环境名应包含 repo name 和 timestamp。
- 不要默认复用任何已有 conda/venv 环境。
- 不要使用 `base` 或 system Python 进行复现。
- 依赖失败后不要自动回退到旧环境。
- 只有用户明确要求复用已有环境时，才允许复用。

## 下载和训练策略

- 首先寻找作者提供的 pretrained checkpoints 或 model weights。
- 如果 pretrained checkpoints 可用，优先进行 checkpoint-based evaluation 或 inference reproduction，而不是 training。
- 优先选择最小有效 evaluation subset 或 smoke test。
- 不要默认下载完整 training datasets。
- 不要默认启动 full 或 long training。
- 如果必须下载 dataset，先估算 dataset size、检查 disk space、优先选择必要的最小 subset，并让理由出现在 logs 或原始 history trace 中。
- 只有在没有可用 checkpoint、无法不训练完成 evaluation，且任务或官方说明要求训练时，才允许 training。

## 安全规则

- 不要把 API keys、tokens、private credentials 或 paid-access secrets 写入 prompts、source files、logs、result tables 或 history。
- 不要使用 Docker、sudo 或危险递归删除命令。
- 不要删除 source code、configs、checkpoints、raw datasets、submission files 或 conversation history。
- 除非任务明确允许，否则不要修改 upstream repository code。
- 如果禁止 source edits，使用 wrappers、environment variables、本地 config copies，或记录 blocker。
- 不要伪造 datasets、checkpoints、metrics 或 paper values。
- 不要把 README 或 official table values 当作本地复现结果展示。
- 不要覆盖 system Python、修改 system CUDA，或依赖 `base`。

## 必需 references

按需加载：

- `references/paper-repo-reproduction.md`：repo scan、target identification、official entrypoint、wrapper policy。
- `references/low-interaction-agent.md`：low-interaction rules、interaction counting、progress updates。
- `references/environment-and-resource.md`：fresh environment/resource gate、CUDA、checkpoint、dataset、HF cache、proxy。
- `references/metric-extraction.md`：metric provenance、`result.md`、raw `history.md`、final submission layout。

## 指标来源 gate

每个最终指标都必须记录：

- metric name；
- numeric value；
- source file；
- source key、CSV column 或 log line；
- run type：`submission`、`smoke`、`diagnostic`、`official_table` 或 `blocked`；
- split：`test`、`validation`、`train`、`all`、`fold<N>`、`5-fold` 或 `unknown`；
- 该值来自 local execution 还是 official reference。

只有 `submission` 类型的 local-execution metrics 可以填入 `result.md`。Smoke、diagnostic 和 official-table-only values 不能作为复现结果提交。

## Change summary

单独生成 `change_summary.md`，不要和 `history.md` 混在一起。

它必须包括：

- 智能体做过的 file edits、wrappers、local config copies、command changes、environment changes 和 path adaptations；
- 每个改动为什么需要；
- 每个改动是否改变 benchmark semantics；
- 与官方 repository requirements 的 before/after comparison；
- 每一阶段的 `Verify` result；
- 未解决的 mismatches 或 accepted drifts。

它不能替代 `history.md`，不能总结原始 conversation trace，不能伪造缺失 tool calls，也不能重命名官方 model/method/dataset/metric names。

## 最终交付物

`result.md`：

- 固定 filename；
- 只包含 Markdown table；
- 不包含 notes、commands、explanations、screenshots 或 provenance text。

`history.md`：

- 固定 filename，路径为 `repo_name/history.md`；
- 完整原始 Codex conversation trace；
- 包含 user messages、assistant replies、tool calls、command records、errors、repair attempts 和 metric extraction process；
- 不允许改写成 report、不允许 summary substitute、不允许压缩或美化 transcript、不允许 assistant-only transcript、不允许伪造 conversation history。

`change_summary.md`：

- 固定 filename，路径为 `repo_name/change_summary.md`；
- 独立总结智能体改动和逐阶段 verification comparisons；
- 包含 expected vs actual versions、configs、datasets、checkpoints、scripts、model names 和 metric keys；
- 记录 mismatch handling 以及 benchmark semantics 是否改变；
- 不包含 raw logs、不包含完整 conversation transcript、不重命名 official model names。
