# AI Development Workflow

A public playbook for shipping production features with one PM + AI, reviewed by a normal product team. Extracted from a real case: an analytics dashboard for ~250 dealers on a classifieds marketplace, shipped in 38 days for ~10× less core-team capacity than the traditional estimate.

> If you came here from a talk or a tweet — this is the source. Read it top-to-bottom, fork it, or point Claude Code at this repo and ask it to walk you through.

## What's inside

| File | What it is |
| --- | --- |
| [workflow.md](./workflow.md) | The 11-stage workflow. Pre-Dev gates before coding, Post-Dev gates where reviewers fix the code themselves. |
| [spec-requirements.md](./spec-requirements.md) | Quality bar for the spec and the prototype. Sections, principles, anti-patterns. |
| [case-study.md](./case-study.md) | The numbers, the timeline, what worked, what got stuck. |
| [templates/spec-template.md](./templates/spec-template.md) | Drop-in spec template with all required sections. |
| [templates/prototype-checklist.md](./templates/prototype-checklist.md) | Checklist for prototype handoff. |
| [templates/pre-dev-qa-checklist.md](./templates/pre-dev-qa-checklist.md) | Pre-Dev QA Check gate checklist. |
| [CLAUDE.md](./CLAUDE.md) | Reading order and usage notes for Claude Code. |
| [llms.txt](./llms.txt) | Index for AI crawlers. |

## TL;DR — the 11 stages

```
Backlog → Research & Spec → Design
       → Pre-Dev: QA Check → Pre-Dev: Tech Review
       → Development
       → Post-Dev: Design Review → Post-Dev: Code Review
       → QA Testing → Deploy → Done
```

Two ideas do most of the work:

1. **Spec = single source of truth.** Not a PRD, an AC doc, and a UX doc. One document. Everyone reads it. If it isn't in the spec, it doesn't exist.
2. **Reviewers fix the code themselves.** No bounce-back to the PM for technical issues. The PM only gets product questions (logic, spec, AC).

The full reasoning, gates, and what each role owns is in [workflow.md](./workflow.md).

## Who this is for

- **PMs with technical depth** who want to ship without waiting on a dev team
- **Engineering leaders** evaluating how AI-assisted development fits into an existing org
- **Designers, QA, devs** working alongside an AI-building PM and wondering how their role changes
- **Anyone curious how a 38-day, 1 PM + AI build actually plays out** when reviewed by a real team

## How to use this with Claude Code

Clone the repo and open it in Claude Code:

```bash
git clone https://github.com/vboldyrev16/ai-dev-workflow.git
cd ai-dev-workflow
claude
```

Then:

- "Walk me through the workflow as if I'm a PM about to start a new feature."
- "Use the spec template to draft a spec for this idea: …"
- "Run my draft spec against the Pre-Dev QA checklist and tell me what's missing."

Claude reads [CLAUDE.md](./CLAUDE.md) automatically and knows the right reading order.

## How to use this without Claude Code

Read [workflow.md](./workflow.md), then [spec-requirements.md](./spec-requirements.md), then [case-study.md](./case-study.md). The templates are drop-in.

## License

[MIT](./LICENSE). Use it, adapt it, ship from it.

## Disclaimer

This is one team's working approach, not a standard. It worked for one PM with a technical background, building one product (an analytics dashboard), in one company. Some parts will transfer cleanly. Some won't. The case study is honest about what got stuck.
