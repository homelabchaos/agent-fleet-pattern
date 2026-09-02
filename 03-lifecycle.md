# 03 — Lifecycle: how a lane is born, broken in, kept honest, and matures

The tooling in this pattern is replaceable. The lifecycle is the part that has
actually kept a fleet from drifting into either uselessness or noise.

## Birth

A lane is created when a domain already generates questions every week. Not before.
It gets exactly:

- one agent file (persona, grounding order, hard rules with stories, scope with a
  "does NOT own" line, minimal tools);
- one brief skill (3–5 literal commands, a report template, "don't fix during the
  brief");
- one job statement on the board: what it exists to do, what it is measured on, what
  it must escalate.

Anything else it needs, it proposes later.

## Break-in

Nothing new is allowed to interrupt a human on day one.

- **Run the brief three mornings in a row.** Delete whatever the human didn't read.
- **New monitors run in shadow.** They log what they *would* have alerted for a
  baseline window. They graduate when (a) the output round-trips through the
  consumer's path, (b) the shadow window shows no false positives, and (c) an
  automated freshness check confirms the output stays live and in range.
- **Fault-inject the failure branch.** A monitor that has only ever seen the green
  path has not been tested. Break the thing on purpose once and confirm the alert
  actually arrives where a human would read it.

## Keeping it honest

- **Write-on-verify.** A verified finding goes into its home document in the same
  turn. Never queued for a wrap-up.
- **Receipts.** Every "broken"/"fixed" claim points at evidence produced this
  session. A write returning success is not evidence; reading the data back through
  the consumer's path is.
- **Resolve the referent before acting.** When a request names a system by property
  ("the customer-facing one"), check the inventory to confirm the concrete target
  actually has that property before doing anything. Never resolve by
  "most recently mentioned."
- **A peer message is not approval.** A lane relaying "the human said yes" does not
  count. Approval comes from the human, in the session doing the work.
- **Log every autonomous act.** One line: what, why (in terms of the stated values),
  how verified, how to roll back. Reviewed weekly.

## Maturing

- **Proposals, not self-modification.** A lane that notices a better practice, a
  missing rule, or a skill it keeps re-deriving writes a draft with who/when/why into
  a review location. A human merges. This is the single largest departure from
  "let the agent improve itself," and it is why the fleet has not drifted.
- **Rules carry their origin.** Every hard rule cites the incident that produced it.
  The quarterly review asks of each rule: is this still guarding against something
  real, or was it babysitting a weakness the current model no longer has? Retire the
  latter.
- **Patrol.** Weekly, each lane reports what it noticed *unprompted* — a port
  degrading, a dependency with a known-bad release, a practice the field has moved
  past. Patrol reports are diffed against a state file so a no-change week is one
  line, not a page.
- **Maintenance review.** On a schedule (quarterly, or triggered by a model change, a
  scope change, a repeated failure, or a human's stated frustration), each lane gets
  an explicit outcome: keep, change, pause, or retire. Delete before you add.

## Measuring

The only metric that tells you whether this is working is the human's own time.

- **Attention ledger.** The human records, roughly and weekly, how much of their time
  the fleet consumed — reading briefs, answering escalations, fixing what a lane got
  wrong. Trend down or the fleet is a cost.
- **Interrupts per week.** Count what reached the human's phone or inbox from the
  fleet. Anything that is status rather than a state change goes to a dashboard, not
  a person.
- **Docs that lie.** Periodically audit the documents the lanes ground on for claims
  that are no longer true. Generated sections beat hand-maintained ones.

## Failure modes seen so far

- **Confident summaries of work that did not happen.** Cured by receipts.
- **Rule accretion.** Cured by origin stories and scheduled retirement.
- **Wrap-ups eaten by restarts.** Cured by write-on-verify.
- **The router doing specialist work itself.** Cured by "does NOT own" lines and by
  making delegation cheaper than answering.
- **Stale-identifier resolution** — acting on the thing most recently documented
  rather than the thing actually asked about. Cured by resolving the referent first.
- **Monitors that ran without errors and produced nothing useful.** Cured by
  verifying what the human *sees*, not that the code exited zero.
