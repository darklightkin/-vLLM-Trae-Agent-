# Skills v2 Design Notes

This folder contains the English version of the v2 low-interaction reproduction skills. The structure is changed from several peer rule files into one master skill plus several external sub-skills.

## Master/Sub-Skill Structure

- `reproduction-master.md`: master skill for stages, gates, routing, and acceptance.
- `paper-repo-reproduction.md`: sub-skill for repository scanning, official entrypoint discovery, and reproduction workflow.
- `low-interaction-agent.md`: sub-skill for low-interaction behavior, progress updates, and stop conditions.
- `gpu-and-checkpoint-handling.md`: sub-skill for GPU, CUDA, checkpoints, HuggingFace cache, and dataset download.
- `metric-extraction.md`: sub-skill for metric extraction, table generation, and result validation.

## Design Idea

This design follows the same pattern as `phd-skills`: the master file keeps only stages and gates, while concrete capabilities live in separate sub-skills. This keeps prompts shorter and makes the workflow reusable across repositories.

## How To Use In A Prompt

In a repo-specific prompt, first reference:

```text
D:\vLLM1\skills_en\reproduction-master.md
```

Then instruct the agent to read the required sub-skill only when the corresponding stage needs it. Do not paste all skill files into the prompt.

## Prompt Invocation Template

```text
Use the local skills v2 workflow for this paper-repository reproduction task.

First read the master skill:
D:\vLLM1\skills_en\reproduction-master.md

Then, following the master workflow, read these sub-skills only when needed:
D:\vLLM1\skills_en\paper-repo-reproduction.md
D:\vLLM1\skills_en\low-interaction-agent.md
D:\vLLM1\skills_en\gpu-and-checkpoint-handling.md
D:\vLLM1\skills_en\metric-extraction.md

Execution rules:
1. Do not copy the full skill text into the response; load the corresponding file when needed.
2. Run repo scan, environment gate, resource gate, and smoke gate before the target evaluation.
3. Minimize user questions. Stop only when credentials, hardware changes, non-temporary deletion, or prohibited source-code edits are required.
4. For long tasks, provide short progress updates with tags such as [scan], [env], [model], [data], [eval], [metrics], and [submit].
5. Generate `reproduction.md` and `conversation_history/history.md`.
```

## Repo-Specific Details

Keep general rules in skills. Put repo-specific information in the task card or prompt, such as:

- GitHub URL;
- benchmark;
- dataset;
- checkpoint;
- target metrics;
- whether source-code edits are allowed;
- output file paths.

## Final Artifacts

Each repo should produce at least:

```text
submission/<repo>/reproduction.md
submission/<repo>/conversation_history/history.md
```

`reproduction.md` contains only the result table. Commands, errors, fixes, data sources, and interaction counts go into history.
