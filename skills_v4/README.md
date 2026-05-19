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
repo_name/
  result.md
  history.md
```

`result.md` must contain only the required Markdown metric table.
`history.md` must contain the complete raw Codex conversation trace, not a summary or rewritten reproduction report.
