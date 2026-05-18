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
