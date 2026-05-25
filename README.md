# AI Development Workflow

A spec-first workflow for AI-assisted development at companies that ship to real users. Built around one principle: **if the spec is precise enough, AI-written code passes review like any other code.** Without that discipline, AI code is unreviewable and unsafe to ship.

The playbook is extracted from a real case where the workflow ran to its extreme: one PM with AI built a microservice in 38 days, reviewed and hardened by the same core team that would otherwise have built it. That's ~10× less core-team capacity than the traditional estimate for the same scope. The 1-PM build is a stress-test of the workflow, not the recommended default — most teams will apply this with developers using AI as a pair-programmer, and the gates work the same way.

> If you came here from a talk or a tweet — this is the source. Read it top-to-bottom, fork it, or point Claude Code at this repo and ask it to walk you through.

**Watch the 20-slide talk:** https://ai-dev-workflow-lake.vercel.app
**See the prototype the case study built around:** https://motors-analytics-demo.vercel.app

## What's inside

| File | What it is |
| --- | --- |
| [workflow.md](./workflow.md) | The 11-stage workflow. Pre-Dev gates before coding, Post-Dev gates where reviewers fix the code themselves. |
| [spec-requirements.md](./spec-requirements.md) | Quality bar for the spec and the prototype. Sections, principles, anti-patterns. |
| [when-to-use.md](./when-to-use.md) | Who this is really for. When the workflow fits, when it doesn't, what modes it supports. |
| [case-study.md](./case-study.md) | The extreme example: 1 PM + AI, 38 days, with what worked and what got stuck. |
| [templates/spec-template.md](./templates/spec-template.md) | Drop-in spec template with all required sections. |
| [templates/prototype-checklist.md](./templates/prototype-checklist.md) | Checklist for prototype handoff. |
| [templates/pre-dev-qa-checklist.md](./templates/pre-dev-qa-checklist.md) | Pre-Dev QA Check gate checklist. |
| [examples/motors-analytics-spec.md](./examples/motors-analytics-spec.md) | The actual spec from the case study, anonymized. 16 sections, real formulas, decision log. |
| [presentation/](./presentation/) | The 20-slide talk that introduces the playbook (HTML, single file). |
| [CLAUDE.md](./CLAUDE.md) | Reading order and usage notes for Claude Code. |
| [llms.txt](./llms.txt) | Index for AI crawlers. |
| [CONTACT.md](./CONTACT.md) | Reach out — LinkedIn, email, Telegram. |

## TL;DR — the 11 stages

```
Backlog → Research & Spec → Design
       → Pre-Dev: QA Check → Pre-Dev: Tech Review
       → Development
       → Post-Dev: Design Review → Post-Dev: Code Review
       → QA Testing → Deploy → Done
```

Three ideas do most of the work:

1. **Spec = single source of truth.** Not a PRD, an AC doc, and a UX doc. One document. Everyone reads it. If it isn't in the spec, it doesn't exist. This matters more under AI development than under traditional development: AI treats the spec as ground truth and fills the gaps with guesses. A tight spec means fewer guesses and shippable code.
2. **Pre-Dev gates do most of the safety work.** QA validates the spec before any code is written. Tech Review validates the architecture. These two gates are why AI-written code can pass Post-Dev review at the same bar as human-written code — most bugs and architecture issues are caught before they exist.
3. **Reviewers fix the code themselves.** No bounce-back to the PM for technical issues. The PM only gets product questions (logic, spec, AC). This works the same whether the PM is the one prompting AI or a developer is.

The full reasoning, gates, and what each role owns is in [workflow.md](./workflow.md).

## Who this is for

**Built for**: teams shipping AI-assisted features into a real product with users, where broken releases have a cost. The whole point of the pre-dev gates and the spec quality bar is to make AI-written code safe to ship at that scale.

- **Engineering leaders and CTOs** evaluating how to roll out AI-assisted development without lowering the production bar
- **Senior developers using Claude Code** who want a structured way to prepare tasks so the AI session is productive rather than "easier to write by hand"
- **PMs with technical depth** in companies where they can build alongside the team
- **QA, designers, and reviewers** working with AI-assisted code, wondering how their role and the gates change

**Also useful for**: solo builders and prototypes. The principles transfer. Just know that the playbook's center of gravity is teams with production code, not greenfield solo projects where the risk profile is different.

See [when-to-use.md](./when-to-use.md) for the honest fit assessment.

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

## Author

Built and maintained by **Vladimir Boldyrev**.

- LinkedIn: [linkedin.com/in/vboldyrev16](https://www.linkedin.com/in/vboldyrev16/)
- Email: v.boldyrev16@gmail.com
- Telegram (DM): [@vboldyrev](https://t.me/vboldyrev)
- Telegram channel: [@vovacoder](https://t.me/vovacoder) — ongoing notes on AI-assisted product engineering

Questions about applying the workflow to your team, feedback after trying it, or just want to chat about AI dev? Any channel above works. See [CONTACT.md](./CONTACT.md) for context on what I'm happy to help with.

## Disclaimer

This is one team's working approach, not a standard. The case study ran the workflow at its extreme — one PM, one new microservice, one company. The workflow itself generalises further than that case (see [when-to-use.md](./when-to-use.md)), but every team has its own constraints. Some parts will transfer cleanly. Some won't. The case study is honest about what got stuck.
