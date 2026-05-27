---
name: bring-skill
description: Backs up the existing repo skill directory and brings the matching skill from Claude user settings into this repo, then commits. ONLY invoke when the user explicitly runs /bring-skill.
argument-hint: <skill-name>
model: sonnet
allowed-tools: Bash, Read, Write
---

# bring-skill

Backs up any existing skill directory (incrementing the suffix number if other backups already exist) and copies the skill from Claude user settings into this repo, then commits.

## Rules

- Never modify, move, or delete anything inside `~/.claude/` — only read from it.
- Never change user-level skill files.
- If `$ARGUMENTS` is empty, ask: "What skill name should I bring?" and stop until answered.
- Always commit at the end.

## Steps

### 1 — Validate input

If `$ARGUMENTS` is empty, ask for the skill name and do not proceed.

Set:
- `NAME` = `$ARGUMENTS`
- `REPO` = result of `pwd`

### 2 — Backup existing directory

Check whether `$REPO/$NAME/` exists.

If it does:
1. List all directories in `$REPO` matching `${NAME}_backup*`.
2. Extract the trailing 4-digit numbers (e.g. `0001`) from those names and find the highest one.
3. Next backup number = highest + 1, zero-padded to 4 digits. If no backups exist yet, use `0001`.
4. Run: `mv "$REPO/$NAME" "$REPO/${NAME}_backup<NNNN>"`

If `$REPO/$NAME/` does not exist, skip this step.

### 3 — Locate the skill in Claude user settings

Search in this order, stopping at the first match:

1. `~/.claude/commands/${NAME}.md`
2. `~/.claude/skills/${NAME}.md`
3. `~/.claude/skills/${NAME}/SKILL.md`

If none of the paths exist, report:

> Skill '$NAME' not found in ~/.claude/commands/ or ~/.claude/skills/

Then stop.

### 4 — Copy into repo

```bash
mkdir "$REPO/$NAME"
cp <found-source-path> "$REPO/$NAME/SKILL.md"
```

### 5 — Stage and commit

```bash
git -C "$REPO" add .
git -C "$REPO" commit -m "Bring $NAME skill from user settings

Co-Authored-By: Claude sonnet <noreply@anthropic.com>"
```
