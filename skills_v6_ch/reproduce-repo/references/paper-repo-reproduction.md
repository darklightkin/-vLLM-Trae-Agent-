# Paper Repo Reproduction Reference

## 职责

处理仓库级复现：扫描官方材料、识别目标、找到官方入口、建立清单、选择最小有效运行，并保持过程可追踪。

## Repo scan

在运行重命令前先阅读：

- README 和 reproduction docs；
- requirements 和 environment files；
- configs；
- train/eval/infer scripts；
- examples 和 shell scripts；
- paper/table references；
- 当本地文档不完整时，阅读 issues 或 model cards。

建立清单：

| Component | What to find |
| --- | --- |
| README | 官方复现说明和目标表格 |
| requirements | Python、CUDA、torch、flash-attn、vLLM、特殊 packages |
| configs | target model、benchmark、split、batch size |
| eval entrypoint | 官方 evaluation script 和 arguments |
| dataset | dataset ID、本地路径、split、schema、估计大小、最小 subset |
| checkpoint | pretrained/author checkpoint source、file names、size、fold layout |
| metrics | JSON/CSV/log output 和 metric names |
| submission | 需要的 file name 和 directory structure |

### Verify

Repo scan 后，在安装依赖或运行重命令前，用仓库自身材料 Verify 本地理解。

对比：

- repository name，commit 或 release tag；
- 官方 Python/package/CUDA requirements；
- 官方 eval/infer/train entrypoints；
- 官方 config files 和 default arguments；
- model names、method names、benchmark names、dataset names、split names 和 metric names；
- 必需 output files 和 submission layout。

在 `change_summary.md` 中记录 expected value、actual local value、expectation source、match status 和 mismatch action。

## Target identification

确认：

- target benchmark；
- target method/model/checkpoint；
- target row 和 columns；
- target split/fold protocol；
- accepted tolerance；
- 需要 one round、one fold 还是 full 5-fold。

如果 README 和 task card 冲突，提交要求优先采用 task card，并在 history 中记录差异。

### Verify

Target identification 后，用 paper、README、config、model card 和 task card Verify 选定目标。

不要重命名或规范化官方 model names、method names、table row labels、dataset split names 或 metric names。如果两个官方来源使用不同写法，保留提交目标要求的写法，并在 `change_summary.md` 中记录差异。

## Workflow

1. 阅读 docs 和 scripts。
2. Verify repository-declared requirements 和 official names。
3. 识别 official eval command。
4. Verify target command、config、metric names、split/fold 和 model names。
5. 搜索 author-provided pretrained checkpoints 或 model weights。
6. Verify checkpoint identity 和 compatibility。
7. 优先进行 checkpoint-based evaluation 或 inference reproduction。
8. 识别最小 target-equivalent evaluation subset。
9. 在任何 dataset download 前估算 dataset size 并检查 disk。
10. Verify dataset identity、version、split、schema 和 preprocessing。
11. 先运行 smoke。
12. Verify smoke output 和 artifact paths。
13. 运行 target submission evaluation。
14. Verify final command、config、dataset、checkpoint、model name 和 metrics。
15. 只有在没有 checkpoint、无法以其他方式完成 evaluation 且 training 是必需时，才训练。
16. 将失败转入 bounded recovery。
17. 将输出转入 metric extraction。
18. Verify metric provenance。
19. 打包 `result.md`、原始 `history.md` 和 `change_summary.md`。

## Checkpoint-first policy

- 如果 pretrained checkpoints 可用，优先进行 checkpoint-based evaluation 或 inference reproduction，而不是 full training reproduction。
- 当存在官方 checkpoint evaluation path 时，不要选择 training entrypoint。
- 在 checkpoint discovery 之前，不要下载完整 training datasets。
- 如果 checkpoint link 损坏或 gated，先记录 blocker，再考虑 training 作为 fallback。

## Dataset and training policy

- 不要默认 full dataset download。
- 不要默认 full 或 long training。
- 如果必须下载 dataset，先估算大小、检查 free disk、优先选择必要的最小 subset，并让理由出现在 logs 或 raw history trace 中。
- 将 full training 视为最后手段，而不是标准复现路径。

## Source-code and wrapper policy

- 默认：不要修改 upstream repo code。
- 如果官方代码与离线/本地资源不兼容，优先使用 external wrapper。
- wrapper 可以适配 paths、合并 configs、设置 cache variables，或调用现有 model/metric functions。
- wrapper 不能改变 benchmark semantics、labels、splits 或 metric formulas。
- 每个智能体所做 edit、wrapper、config copy、command-line change、path adaptation 或 environment workaround 都必须总结到 `change_summary.md`。
- 对每个改动，将 before/after behavior 与仓库官方版本或说明进行对比。
- 记录：

```text
Original repo modified: Yes/No
Wrapper script added: Yes/No
Wrapper reason: <reason>
Official behavior changed: Yes/No
Verify result: match/acceptable-drift/mismatch/unknown
```

## Lessons from history_v1

MuQ-Eval v1 运行中使用 external wrapper 而不是 patch upstream code，这一点表现良好。缺失部分是 final packaging 和 history preservation。未来运行必须只打包 `result.md`、原始 `history.md` 和 `change_summary.md`；`history.md` 必须是原始 Codex interaction trace，而不是清洗后的 reproduction report。
