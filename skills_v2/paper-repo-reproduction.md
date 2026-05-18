# paper-repo-reproduction v2

## 职责

负责“科研仓库复现”的通用执行流程：读官方材料、识别入口、建立 inventory、选择最小可验证 run，并把复现过程变成可追溯产物。

本文件是 `reproduction-master.md` 的小 skill，只处理仓库和复现流程，不处理 GPU/checkpoint 细节，也不负责最终指标表格式。

## 输入

- GitHub URL 或本地 repo 路径；
- paper/table/README 中的目标结果；
- benchmark、模型、checkpoint、dataset；
- 提交文件要求；
- 是否允许修改原仓库代码。

## Repo Inventory

进入仓库后先建立 inventory，不要直接跑命令：

| Component | 要找什么 |
| --- | --- |
| README | 官方复现说明、目标表格、下载链接 |
| requirements | Python、CUDA、torch、特殊包 |
| configs | 目标模型和 benchmark 配置 |
| scripts | train/eval/infer 入口 |
| data loader | dataset 名称、split、字段 |
| checkpoint | 作者权重、fold 结构、文件名 |
| output | logs、json、csv、metrics 路径 |

## 执行流程

1. 读取 README、requirements、configs、scripts、examples。
2. 找到官方 evaluation entrypoint。
3. 找到最小目标配置，不做无关 full sweep。
4. 确认 repo 是否完整：eval、config、dataset、checkpoint、metric parser 是否存在。
5. 先跑 smoke，再跑目标命令。
6. 保存所有命令、日志、结果文件路径。
7. 把失败分类后交给对应小 skill 或总控流程。

## 代码修改规则

- 默认不修改原仓库代码。
- 如果 prompt 明确禁止修改原仓库，必须使用 wrapper、环境变量、临时 copy 或记录 blocker。
- 如果 prompt 允许修改，也只做最小兼容 patch，并记录 diff、原因、验证命令。
- 不能为了让结果好看而改 metric、dataset split 或评测协议。

## 成功标准

- 知道官方目标命令是什么；
- 知道实际跑通的命令是什么；
- 知道任何偏离官方流程的原因；
- 最终指标能追溯到真实日志或结果文件。
