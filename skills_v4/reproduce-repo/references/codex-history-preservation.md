# Codex History Preservation

## Responsibility

Preserve the real Codex conversation history as an audit artifact. `history.md` must preserve the original Codex conversation trace instead of a rewritten or summarized reproduction report.

## Real Conversation History

Real conversation history means the original conversation records produced by Codex or the host environment, including:

- user messages;
- assistant messages;
- tool calls;
- tool results;
- errors and interruptions;
- repair attempts;
- execution traces;
- timestamps or metadata when available;
- ordering exactly as recorded.

Do not delete, reorder, compress, paraphrase, or keep only assistant output.

## Forbidden Pseudo-History

Do not use any of these as a replacement for real conversation history:

- a hand-written summary;
- a reconstructed timeline;
- a Markdown narrative of what happened;
- only final commands;
- only assistant outputs;
- a shortened or cleaned transcript;
- a generated `history.md` that omits tool interactions.
- a filtered transcript that removes failed commands or repair attempts.

Summaries are allowed only as supplemental files inside the work directory, not as the required preserved conversation history.

## Export Procedure

Use the host-provided Codex export mechanism when available. If the raw history already exists in the repo or workspace, copy it unchanged into `conversation_history/`.

If the host cannot expose raw history, create a clear blocker note in the work directory and do not claim full history preservation. The final `run_summary.json.success` should be false if full conversation history is a required submission artifact and cannot be exported.

## Final Directory Rule

The final submission directory must be:

```text
repo_name/
  result.md
  conversation_history/
```

`conversation_history/` must contain the raw Codex history files and `run_summary.json`.

If a single `history.md` file is required, write the raw conversation trace into that file without reorganizing it into a reproduction report. Do not convert it into sections such as "Environment", "Commands", or "Metrics" unless those sections already exist in the raw trace.

Keep logs, checkpoints, outputs, scripts, summaries, caches, and temporary files outside the final submission directory.

## Integrity Checks

Before finalizing:

- verify user, assistant, and tool interactions are present;
- verify ordering is preserved;
- verify no redaction was done except for secrets that must not be stored;
- verify history is not just a summary;
- record the export source and file names in the work logs.
