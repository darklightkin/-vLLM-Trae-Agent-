# reproduce-repo v6 设计思路

你的 `skills_v5` 当前已经有了正式 skill 的骨架，比 `autoresearch/program.md` 更规范。接下来改进的核心不是继续堆文字，而是把它从“复现说明书”升级成一个可执行、可审计、可迭代的复现 Agent 协议。

## 总体设计思路

`autoresearch` 的设计是：

```text
program.md = Agent 行为协议
train.py = Agent 可修改对象
固定指标 = 判断是否成功
实验循环 = 自动迭代
```

你的复现仓库 skill 应该对应成：

```text
SKILL.md = 总控协议，定义什么时候触发、必须遵守什么
references/ = 分阶段细则
scripts/ = 可重复、易错、必须稳定的检查动作
templates/ 或 assets/ = result.md / history.md / run_summary.json 模板
work logs = 复现过程状态机和证据链
```

也就是说，`SKILL.md` 不应该越来越长，而应该越来越像“调度器”。

## 你现在 v5 的优点

当前 [SKILL.md](D:/vLLM1/-vLLM-Trae-Agent-/skills_v5/reproduce-repo/SKILL.md) 已经有几个很好的设计：

- 触发条件清楚：GitHub repo、本地 repo、paper benchmark、失败复现继续。
- 输出强约束清楚：最终只要 `result.md` 和 `history.md`。
- 有 checkpoint-first 思想：先评估作者权重，不默认训练。
- 有低交互规则：能自动推断就不问用户。
- 有 metric provenance gate：避免把 README 指标当本地复现结果。
- references 拆得合理：环境、指标、低交互、仓库扫描分开了。

这个方向是对的。

## 现在最大的缺口

下一版建议重点补 5 件事。

### 1. 加状态机

现在 workflow 是线性列表，但复现任务经常会失败、回退、分叉。你需要把它设计成状态机：

```text
SCAN
TARGET_LOCK
ENV_READY
CHECKPOINT_READY
DATA_READY
SMOKE_PASSED
EVAL_DONE
METRIC_VERIFIED
SUBMISSION_READY
BLOCKED
```

每个阶段都应该有：

```text
entry criteria
actions
exit criteria
failure handling
artifacts produced
```

这样 Agent 不会“感觉差不多了就继续”。

### 2. 加 run_summary.json 作为内部审计文件

虽然最终提交目录只放：

```text
result.md
history.md
```

但工作目录里应该生成一个结构化状态文件，例如：

```json
{
  "repo": "MuQ-Eval",
  "target": "paper table 1",
  "status": "submission_ready",
  "environment": {
    "python": "...",
    "torch": "...",
    "cuda_available": true
  },
  "checkpoint": {
    "source": "...",
    "path": "...",
    "hash_or_size": "..."
  },
  "dataset": {
    "name": "...",
    "split": "...",
    "sample_count": 1234
  },
  "metrics": [
    {
      "name": "accuracy",
      "value": 0.923,
      "source_file": "eval.log",
      "run_type": "submission",
      "local_execution": true
    }
  ],
  "blockers": []
}
```

这不是最终提交物，而是 Agent 自己判断“我现在是否真的完成了”的依据。

### 3. 把易错检查变成 scripts

现在很多规则写在 Markdown 里，Agent 每次都要重新理解。建议新增：

```text
scripts/
  scan_repo.py
  collect_env.py
  metric_extract.py
  validate_submission.py
  make_submission.py
```

特别是这几个动作值得脚本化：

```text
检查 result.md 是否只有 Markdown table
检查最终目录是否只包含 result.md 和 history.md
提取日志中的 metric
记录 Python / CUDA / torch / disk 信息
生成 run_summary.json
```

Markdown 负责“什么时候做”，脚本负责“稳定地做”。

### 4. 把 history.md 规则再落地

你现在已经写了 raw trace rule，但现实里 Agent 很容易把 history 写成总结报告。建议增加明确判断：

```text
history.md invalid if:
- begins with "Summary" / "Reproduction Report"
- omits tool calls
- omits failed commands
- only contains final explanation
- rewrites chronology
```

还可以在 `metric-extraction.md` 里加入一句：

```text
If raw trace cannot be exported from the host system, record BLOCKED instead of fabricating history.md.
```

这个很重要，防止假历史。

### 5. 增加失败预算和停止条件

复现任务不像 `autoresearch` 可以无限跑。你应该定义 bounded repair：

