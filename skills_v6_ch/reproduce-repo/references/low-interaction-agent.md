# Low Interaction Reference

## 职责

在保持正确性和可审计性的同时，减少人类交互轮数。

## Interaction counting

- human turns 按用户主动发送给 Codex 的消息计算。
- 用户 path fixes、GPU changes、credentials、private data decisions、large-download approvals、long-training approvals、deletion approvals、system-level modification approvals 和 source-edit approvals 都计为额外交互轮数。
- Agent scans、shell commands、progress updates、automatic repairs 和 log analysis 不计数。
- 每个不可避免的问题都必须记录为 `interaction_count_candidate`。

## 可推断时不要询问

不要向用户询问可以从以下来源推断的信息：

- README、configs、scripts、paper、model card 或 issue text；
- local directory names 和 `git remote -v`；
- offline resource README files；
- environment probes；
- dataset/checkpoint inspection；
- logs 和 tracebacks。

选择最安全、保持 benchmark 语义的默认值，并记录它。

默认环境决策：创建新的 isolated environment。不要询问是否复用环境，也不要静默复用 `base`、current shell environment、previous conda env 或 old venv。只有用户明确要求复用时，才允许复用。

默认复现决策：先寻找 author checkpoints 或 pretrained weights，然后用最小有效 evaluation subset 或 smoke test 运行 checkpoint-based evaluation/inference。除非 training gate 已满足，否则不要询问是否启动 full training。

默认 Verify 决策：每个主要阶段后，将本地状态与 repository-specified versions、configs、model names、dataset versions、checkpoint identities、script entrypoints、split/fold protocols 和 metric keys 对比。不要让用户 Verify 可以从 repository files、papers、task cards、model cards、logs 或 local command outputs 中检查的信息。

## 阻塞时必须询问

只在以下情况询问：

- API key、account、token 或 private dataset permission；
- paid download 或 license decision；
- large dataset download 或 unknown-size dataset download；
- long training 或 full training run；
- machine/GPU/server switch；
- system-level modification；
- 删除 non-temporary files；
- 在禁止修改 upstream code 时仍需修改；
- 无法从任务要求中解析的 ambiguous final submission choice。

## Progress updates

使用简短阶段更新：

```text
[scan] current action; reason; next step.
[verify] current comparison; expected vs actual; next action.
[env] current action; reason; next step.
[data] current action; reason; next step.
[model] current action; reason; next step.
[smoke] current action; reason; next step.
[eval] current action; reason; next step.
[repair] current action; reason; next step.
[metrics] current action; reason; next step.
[submit] current action; reason; next step.
```

在 environment setup、dataset handling、checkpoint handling、smoke tests、evaluation、metric extraction、agent-made edits 和 final packaging 后使用 `[verify]`。

## Interaction stats

有用时，在 internal notes 或 raw history trace 中追踪 interaction stats：

- `human_turns`：统计 user-sent messages，不统计 agent tool calls 或 automatic repairs；
- `repair_count`：统计 bounded automatic repair attempts；
- `smoke_count`：统计 final evaluation 前运行的 smoke commands；
- `final_success`：只有当 `result.md` 来自 valid submission run 时才为 true；
- `verify_count`：统计已完成的 stage verification checks；
- `mismatch_count`：统计需要 repair、accepted drift 或 blocker recording 的 verification mismatches；
- `change_summary_ready`：只有当 `change_summary.md` 记录了 agent edits 和 stage verification comparisons 时才为 true。

除非用户明确要求，不要把 stats files 放入最终提交目录。
