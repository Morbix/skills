---
name: detailed-plan-tech
description: Generates a detailed multi-stage technical plan as markdown files, progressing through research, draft, specialist review, and final plan stages. ONLY invoke when the user explicitly runs /detailed-plan-tech — never auto-invoke based on context.
allowed-tools: Read, Write, Edit, Bash, Agent, WebFetch, WebSearch
---

# detailed-plan-tech

Produces a structured technical plan through 3 permanent stage files: research, draft (with specialist annotations), and final plan.

## Rules

- **Never commit** any files unless the user explicitly requests it.
- **Never modify existing code** unless the user explicitly requests it.
- **Keep all stage files** — never delete or overwrite a previous stage.
- If `$ARGUMENTS` is empty, ask: "What context name should I use for this plan? (e.g. AUTH_REFACTOR, PAYMENT_API)" — do not proceed until answered.

## Initialization (run at skill start, before any stage)

Capture these values once and reuse them throughout — all files must share the same values:

```bash
ROOT=$(pwd)
DATE=$(date +%Y%m%d)
NAME=$(echo "$ARGUMENTS" | tr '[:lower:]' '[:upper:]' | tr ' ' '_')
```

If `$ARGUMENTS` is empty, ask for the context name before running these commands.

## File naming

Pattern: `<NAME>_<STAGE>_<DATE>.md`, saved in `$ROOT`.

- All uppercase
- NAME: from `$ARGUMENTS`, uppercased, spaces → underscores (computed above)
- STAGE: `01_RESEARCH`, `02_DRAFT`, or `03_PLAN`
- DATE: captured from `date +%Y%m%d` **at invocation time** — the same value used for all 3 files

## Workflow

Present this checklist at the start and update it as stages complete:

```
Plan Progress:
- [ ] Stage 1 — Research
- [ ] Stage 2 — Draft
- [ ] Stage 3 — Specialist review (Architecture / Staff / Manager)
- [ ] Stage 4 — Final plan
```

---

### Stage 1 — Research → `<NAME>_01_RESEARCH_<DATE>.md`

Explore the project thoroughly before writing anything:

1. Read `CLAUDE.md`, `README*`, package manifests, and config files
2. Map the directory tree: `find . -maxdepth 3 -not -path '*/node_modules/*' -not -path '*/.git/*'`
3. Identify stack, frameworks, patterns, test setup, and CI configuration
4. Locate code areas relevant to the user's request
5. Note tech debt, constraints, and open questions that affect planning

Write the research file:

```markdown
# Research: <context name>

## Project overview
[stack, purpose, scale, maturity]

## Architecture
[key components, data flow, dominant patterns]

## Relevant code areas
[specific files/modules touched by or related to this request]

## Constraints & risks
[existing tech debt, performance concerns, compatibility, hard deadlines]

## Open questions
[ambiguities that must be resolved before or during planning]
```

---

### Stage 2 — Draft → `<NAME>_02_DRAFT_<DATE>.md`

Produce the first version of the plan using the research findings.

Tasks: typically 2–10 items, varying by scope — more or fewer as context demands.
Each task must have: **Goal**, **Approach**, and **Acceptance criteria**.

```markdown
# Draft Plan: <context name>

## Objective
[what this plan achieves and why it matters]

## Scope
**In scope:** ...
**Out of scope:** ...

## Tasks

### Task 1 — <title>
**Goal:** ...
**Approach:** ...
**Acceptance criteria:** ...

[repeat for all tasks]

## Execution order
[which tasks must precede others and why]

## Top risks
[3 risks with mitigations]
```

---

### Stage 3 — Specialist Review (annotate the draft in-place)

Three specialists review `02_DRAFT` independently. Append a `## Specialist Reviews` section to the **draft file** — do not create a new file for this stage.

#### [A] Architecture Engineer
Focuses on: scalability, design patterns, coupling, extensibility, system boundaries.

Add `### [A] Architecture Review` with:
- Pattern choices and their trade-offs
- Scalability or coupling concerns
- Suggested structural changes

#### [B] Staff Engineer
Focuses on: performance, code quality, testability, maintainability, key trade-offs, risks.

Add `### [B] Staff Engineer Review` with:
- Performance hotspots or regressions introduced
- Code quality and testability concerns
- Concrete risk mitigations

#### [C] Engineering Manager
Focuses on: does the plan solve the real problem? Does it deliver proportional value to users and the project?

Add `### [C] Engineering Manager Review` with:
- Alignment with the user's actual stated need
- Value vs effort assessment
- Scope creep flags or missing scope
- Prioritization or sequencing suggestions

---

### Stage 4 — Final Plan → `<NAME>_03_PLAN_<DATE>.md`

Consolidate all specialist annotations into the best possible plan.
Resolve reviewer conflicts; prefer the more conservative risk posture when uncertain.
Tasks that are out of scope but valuable should be preserved in a "Future" section.

```markdown
# Technical Plan: <context name>
_Date: <DATE value captured at invocation>_

## Objective
[refined from draft after review]

## Scope
**In scope:** ...
**Out of scope:** ...

## Checklist
- [ ] Task 1 — <title>
- [ ] Task 2 — <title>
[one line per task, same order as Tasks section below]

## Tasks

### Task N — <title>
**Goal:** ...
**Approach:** ...
**Acceptance criteria:** ...
**Review notes:** [key input from specialists, if any]

[repeat]

## Execution order
[finalized with rationale]

## Risks & mitigations
[consolidated from all three reviewers]

## Future / out of scope
[deferred but valuable items worth tracking]

## Review delta
[1–3 sentences on what changed most between draft and final plan, and why]
```

---

## Final report to user

After all stages complete, output:
- Paths of the 3 files created
- Task count in the final plan
- Top risk and its mitigation
- One-sentence summary of the most impactful specialist feedback
