# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of personal Claude Code skills (slash commands). Each skill lives in its own directory containing a single `SKILL.md` file. Skills are invoked in Claude Code sessions via `/skill-name`.

## Skill structure

Each skill directory follows this pattern:

```
<skill-name>/
  SKILL.md      ← the skill definition
```

`SKILL.md` uses YAML frontmatter followed by the skill prompt body:

```markdown
---
name: skill-name          # optional; defaults to directory name
description: one-liner    # shown in skill picker
model: haiku|sonnet|opus  # model to run the skill on
allowed-tools: Read, Write, Edit, Bash, Agent, ...   # tools the skill may use
---

<skill prompt body>
```

The frontmatter fields `model` and `allowed-tools` are optional. Omit them to inherit defaults. The prompt body can use `!`backtick`command`backtick`` syntax to shell-interpolate values at invocation time, and `$ARGUMENTS` to receive arguments the user passes after the slash command.

## Adding a new skill

1. Create a new directory: `mkdir <skill-name>`
2. Create `<skill-name>/SKILL.md` with frontmatter + prompt
3. The skill becomes available as `/<skill-name>` in Claude Code sessions

## Conventions observed in this repo

- Commit messages use imperative mood, explain *why*, ≤72 chars, no trailing period.
- Skills that should never auto-invoke state that explicitly in their `description` frontmatter field (e.g. "ONLY invoke when the user explicitly runs /...").
- Skills include an `<output_format>` section when their output shape matters.
- Co-authorship line in commits: `Co-Authored-By: Claude <model> <noreply@anthropic.com>`.
