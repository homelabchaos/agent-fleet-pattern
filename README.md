# Agent Fleet Pattern

How a "chief of staff + standing specialist lanes" setup for Claude Code is built,
how it matures, and how the pieces talk to each other — written as a pattern, not a
product. Nothing here is specific to any one environment; the point is what
transfers when you move the same idea from an always-on personal server to a
single-operator corporate laptop.

## Read in order

| File | What it covers |
|---|---|
| [01-concept.md](01-concept.md) | The five bones of the pattern and the sources it comes from |
| [02-substrate-and-deviations.md](02-substrate-and-deviations.md) | Where the same concept lands differently depending on the floor it runs on |
| [03-lifecycle.md](03-lifecycle.md) | How a lane is born, broken in, kept honest, and allowed to mature |
| [04-agents-and-skills-primer.md](04-agents-and-skills-primer.md) | Writing an agent vs writing a skill, with one worked example of each |
| [05-chief-of-staff-architecture.md](05-chief-of-staff-architecture.md) | The reference architecture, sanitized from a running setup |

## Attribution

The underlying ideas — an above-the-loop router, standing agents with job statements,
receipts instead of self-reported success, a maintenance loop with explicit
keep/change/pause/retire decisions, and "session-to-skill" extraction as a draft-only
step — come from Nate B Jones' paid newsletter and kits
(https://natesnewsletter.substack.com). None of his text is reproduced here; this is
one implementation's notes on applying those ideas. If something here reads as
insight, assume it's his; if it reads as a mistake, assume it's ours.

Claude Code agent/skill syntax: https://docs.claude.com/en/docs/claude-code

## What is deliberately not here

- Any hostnames, addresses, domains, credentials, or names of people.
- The actual lane definitions, rules, or board contents from the setup this was
  written from — only their *shape*.
- Anything from the source material itself.

## License

Prose in this repository: CC BY 4.0. Code snippets: MIT.
