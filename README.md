# skills

Personal Claude Code skills (slash commands). Each skill lives in its own directory with a `SKILL.md` that defines its prompt, model, and allowed tools.

## Skills

| Skill | Description |
|---|---|
| `/commit-all` | Stage all changes and create a well-crafted commit |
| `/commit-staged` | Create a well-crafted commit from already-staged changes |
| `/detailed-plan-tech` | Generate a multi-stage technical plan (research → draft → review → final) |

## Adding a skill

```
mkdir <skill-name>
# create <skill-name>/SKILL.md with frontmatter + prompt
```

See `CLAUDE.md` for the full skill format.
