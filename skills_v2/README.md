# Skills v2 设计说明

本目录保存第二版低交互复现 skills。设计从“多个平级规则文件”调整为“一个总控 skill + 多个外接小 skill”。

## 总分结构

- `reproduction-master.md`：总控 skill，负责阶段流程、门禁、路由、验收。
- `paper-repo-reproduction.md`：小 skill，负责仓库扫描、官方入口识别、复现流程。
- `low-interaction-agent.md`：小 skill，负责低交互策略、进度反馈、停止条件。
- `gpu-and-checkpoint-handling.md`：小 skill，负责 GPU、CUDA、checkpoint、HuggingFace cache 和数据下载。
- `metric-extraction.md`：小 skill，负责指标抽取、表格生成和结果校验。

## 借鉴思想

参考 `phd-skills` 的组织方式：总控文件只保留阶段和门禁，具体能力拆到独立 skill。这样 prompt 更短，也更容易在不同 repo 间复用。

## 使用方式

在 repo prompt 中优先引用：

```text
D:\vLLM1\skills\reproduction-master.md
```

然后要求 agent 按总控流程，在对应阶段读取需要的小 skill。不要把所有 skill 全文直接塞进 prompt。

## Prompt 调用模板

在给 agent 的 repo prompt 里，可以直接加入下面这一段：

```text
请使用本地 skills v2 执行本次论文仓库复现任务。

先读取总控 skill：
D:\vLLM1\skills\reproduction-master.md

然后按照总控流程，在对应阶段按需读取以下小 skill：
D:\vLLM1\skills\paper-repo-reproduction.md
D:\vLLM1\skills\low-interaction-agent.md
D:\vLLM1\skills\gpu-and-checkpoint-handling.md
D:\vLLM1\skills\metric-extraction.md

执行规则：
1. 不要把所有 skill 全文复制进回答，只在需要时读取对应文件。
2. 先做 repo scan、environment gate、resource gate、smoke gate，再跑目标评测。
3. 尽量少问用户；只有缺权限、缺 token、需要切换硬件、需要删除非临时文件、或必须修改被禁止修改的原仓库代码时才停下来。
4. 长任务必须用 `[扫描]`、`[环境]`、`[模型]`、`[数据]`、`[评测]`、`[指标]`、`[提交]` 这种格式给简短进度。
5. 最终生成 `reproduction.md` 和 `conversation_history/history.md`。
```

如果是 MuQ-Eval，可以在模板后面追加 task card：

```text
Task card:
- GitHub: https://github.com/dgtql/MuQ-Eval
- Local repo: D:\vLLM1\model\MuQ-Eval
- Benchmark: MusicEval
- Dataset: BAAI/MusicEval
- Checkpoint: zhudi2825/MuQ-Eval-A1
- Method: A1 (Frozen+MSE) [recommended]
- Metrics: System SRCC, Utterance SRCC
- Output table: D:\vLLM1\submission\MuQ-Eval\reproduction.md
- History: D:\vLLM1\submission\MuQ-Eval\conversation_history\history.md
- Do not modify original repo code.
```

## Repo 细节

通用规则留在 skills，具体 repo 信息放在 task card 或 prompt，例如：

- GitHub URL；
- benchmark；
- dataset；
- checkpoint；
- 目标指标；
- 是否允许修改原仓库代码；
- 输出文件路径。

## 最终产物

每个 repo 至少生成：

```text
submission/<repo>/reproduction.md
submission/<repo>/conversation_history/history.md
```

`reproduction.md` 只放结果表格；命令、错误、修复、数据来源和交互次数写入 history。
