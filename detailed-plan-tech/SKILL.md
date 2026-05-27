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

Pattern: `<DATE>_<NAME>_<STAGE>.md`, saved in `$ROOT`.

- All uppercase
- DATE: captured from `date +%Y%m%d` **at invocation time** — the same value used for all 3 files (prefix for chronological sorting)
- NAME: from `$ARGUMENTS`, uppercased, spaces → underscores (computed above)
- STAGE: `01_RESEARCH`, `02_DRAFT`, or `03_PLAN`

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

### Stage 1 — Research → `<DATE>_<NAME>_01_RESEARCH.md`

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

### Stage 2 — Draft → `<DATE>_<NAME>_02_DRAFT.md`

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

Three specialists review `02_DRAFT` independently. Launch all three as **parallel subagents** using `model: sonnet`. Each subagent reads the draft, inserts inline comments throughout the document at the precise locations they refer to, and returns a short summary. Wait for all three to complete, then merge the annotated drafts and append the summaries.

#### Inline comment format

Each comment is a blockquote immediately after the line or paragraph it addresses, tagged with the persona letter:

```markdown
> **[A]** The proposed singleton here creates tight coupling — consider a protocol-based injection instead.

> **[B]** This task has no rollback path if the migration fails mid-way.

> **[C]** This task is underspecified — acceptance criteria don't cover the error state.
```

Comments must be placed **directly after** the specific sentence, task, heading, or list item they concern — not grouped at the top or bottom of a section.

#### Launching subagents

Spawn three agents in a single message (parallel), each with `model: sonnet` and a self-contained prompt that includes:
- The absolute path to the draft file (read it first)
- The persona role and focus areas below
- Instruction to return the **full file content** with inline `> **[X]**` comments inserted at the exact locations they refer to
- Instruction to also return a short summary block (3–6 bullet points) at the end of the response, separated by `---SUMMARY---`

#### [A] Architecture Engineer
Focuses on: scalability, design patterns, coupling, extensibility, system boundaries.

Annotates inline wherever pattern choices, coupling risks, or structural concerns arise. Summary covers the top architectural findings only.

#### [B] Staff Engineer
Focuses on: performance, code quality, testability, maintainability, key trade-offs, risks.

Annotates inline wherever performance hotspots, quality gaps, or risk blind spots appear. Summary covers the top quality/risk findings only.

#### [C] Engineering Manager
Focuses on: does the plan solve the real problem? Does it deliver proportional value to users and the project?

Annotates inline wherever scope, effort, or alignment with user needs is in question. Summary covers value vs effort and any sequencing concerns.

#### Collecting results

1. Each subagent returns an annotated version of the full file plus a `---SUMMARY---` block.
2. Merge all three sets of inline comments into the draft file. Where multiple personas comment on the same location, stack their blockquotes in A → B → C order.
3. Append a `## Specialist Reviews` section to the draft file containing only the three summaries:

```markdown
## Specialist Reviews

### [A] Architecture Engineer
- <bullet from summary>

### [B] Staff Engineer
- <bullet from summary>

### [C] Engineering Manager
- <bullet from summary>
```

---

### Stage 4 — Final Plan → `<DATE>_<NAME>_03_PLAN.md`

Spawn a **single subagent with `model: opus`** to produce the final plan. This is a senior architect synthesizing everything — it must read the fully annotated draft (inline comments + summaries) and produce the best possible implementation approach, not a mechanical merge.

#### Subagent prompt must include

- Absolute path to the annotated draft file (read it first)
- The context name and original user request
- The instructions below

#### Subagent instructions

1. Read every inline `> **[A/B/C]**` comment in the draft. Treat them as expert input, not directives — where reviewers conflict, choose the approach with the better risk/value tradeoff and explain why.
2. For each task, synthesize the best implementation approach considering all reviewer angles (architecture soundness, code quality, and user value). Don't average opinions — pick the strongest path and justify it in **Review notes**.
3. Prefer the more conservative risk posture when uncertain.
4. Preserve deferred but valuable items in a `## Future` section.
5. Write the final plan file at the path provided, using the structure below.

#### Output file structure

```markdown
# Technical Plan: <context name>
_Date: <DATE value captured at invocation>_

## Objective
[refined from draft, sharpened by reviewer input]

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
**Approach:** [best synthesized approach — concrete and actionable]
**Acceptance criteria:** ...
**Review notes:** [which reviewer inputs shaped this task and how conflicts were resolved]

[repeat]

## Execution order
[finalized with rationale from architectural and risk considerations]

## Risks & mitigations
[consolidated from all three reviewers, deduplicated, ordered by severity]

## Future / out of scope
[deferred but valuable items worth tracking]

## Review delta
[2–4 sentences on what changed most from draft to final, which reviewer had the most impact, and why]
```

---

## Final report to user

After all stages complete, output:
- Paths of the 3 files created
- Task count in the final plan
- Top risk and its mitigation
- One-sentence summary of the most impactful specialist feedback
