Reproduce the requested paper result with the fewest possible human interactions.

Repository: https://github.com/microsoft/GUI-Actor
Local path: D:\vLLM1\model\GUI-Actor
Paper: https://www.arxiv.org/pdf/2506.03143
Target: Reproduce ScreenSpot-Pro with GUI-Actor-7B from this repository.
Benchmark: ScreenSpot-Pro
Recommended model/checkpoint: microsoft/GUI-Actor-7B-Qwen2-VL
Required settings:
- {"method": "GUI-Actor-7B", "backbone": "Qwen2-VL"}

Requirements:
1. Inspect the repository README and scripts before running anything expensive.
2. Create or reuse an isolated environment and record the exact commands.
3. Prefer the repository's official evaluation script: python eval/screenSpot_pro.py --save_path results/screenspot_pro --data_path <path_to_ScreenSpot-Pro>
4. Save raw logs and result files under an outputs/results directory.
5. Produce only the required reproduction metrics table for submission.
6. Preserve the complete conversation history.
7. If a dependency, dataset, checkpoint, GPU, or API key is missing, report the smallest concrete blocker and the command or path needed to continue.

Submission folder name: GUI-Actor
Table columns: Method, ScreenSpot-Pro
