# 01 — The concept

## Where it comes from

This is not an original design. It is an implementation of ideas Nate B Jones has
been publishing through 2026 (see README for attribution): a human sitting *above*
the loop rather than in it; standing agents with written job statements; receipts
instead of self-reported success; a maintenance loop with explicit
keep/change/pause/retire outcomes; and skills extracted from sessions as *drafts*
that a human promotes.

What we added was mostly the connective tissue — how the pieces run, resume, and
talk — and the discipline of writing down why each piece exists so it can later be
removed.

## The five bones

**1. One router session, called the chief.**
It holds decisions, context, and the human's attention. It does *not* do specialist
work. When a question belongs to a domain, the chief delegates to that domain's lane
and relays the answer. When something cuts across domains, the chief is the only
place the pieces meet.

**2. Standing domain lanes.**
Each recurring domain gets exactly two artifacts:
- an **agent** — persona, mandatory grounding order, hard rules that each cite the
  failure that produced them, a scope with a "does NOT own" line, minimal tools;
- a **brief skill** — the numbered procedure it runs when it wakes up or is poked,
  ending in a fixed report shape.

"An agent knows; a skill does." If you are writing "then run X, then check Y" in an
agent, it's a skill. If you are writing "never assume Z, read W first" in a skill,
it's an agent.

**3. A shared board.**
Lanes post state *changes* (not status) to one board. The chief reads a diff of the
board on a schedule instead of interrogating every lane. The board is the single
source of truth for "what is open, who owns it, what is it blocked on." Anything a
lane learns that the chief would need is written there the moment it's verified.

**4. Receipts.**
Any claim of the form "X is broken" or "I fixed X" must point at evidence produced
in *this* session — a command's output, a read-back through the consumer's path —
never at the documentation. A write that returned success is not a receipt; reading
the data back through whatever consumes it is. This rule exists because a model
judging its own success is measurably worse than a coin flip, and because the
cheapest failure to produce is a confident summary of work that did not happen.

**5. A maturity loop with a human gate.**
Lanes may *propose* — a new skill, a rule change, a retirement — by writing a draft
with who/when/why into a review folder. A human merges. Lanes never self-modify.
Rules carry their origin story so a later review can tell which ones were babysitting
an old model's weakness and retire them.

## What holds it together

- **Write-on-verify.** A verified finding goes into its home document in the same turn
  it's verified. Never queued for an end-of-session wrap-up: wrap-ups are the first
  thing lost to a reboot, a closed laptop, or a context window filling up.
- **Job statements.** Each lane has a one-paragraph statement of what it exists to
  do, what it is measured on, and what it must escalate. This is what the chief uses
  to route, and what the maintenance review uses to decide keep/change/pause/retire.
- **A reversibility test for autonomy.** The line for "may act without asking" is not
  *what* is touched but *whether it can be undone*. Reversible (config with a backup,
  a restart, code in git): the lane may act and must log it. Irreversible (deletion,
  credential changes, anything outward-facing that cannot be unsent): ask, always.
- **An attention ledger.** The human records roughly how much of their own time the
  fleet consumed each week. If that number is not falling, the fleet is not working,
  regardless of how busy it looks.
