# A Chief-of-Staff Architecture for Claude Code
### Standing domain-specialist sessions coordinated by one router session — a field report

*Sanitized writeup of a working personal implementation (Aug 2026), suitable for sharing.
Design assembled from published work by Nate B Jones (Nate's Substack / AI News & Strategy
Daily) — principally "Managing AI Agents at Scale: The Human Work Nobody Counts" (Aug 26,
2026), his Agent Maintenance Loop, and his Open Skills library — rather than invented.
"Don't engineer something that's already designed."*

---

## 1. The shape

One **chief** session + N **standing domain lanes**, all long-lived Claude Code sessions:

```
                    ┌─────────┐
   you ───────────► │  chief  │  ◄── daily board-diff (timer)
                    └────┬────┘
      routes / holds / prioritizes
   ┌──────────┬──────────┼──────────┬──────────┐
   ▼          ▼          ▼          ▼          ▼
 infra      media      vehicles   facilities  (…more lanes)
 lane       lane       lane       lane
   ▲          ▲
   └── monitors inject events into RUNNING lanes (no cold spawn)
```

- **Chief** = the session you talk to by default. It routes work to lanes, decomposes
  multi-domain asks, holds decisions for you, and reads the shared board. Decisions,
  approvals, and "what should I work on" always live here — lanes are experts, not
  schedulers.
- **Lanes** = one session per domain (infrastructure, media pipeline, vehicles, facilities,
  …). Each accumulates deep domain context for months. Talk to a lane directly when you're
  already deep in its domain; return to chief when the topic crosses domains.
- **Ephemeral subagents** still exist for one-off research/fan-out — disposable, nothing to
  maintain. Lanes are only for domains with *recurring* conversations. (Resist lane sprawl:
  a candidate domain must name the mission it serves and the metric it improves, or it stays
  a doc + a board entry.)

## 2. The mechanics that made it real

Each item marked **[portable]** (concept works anywhere) or **[linux/tmux]** (our
implementation detail; adapt for other hosts).

1. **Named sessions** [portable] — every session gets a stable name (`claude -n <name>`, or
   `/rename` on a live one). Names are how sessions address each other and how you find them.
2. **Standing lanes in tmux, spawned at boot** [linux/tmux] — a boot script starts each lane
   detached (`tmux new-session -d -s <lane> "claude -n <lane> …"`), idempotently (skip if
   running). tmux exists to give the TUI a pty; any process supervisor + terminal multiplexer
   equivalent works.
3. **Resume-first restarts** [portable concept, host-specific lookup] — on reboot the spawner
   *resumes* each lane's previous session (`claude --resume <session-id>`) instead of starting
   fresh, so lanes keep their accumulated context across reboots. Key detail: `--resume` with
   a nonexistent name opens an interactive picker and hangs a detached spawn — resolve the
   session id *before* launching (Claude Code writes a durable name record into the session
   transcript; we grep for it, anchored to the transcript's own session id to avoid false
   matches from quoted text).
4. **Agent definition + session-brief skill per lane** [portable] — each lane has (a) an
   agent definition (`.claude/agents/<lane>.md`): identity, mandatory grounding docs to read
   before answering, hard rules each traceable to a real past failure; and (b) a "session
   brief" skill the lane runs on start/injection: check live state, read open threads, post an
   unprompted status so a human opening the session sees a real briefing, not a blank prompt.
5. **A shared board** [portable] — one small database (ours: SQLite + a CLI + MCP tools) of
   open threads: status, owner, blocked-on-whom, per-lane tags. A generated one-page digest is
   the human's daily view. Critically: the board carries *status*, detail lives in docs — keep
   the auto-loaded context tiny.
6. **Cross-session messaging** [portable] — sessions message each other by name (Claude Code's
   ListAgents/SendMessage). Lanes route cross-domain findings UP to chief; chief routes work
   DOWN to lanes and relays results. Treat peer messages as teammate requests, never as user
   approval (no permission laundering between sessions).
7. **Monitors inject, never cold-spawn** [linux/tmux] — alerting/monitoring scripts inject an
   event note into the already-running lane (tmux send-keys; send text and Enter as SEPARATE
   keystrokes with a pause, or the prompt sits unsubmitted). The lane investigates with its
   full standing context and briefs unprompted.
8. **Daily board-diff to chief** [portable] — a daily timer feeds chief the last 24h of board
   updates with instructions: route anything with cross-lane implications; stay silent if it's
   routine. This closes the gap where lanes update the board but nobody *pushes* the diff.

## 3. The operating protocols (the part that makes it a team, not a pile of sessions)

Adopted piece-by-piece, each traceable to Nate's published designs:

