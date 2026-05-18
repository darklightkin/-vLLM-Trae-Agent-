Reproduce the requested paper result with the fewest possible human interactions.

Repository: https://github.com/SalesforceAIResearch/GUI-KV
Local path: D:\vLLM1\model\GUI-KV
Paper: https://arxiv.org/pdf/2510.00536
Target: Reproduce Table 1 on AgentNetBench with the pretrained UI-TARS-1.5-7B model.
Benchmark: AgentNetBench
Recommended model/checkpoint: ByteDance-Seed/UI-TARS-1.5-7B
Required settings:
- {"method": "GUI-KV", "budget": "80%"}
- {"method": "GUI-KV", "budget": "10%"}

Requirements:
1. Inspect the repository README and scripts before running anything expensive.
2. Create or reuse an isolated environment and record the exact commands.
3. Prefer the repository's official evaluation script: bash eval/agentnetbench_eval.sh, or call eval/agentnetbench_eval.py with --kv_cache gui_kv and the target budgets.
4. Save raw logs and result files under an outputs/results directory.
5. Produce only the required reproduction metrics table for submission.
6. Preserve the complete conversation history.
7. If a dependency, dataset, checkpoint, GPU, or API key is missing, report the smallest concrete blocker and the command or path needed to continue.

Submission folder name: gui-kv
Table columns: Method, Budget, Step Accuracy
