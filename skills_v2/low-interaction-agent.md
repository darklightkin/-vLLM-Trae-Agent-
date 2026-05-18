# low-interaction-agent v2

## 职责

负责减少人类交互轮次，同时保持过程可审计。它不替代复现流程，只规定什么时候自动决策、什么时候必须停下来问用户。

## 交互计数

- 初始 prompt 计 1 次用户交互。
- 用户补路径、换 GPU、授权私有数据、决定是否改代码，计额外交互。
- agent 自动扫描、工具调用、日志分析、自修复、进度更新，不计交互。

## 默认行为

先自动做：

- 读 README、configs、scripts、task card；
- 查 Python、conda/venv、CUDA、GPU、磁盘、代理；
- 查本地数据、checkpoint、cache；
- 运行最小 smoke；
- 分析错误并做有边界修复。

不要因为“可以问得更清楚”就问用户。只有没有安全默认值时才问。

## 必须询问用户的情况

- 需要 token、账号、私有数据权限；
- 需要切换机器、GPU 或远程服务器；
- 需要删除非临时文件；
- 需要修改 prompt 明确禁止修改的原仓库代码；
- 多个结果都可能作为提交结果，且无法从任务要求判断。

## 进度反馈

长任务必须输出状态，格式：

```text
[阶段] 当前动作；原因；下一步。
```

推荐阶段：

- `[扫描]` 仓库、README、configs、scripts；
- `[环境]` Python、CUDA、GPU、磁盘、代理；
- `[依赖]` package import 或安装；
- `[模型]` checkpoint 和 encoder；
- `[数据]` dataset、split、cache；
- `[试跑]` smoke 或最小 run；
- `[评测]` 目标 benchmark；
- `[修复]` 错误分类和最小修复；
- `[指标]` 解析 JSON/log/csv；
- `[提交]` 写表格和 history。

## 有界修复循环

每次失败按顺序做：

1. 读完整 traceback 或 log。
2. 分类：依赖、配置、路径、数据、checkpoint、显存、网络、代码接口、权限。
3. 做最小修复或最小绕过。
4. 重跑能到达失败点的最小命令。
5. 同类失败重复 2 次后，停止并写清 blocker。

## 安全默认值

- 使用隔离环境，不污染系统 Python。
- 优先官方脚本和作者 checkpoint。
- CUDA 可用才跑目标重任务；CPU 先做 smoke 或单 fold。
- 输出放到 repo-local `outputs/results` 或 workspace `outputs/<repo>`。
- history 记录实际选择，不在最终表格解释过程。
