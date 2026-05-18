# 一键复现 Paper Repo 的低交互方案

## Summary

目标从“环境自动配置工作流”调整为：用 vibe coding 工具在尽量少的人类交互轮次下，完成 3 个指定 repo 的指标复现，并提交每仓库 2 类文件：精度表格 md 和完整 conversation_history。

默认路线：使用成熟 Claude/Codex 类 agent 工具，配一套自定义 prompt + skill pack；模型策略采用“小到大阶梯”：先用便宜/小模型跑，失败再升到 Sonnet/Opus，并在 PPT 中记录“最小成功基座模型”。内部保留日志、脚本、指标解析器，但最终按官方要求只提交表格和 conversation history。

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
- 本方案优化点是低交互、低模型成本、可复用 prompt/skill。

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

每个 repo 使用同一个总 prompt，加一个 repo task card，避免每次重新解释规则。

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