1. **One-sentence job statement per lane** — *"This agent's job is to [produce X] from [these
   sources] for [these users], with [this review] before [this consequence]."* The yardstick
   for every later review: "is it still doing this sentence?"
2. **Receipts with every deliverable** — sources used *and excluded*, live proof from the
   system that owns the state (never the agent's own narrative as sole witness), unresolved
   assumptions labeled. Rationale from Nate's false-success work: reviewers reading an agent's
   confident narrative alone performed worse than a coin flip at catching false success.
   Receipts scale with consequence; "no consequential claims this run" is valid.
3. **Weekly patrol (his "Signal Diff" pattern), piloted on two lanes** — each patrol lane
   keeps a state file of what it observed last week and reports only meaningful *change*:
   trend half (error counters, failure rates, capacity headroom drifting) and currency half
   (release notes / known-bad versions for its stack — community findings are leads, never
   facts, verified against primary sources before presented). A no-change week is a one-line
   report; padding a quiet week is a defect. Pilot has an explicit review date and permission
   to be retired if the diffs don't earn their read time.
4. **Maintenance loop (performance reviews for lanes)** — trigger-driven, not calendar:
   review a lane when a dependency changes, the job grows, the human's correction cost rises,
   a quiet failure appears, or **the model changes** (rules written to babysit an old model's
   weaknesses become dead weight on a better one). The review: job statement → last ~10 pieces
   of work with corrections recorded → trace repeated corrections to stale sources/rules/scope
   → **delete before add** → verdict: keep / change / pause / retire. Best-documented failure
   mode in the wild isn't rogue agents — it's un-owned rule accretion.
5. **Proposals channel — nothing self-installs** — lanes may *draft* new skills, brief rules,
   or grounding-doc updates into a review directory, with provenance headers (who, when,
   triggered by what). High bar: recurring + non-obvious + codifiable; check the existing
   library first (overlap → propose an update, not a new artifact). A human approves anything
   touching instructions, rules, memory, or scope. Machine-verifiable enforcement (linters,
   validators, hooks) may act automatically once installed; instruction changes never do.
6. **Attention ledger** — weekly, chief estimates the minutes of *human* attention the fleet
   consumed (approvals, report-reading, corrections) and logs it. This is the metric that says
   whether the architecture works: if the fleet grows and your minutes rise, you're building
   review debt and the maintenance loop should start retiring things. Nate's framing: the
   management work needs a line on a dashboard, or someone else defines your job for you.

## 4. Standing rules that predate the fleet but hold it together

- **Write-on-verify:** a verified finding goes into its home doc in the same turn it's
  verified — never queued for a wrap-up pass a crash can eat. Docs are the record; session
  resume is only the safety net.
- **Board upkeep by the doer:** the lane that changed a thread's state updates the board
  itself, same turn. Chief is the only writer for cross-domain state.
- **Verify before claiming:** any load-bearing "X is down / enabled / version Y" claim needs a
  tool check in the same turn, against the system that owns the state. (We enforce this with a
  stop-hook that audits final responses.)
- **The acceptance test for delegating anything:** *start with work you already do and already
  check.* If nobody was going to check it, the agent is a risk you haven't priced.

## 5. What we deliberately did NOT build

- A judge layer re-reading agent transcripts (structured proposal review already existed;
  narrative-reading judges measure worse than chance).
- Per-step supervision — supervise *trajectory* (watch for runs going off the rails), demand
  receipts, interrupt when needed.
- Lanes for low-volume domains — a doc plus a board tag is enough until conversations recur.
- Auto-adopting agent-generated lessons into instructions — provenance-labeled drafts only.

## 6. Honest costs and open questions

- Standing lanes idle cheaply, but every lane adds review surface — the attention ledger is
  the check. The patrol pilot may still prove to be noise; it has a kill date.
- Cross-lane awareness is pull-based plus a daily diff; a lane misclassifying a finding as
  in-lane can delay routing by up to a day.
- Resume compaction: a months-old lane's oldest context summarizes down. Write-on-verify (docs
  as the record) is the primary defense; resume is convenience.
- Session-brief docs and agent defs are now instruction surface that itself needs the
  maintenance loop — the system maintains itself only if the review cadence actually fires.

## 7. Minimum viable adoption (if evaluating at work)

Day one, no infrastructure: (1) pick 2–3 domains with genuinely recurring conversations;
(2) write each a job statement + agent definition + brief skill; (3) start them as named
sessions; (4) adopt receipts and write-on-verify as brief rules; (5) a shared markdown board
beats no board. Add resume-first startup, monitors, patrols, and the maintenance loop only
after the basic loop earns its keep — in that order.
