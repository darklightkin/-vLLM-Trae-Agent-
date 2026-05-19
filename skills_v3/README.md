# Local Codex Skills

This directory contains local skills for low-interaction paper repository reproduction.

## Available Skill

```text
.codex/skills/reproduce-repo/SKILL.md
```

Use this skill when a user provides a GitHub repository URL, local repository path, paper repository, benchmark repository, or asks to reproduce paper/table/benchmark results with minimal human interaction.

## Required Output Contract

Final submission directory must contain only:

```text
result.md
conversation_history/
```

The agent must also create an internal run summary:

```text
conversation_history/run_summary.json
```

`result.md` must contain only the required Markdown metric table.
