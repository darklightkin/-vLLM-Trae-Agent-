# Metric Extraction and Submission Reference

## 职责

从真实 artifacts 中提取 metrics，强制 provenance，写入固定 `result.md`，将原始 `history.md` 保存为 benchmark interaction trace，并为智能体改动和逐阶段 verification comparisons 写入独立 `change_summary.md`。

## Run classification

每个产生 metrics 的运行都必须分类：

| Run type | Meaning | Can feed result.md |
| --- | --- | --- |
| `smoke` | 最小 import/model/data/forward/metric check | No |
| `diagnostic` | schema/split/system/cache/protocol inspection | No |
| `submission` | 为最终表格选择的 target benchmark/model/split/fold | Yes |
| `official_table` | README 或 paper reference value | No |
| `blocked` | 没有产生有效 local metric | No |

## Metric provenance table

对每个最终 metric，在 work logs 或 raw trace artifacts 中记录 provenance：

| Metric | Value | Source file | Source key/line | Run type | Split/fold | Local execution? |
| --- | --- | --- | --- | --- | --- | --- |

允许的 split/fold values 包括：

- `test`；
- `validation`；
- `train`；
- `all`；
- `fold0`、`fold1` 等；
- `5-fold`；
- `unknown`。

如果 source 是 `all`、`smoke`、`diagnostic` 或 `official_table`，除非任务明确要求，否则不要用于 `result.md`。

## Verify

Metric extraction 后，用 repository、paper、task card 和 evaluation script expectations Verify 每个 metric。

Verify：

- metric name spelling 与 official materials 或 required submission schema 完全一致；
- value 来自本地 `submission` run，而不是 README、paper、smoke、diagnostic 或复制的 official table values；
- source file 和 source key/line 已记录；
- split/fold 与 target protocol 匹配；
- model name 和 method name 被准确保留；
- 没有被 agent wrapper 或 edit 改变 metric formula、label mapping、split 或 aggregation。

在 `change_summary.md` 中记录 expected vs actual metric names、split/fold、source artifacts 和任何 accepted drift。

## result.md rules

最终 filename 固定为：

```text
result.md
```

内容只能是一个 Markdown table：

```markdown
| Method | Metric 1 | Metric 2 |
| --- | --- | --- |
| <method> | <value> | <value> |
```

不包含 notes、commands、provenance、screenshots 或 explanations。

## history.md raw trace rule

`history.md` 不是 report template。它必须是该运行的完整原始 Codex conversation record。

必须按原样包含：

- user inputs；
- assistant replies；
- tool calls；
- command execution records；
- errors 和 repair attempts；
- metric extraction process。

禁止：

- 用 summary 替代 history；
- 用 reproduction report 替代 history；
- rewrite、compress、beautify、filter 或 reorganize raw conversation；
- 只保留 assistant output；
- 伪造 conversation history。

Metric provenance 可以出现在 raw trace 和 internal logs 中，但不要把 `history.md` 转换成 polished provenance report。

如果 host system 无法导出 raw trace，记录 `blocked`，不要伪造 `history.md`。

## change_summary.md rules

`change_summary.md` 是单独的 documentation artifact。它不是 `history.md` 的替代品。

它必须总结：

- agent-made source edits、wrappers、config copies、command changes、path adaptations 和 environment workarounds；
- 每个 change 为什么发生；
- 与 repository-specified versions 或 behavior 的 before/after comparison；
- repo scan、target selection、environment、dataset、checkpoint、smoke、evaluation、metrics 和 packaging 的阶段 `Verify` results；
- benchmark semantics 是否改变；
- unresolved mismatches、accepted drifts 和 blockers。

建议表格：

```markdown
| Stage | Official requirement | Local actual | Status | Source | Action |
| --- | --- | --- | --- | --- | --- |
| environment | Python 3.10 | Python 3.10.13 | match | README | none |
```

不要用 `change_summary.md` 重命名 official model names、改写 raw conversation、隐藏 failed attempts，或把 smoke/diagnostic metrics 呈现为 final results。

## Final submission directory

最终 package directory 只能包含：

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

除非用户明确要求，不要把 raw checkpoints、datasets、cache、raw JSONL、scripts、outputs、logs 或 work files 放入最终提交目录。
