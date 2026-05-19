# reproduce-repo v6 design notes

This file is design context, not the runtime protocol. During execution, the agent should start from `reproduce-repo/SKILL.md` and load only the needed reference files.

## Design Goal

Turn paper-repo reproduction into a small state machine:

```text
SCAN -> TARGET -> ENV -> RESOURCE -> SMOKE -> EVAL -> METRIC -> SUBMIT
                         \-> REPAIR -> previous gate
                         \-> BLOCKED
```

Each gate has:

- entry criteria;
- actions;
- verification;
- artifacts;
- bounded repair or stop condition.

## Key Principles

- `SKILL.md` is a dispatcher, not a tutorial.
- References hold stage details.
- The agent should read less, act sooner, and verify critical facts.
- Prefer pretrained/checkpoint evaluation over training.
- Prefer minimal eval resources before full runs.
- Preserve raw history; summarize edits separately.

## Required Runtime Artifacts

Final submission:

```text
repo_name/
  result.md
  history.md
  change_summary.md
```

Meaning:

- `result.md`: table only, from valid local submission metrics.
- `history.md`: raw Codex trace only.
- `change_summary.md`: agent changes, deviations, and verification comparisons.

## Verification Contract

At major gates, compare official expectation to local actual:

```text
Official requirement | Local actual | Status | Source | Action
```

Statuses:

- `match`;
- `acceptable-drift`;
- `mismatch`;
- `unknown`.

Verify at least:

- repo scan and target lock;
- environment and dependency setup;
- dataset binding/download;
- checkpoint discovery/download;
- smoke test;
- official evaluation;
- metric extraction;
- source edits/wrappers/config copies/path adaptations;
- final packaging.

## Lessons Incorporated

v6 addresses common failures:

- accidentally reusing old environments;
- defaulting to training or full dataset downloads;
- losing checkpoint-first behavior;
- replacing raw conversation history with a polished report;
- hiding environment fallbacks or wrapper changes;
- copying README/paper metrics as reproduced values;
- spending too long on slow networks without recording fallback decisions.

## Future Optional Scripts

Scripts can be added later for stable checks:

```text
collect_env.py
validate_result_md.py
validate_submission_dir.py
extract_metrics.py
write_run_summary.py
```

Do not add scripts unless repeated failures show Markdown instructions are insufficient.