```text
同一类错误最多自动修复 3 次
依赖冲突最多换 2 种安装策略
CUDA OOM 最多尝试 3 个降级方案
数据 schema 不明时最多做 2 次探测
超过预算进入 BLOCKED
```

否则 Agent 可能一直修环境、一直跑偏。

## 建议的 v6 目录结构

可以从现在的：

```text
reproduce-repo/
  SKILL.md
  references/
    paper-repo-reproduction.md
    environment-and-resource.md
    metric-extraction.md
    low-interaction-agent.md
```

升级成：

```text
reproduce-repo/
  SKILL.md
  references/
    state-machine.md
    paper-repo-reproduction.md
    environment-and-resource.md
    data-and-checkpoint.md
    failure-recovery.md
    metric-extraction.md
    low-interaction-agent.md
    submission-contract.md
  scripts/
    collect_env.py
    validate_result_md.py
    validate_submission_dir.py
    extract_metrics.py
    write_run_summary.py
  assets/
    result_template.md
    run_summary_schema.json
```

其中 `SKILL.md` 只保留最核心的调度：

```text
1. Trigger
2. Required outputs
3. State machine order
4. Non-negotiable safety rules
5. Which reference to load at each stage
6. When to use scripts
7. Final validation gate
```

## SKILL.md 应该怎么迭代

不要按“想到什么加什么”来改。建议按这个循环：

```text
一次真实复现任务
记录失败点
判断失败类型
决定放哪里
更新 skill
再用新任务验证
```

失败类型可以这样分：

```text
Agent 忘了做某事 -> 加到 SKILL.md 的硬规则
Agent 不知道怎么判断 -> 加到 references
Agent 每次判断不稳定 -> 写 scripts
Agent 输出格式常错 -> 写 validator
Agent 经常问用户 -> 加 low-interaction 默认策略
Agent 经常跑重任务 -> 加 gate / stop condition
```

## 最关键的一条原则

`SKILL.md` 不是教程，应该是协议。

好的 `SKILL.md` 应该像这样：

```text
什么任务触发我
我必须产出什么
我绝对不能做什么
我按什么状态推进
每个阶段读哪个 reference
什么时候必须停
最后怎么验证自己没造假
```

细节、例子、异常处理、历史经验，不要都塞进 `SKILL.md`，放到 references 或 scripts。

## 接下来最值得做的改进顺序

1. 新增 `references/state-machine.md`
2. 新增 `references/failure-recovery.md`
3. 新增 `references/submission-contract.md`
4. 在 `SKILL.md` 里把线性 workflow 改成状态机入口
5. 加 `scripts/validate_result_md.py`
6. 加 `scripts/validate_submission_dir.py`
7. 加 `run_summary.json` 内部审计规范

## 本轮 v6 迭代要求

本轮 v6 在 `skills_v5` 基础上迭代，保留现有文档框架、skill 名称、reference 文件名和模型/方法/数据集/指标名称，不通过重命名来制造一致性。改动重点放在每个 skill/reference 的行为协议上。

新增核心要求：

```text
每一阶段之后都必须 Verify
Verify 必须与仓库规定版本、配置、模型名称、数据集、权重、脚本入口和指标协议对比
Agent 自己做的改动必须同时出现在 history.md 和单独的 change_summary.md
change_summary.md 必须总结改动原因、前后对比、Verify 结果和是否改变 benchmark 语义
```

阶段 Verify 至少覆盖：

- 仓库扫描和目标识别；
- 环境创建和依赖安装；
- dataset 发现、下载、绑定、schema 检查和预处理；
- checkpoint/权重发现和下载；
- smoke test；
- benchmark evaluation；
- metric extraction；
- Agent 自己做的源码改动、wrapper、配置复制、路径适配和命令变化；
- 最终打包。

每个 Verify 都要记录：

```text
Official requirement
Local actual
Status: match / acceptable-drift / mismatch / unknown
Source
Action
```

最终提交目录从原来的：

```text
repo_name/
  result.md
  history.md
```

扩展为：

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

其中 `history.md` 仍然是原始 Codex 交互轨迹，不能被总结报告替代；`change_summary.md` 是单独的改动与 Verify 对比文档，不能替代原始历史。

## 一句话总结

你的 v5 已经是“规则型 skill”，下一版应该升级成“状态机 + 验证器 + 审计文件”的复现系统。
