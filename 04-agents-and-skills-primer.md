# Writing an Agent and Writing a Skill (Claude Code)
### A short primer with one worked example of each

*Written from a working setup (Aug–Sep 2026). Examples are generic/work-shaped on purpose.
Official reference: https://docs.claude.com/en/docs/claude-code — read it for current
syntax; this primer is about the *shape* of a good one, not the spec.*

---

## The one-line distinction

| | **Agent** (`.claude/agents/<name>.md`) | **Skill** (`.claude/skills/<name>/SKILL.md`) |
|---|---|---|
| What it is | A *persona with grounding* — who it is, what it must read first, what it must never do | A *procedure* — do these steps in this order, report in this shape |
| Invoked by | The main session delegating a task (`Agent` tool), or the user opening it as a standing session | `/<name>` by the user, or auto-loaded by the model when the description matches (unless disabled) |
| Lives for | A conversation (or months, if it's a standing session) | One run |
| Good test | "Would I trust this to *answer* a question in its domain?" | "Could a new hire follow this without asking anything?" |

Rule of thumb: **an agent knows; a skill does.** If you're writing a lot of "then run X, then check Y" in an agent, it's a skill. If you're writing "never assume Z, read doc W first" in a skill, it's an agent.

---

## Anatomy of an agent

```markdown
---
name: azure-monitoring
description: Azure monitoring lane — Log Analytics, alerts, action groups, workbook health.
  Use for ANY Azure-monitoring question instead of answering from general knowledge.
tools: Bash, Read, Grep, Glob, WebFetch
---

You are the Azure monitoring specialist for this team.

## Grounding — MANDATORY before answering
Read, in order:
1. `docs/monitoring/CURRENT-STATE.md`   — what exists today (as-built)
2. `docs/monitoring/decisions.md`       — why it's shaped that way
3. Live state via `az monitor …`        — never quote the doc's *values*; query them

## Hard rules (each traceable to a real failure)
- Never declare an alert rule "missing" from the doc alone — list it live first.
  (2026-03: we "re-created" a rule that existed under a different name.)
- Never change a threshold in the same turn you diagnose it — propose, then wait.
- Verify from the consumer's side: an alert "fires" only if the action group delivered.

## Scope
Owns: workspaces, DCRs, alert rules, action groups, workbooks.
Does NOT own: app code, cost decisions (route to the platform owner).

## Escalation
Anything irreversible, cross-team, or touching credentials → stop, present options,
name the blast radius, wait.
```

What makes it good:

1. **Description says "use for ANY X question instead of answering from general knowledge."** That sentence is what makes the router actually delegate. Vague descriptions never get picked.
2. **Grounding is ordered and mandatory.** As-built doc → decision doc → live values. The most common agent failure is answering from training data about a system it has never looked at.
3. **Every hard rule cites the failure that produced it.** Rules without a story accrete forever and nobody can tell later which are dead weight. Rules with a story can be reviewed and retired.
4. **Scope has a "does NOT own" line.** Otherwise it will helpfully answer everything.
5. **`tools:` is the minimum.** A read-only advisor gets no `Write`/`Edit`.

Anti-patterns: a 300-line agent (it's a doc, link to it); rules that babysit an old model's weakness (review when the model changes); personality fluff.

---

## Anatomy of a skill

```markdown
---
name: monitoring-brief
description: Morning briefing for the Azure-monitoring lane — live state, open threads,
  unprompted status.
disable-model-invocation: true      # user runs /monitoring-brief; the model doesn't self-trigger
allowed-tools: Bash, Read, Grep
---

# Monitoring lane — session brief

## Do, in order
1. `az monitor metrics alert list -o table` — note any rule disabled or in error state.
2. `az monitor action-group list -o table` — confirm the on-call group still has ≥1 receiver.
3. Read `docs/monitoring/OPEN-THREADS.md` — pull items tagged `monitoring`.
4. Check the last 24h of fired alerts for anything that fired ≥3× (flapping candidate).

## Report (post this unprompted, ≤15 lines)
- **State:** OK / DEGRADED (+ one line why)
- **Fired last 24h:** count, top 3 by frequency
- **Open threads:** each with status and who it's blocked on
- **Proposed next action:** one, or "none"

## Rules
- Don't fix anything during the brief. Report, then wait.
- Every "X is broken" claim must come from a command run *this session*, not from the doc.
- If a command fails, say so and continue — a partial brief beats no brief.
```

What makes it good:

1. **Numbered steps with the literal commands.** A skill that says "check the alerts" is a suggestion, not a skill.
2. **A report template.** The output shape is the contract; it's what makes runs comparable week to week.
3. **`disable-model-invocation: true` for anything with side effects or cost.** Let the human pull the trigger; let the model auto-load only reference-style skills.
4. **`allowed-tools` fenced to what the steps need.**
5. **"Don't fix during the brief."** Separate *observe* from *act* — the number-one way a helper turns into an incident.

Anti-patterns: a skill that's really a prompt ("be thorough"); no report shape; steps that depend on state the skill didn't check.

---

## How they fit together (the 30-second version)

- Each recurring domain gets **one agent + one brief skill**. The agent is the standing
  expert; the skill is what it runs when it wakes up or when a monitor pokes it.
- A **router session** ("chief") delegates to agents and holds decisions. See
  `chief-of-staff-architecture.md` alongside this file.
- New skills and rule changes are **proposed, never self-installed** — an agent may draft
  one into a review folder with a header saying who/when/why; a human merges it.

## Start here if you have an afternoon

1. Pick the one domain you already answer questions about every week.
2. Write its agent: grounding list + three hard rules with their stories + a scope line.
3. Write its brief skill: 3–5 commands, a report template.
4. Run the brief three mornings in a row. Delete whatever you didn't read.
