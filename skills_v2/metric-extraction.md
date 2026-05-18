# metric-extraction v2

## 职责

负责从真实运行产物中抽取目标指标，并生成提交用 Markdown 表格。它只处理“指标可信提取”和“表格格式”，不解释复现过程。

## 输入来源

优先级从高到低：

1. 目标 run 的 JSON/CSV 结果文件；
2. 目标 run 的 stdout/stderr log；
3. 官方 eval script 输出；
4. README/paper 参考值。

README/paper 参考值只能标为 reference，不能伪装成本地 run 结果。

## 匹配规则

抽取指标前必须确认 run 匹配：

- repo；
- benchmark；
- model/method；
- checkpoint；
- fold 或 setting；
- dataset split；
- 运行时间和输出路径。

多个 run 同时存在时，选择最符合任务要求的 run；不要混合不同 run 的单元格。

## 表格规则

最终文件名默认：

```text
reproduction.md
```

文件内容只允许 Markdown 表格：

```markdown
| Method | System SRCC | Utterance SRCC |
| --- | --- | --- |
| A1 (Frozen+MSE) [recommended] | <value> | <value> |
```

不要写命令、解释、截图、来源说明或 “注：”。这些内容放到 history。

## 数值规则

- 保留任务要求的小数位；如果没有要求，默认 3 位小数。
- 不要为了达标四舍五入到误导程度。
- 如果指标缺失，写清 blocker，不要填猜测值。
- 如果是单 fold、subset、CPU smoke 等非完整复现，表格仍可写真实值，但 history 必须说明限制。

## 验收

提交前确认：

- 每个单元格都能追溯到原始文件；
- 没有 placeholder；
- 表格列名和任务要求一致；
- 原始日志和 JSON/CSV 不写进 `reproduction.md`；
- history 中记录了指标来源路径。
