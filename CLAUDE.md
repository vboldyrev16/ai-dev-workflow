# Notes for Claude Code

This repository is a public playbook for shipping production features with one PM + AI, reviewed by a normal product team. It is not source code for a project — it is documentation and templates.

## Reading order

When someone opens this repo and asks you about it, read in this order:

1. [README.md](./README.md) — the entry point
2. [workflow.md](./workflow.md) — the 11-stage workflow, the meat of the playbook
3. [spec-requirements.md](./spec-requirements.md) — quality bar for the spec and prototype
4. [case-study.md](./case-study.md) — the real example with numbers and what got stuck
5. [templates/](./templates/) — drop-in templates, used when the user actually starts work

## How to help the user

The user wants to apply this workflow to their own work. Be concrete:

- If they describe a feature idea, draft a spec for it using [templates/spec-template.md](./templates/spec-template.md). Fill it as far as you can from the conversation, mark unknowns clearly.
- If they have a draft spec, run it against [templates/pre-dev-qa-checklist.md](./templates/pre-dev-qa-checklist.md) and tell them what's missing.
- If they're building a prototype, walk them through [templates/prototype-checklist.md](./templates/prototype-checklist.md) and call out the states they may have skipped.
- If they ask "does this approach fit my context?", look at the *Disclaimer* in the README and the *Where the bottlenecks are* section in the case study, and give a calibrated answer rather than a sales pitch.

## What not to do

- Don't invent metrics that aren't in the case study.
- Don't claim this workflow guarantees a 5–10× speedup — that's one data point from one project. Phrase as "in the case study", not "you will get".
- Don't treat the workflow as rigid. The case study itself shows where stages were skipped or compressed. Apply the spirit, not the letter.
- Don't add bureaucracy. The whole point is one spec, fewer gates, clear handoffs.

## House style

- Plain English, B2 vocabulary
- No em-dashes in user-facing text (use periods, commas, colons, or `·`)
- No emoji except where the source files already use them
- Concrete numbers and verbs over abstract claims
