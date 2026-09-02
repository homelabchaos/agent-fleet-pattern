# 02 — Substrate and deviations

The five bones in `01-concept.md` are floor-independent. Almost everything that
*looks* like a design difference between two implementations is actually the floor
showing through. This file names the two floors we have thought about and what each
bone becomes on each.

## The two floors

**Floor A — always-on personal host.**
A Linux server that never sleeps, owned by one person with root. Sessions can be
long-lived processes in a terminal multiplexer. Scheduled jobs exist. Anything can
be scripted. There is a meaningful distinction between "production" (other people
depend on it) and "lab" (only the operator does), and the lab zone tolerates
mistakes as tuition.

**Floor B — single-operator corporate laptop.**
A managed endpoint that sleeps, with the assistant running as a CLI inside an
editor. No root, no daemons, no scheduled jobs the operator controls. Source lives
on an enterprise git host with pull-request review. Authority to change anything
real goes through privileged-access management and change control. Everything is
production.

## How each bone lands

| Bone | Floor A | Floor B | What actually changed |
|---|---|---|---|
| **Standing lanes** | Named sessions running 24/7, spawned at boot, resumed by name across reboots | Named sessions that are *re-openable* — resumed on demand, not running | "Standing" becomes "durable." The lane's memory lives in its transcript and its docs, not in a live process. |
| **Events reaching a lane** | Monitors push: a timer injects "check this" into the running session | Pull: the brief skill queries the sources when the lane is opened | Same information, opposite direction. Floor B's brief must enumerate its sources explicitly because nothing will come to it. |
| **Chief ↔ lane messaging** | Native session-to-session messages between concurrent sessions | File handoff through the board; the chief reads it when opened | Concurrency is what enables messaging. Without it, the board *is* the message bus — which is fine, and more auditable. |
| **Shared board** | A small database behind a tool, regenerated to markdown daily, diff injected into the chief on a schedule | A markdown file in the repo, edited by lanes, reviewed by the chief at open | Lower tech on Floor B. Arguably better: every board change is a commit with an author. |
| **Receipts** | Commands run in-session; alerts verified from the consumer's side (what the phone showed, what the dashboard rendered) | Identical rule; consumers are the monitoring platform, the ticket system, the SIEM | Concept unchanged. Only the list of consumer paths differs. |
| **Proposals channel** | A drafts folder; the human moves accepted drafts into place | A pull request with a required reviewer | Floor B's version is the stronger governance and is what the organization already trusts. Use it. |
| **Authority** | A personal contract: reversible → act and log; irreversible → ask | Institutional: the lane never holds standing write authority. It drafts; a human with elevated access executes | On Floor B the reversibility test still decides *what to draft*, but never grants execution. |
| **Blast-radius framing** | Production vs lab zones; lab breakage is tuition | Everything is production | With no lab zone, the autonomy contract collapses to "observe and propose" until an explicit sandbox exists. Building that sandbox is the first real infrastructure task. |
| **Rule origins** | Each hard rule cites the incident that produced it | Same — cite the ticket or post-incident review | Unchanged. If anything, Floor B has better incident records to cite. |
| **Data fences** | Path deny-lists, an egress guard on third-party tools, local-only domains | Same mechanisms; the classification scheme already exists | The *mechanism* transfers directly. Reuse the organization's data classification rather than inventing one. |
| **Personal / non-utilitarian features** | Lanes for personal domains; a deliberately playful alert channel | Not applicable | Don't port. Worth noting only that a fleet can carry a feature that exists for no reason but enjoyment, and that is not a defect. |

## The three deviations that matter

Everything above reduces to three real differences:

1. **Concurrency.** Floor A has it; Floor B mostly doesn't. Concurrency is what makes
   "standing," "push," and "messaging" possible. Without it, design around durable
   state (transcripts, board, docs) and make every lane's brief self-sufficient.

2. **Authority.** Floor A can grant a lane a bounded right to act. Floor B cannot and
   should not try; the lane drafts, the human executes through the existing
   controls. This is not a weaker version of the pattern — it is the pattern with
   the human placed exactly where the organization already requires a human.

3. **A lab zone.** Floor A has a place where a wrong call costs nothing but time.
   Floor B needs one built deliberately (a sandbox subscription, a non-production
   workspace, a test tenant) before any lane is allowed to do more than read.

## What to port first, in order

1. The agent + brief-skill pair for one domain you already answer questions about
   weekly.
2. The board file and the habit of writing to it on verify.
3. The receipts rule, enforced in the brief skill's report template.
4. The proposals channel as a PR flow.
5. Only then: any automation.
