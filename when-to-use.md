# When to Use This Workflow

> The honest answer to "does this fit my situation?". Read this before adopting.

## The core claim

The workflow does one thing well: **it makes AI-assisted code safe to ship into a product with real users.** That's it. The 11 stages, the spec quality bar, the Pre-Dev gates — all of it serves that single goal.

It does not promise:
- That you can fire your developers
- That every feature gets 10× faster
- That AI replaces the need for senior engineering judgement
- That a non-technical PM can ship anything

What it actually delivers: a process discipline that lets AI-written code clear the same review bar as human-written code, without the team losing trust in what's going to production.

## Who this is built for

### Primary audience: teams with production code and real users

If you have a working product with users who pay, complain, or churn — broken releases have a cost. Adopting AI development without rigorous spec preparation and pre-dev gates is how you ship code that nobody can confidently review and nobody can confidently roll back.

This playbook is built for that situation. The spec-first discipline and the two pre-dev gates exist *specifically* to keep AI-assisted development at the same quality bar as the rest of your engineering work.

If you're a CTO, tech lead, or engineering manager at a company like this, the parts that matter to you are:

- [spec-requirements.md](./spec-requirements.md) — what makes a spec actually good enough to drive AI development
- [workflow.md](./workflow.md) — the gates, why each one exists, and what each role owns
- [templates/](./templates/) — drop-in artifacts your team can start using this week

### Secondary audience: senior developers using Claude Code

You already know how to write code. The question is how to prepare a task so that one Claude Code session ships something reviewable, instead of ending in "easier to rewrite by hand". The answer is the same: a tight spec with all states described, formulas explicit, edge cases enumerated, and a clickable prototype.

Use the spec template, run your draft against the Pre-Dev QA checklist, *then* start the AI session. The session will be dramatically more productive.

### Stretch audience: PMs with technical depth

If you can read code, run a debugger, and have a strong sense for system design, the workflow can scale up to letting you build alongside the team — like in the case study. This requires:

- A team that's willing to review your output rather than reject it on principle
- A spec discipline that the team can trust (Pre-Dev: QA Check is your friend)
- Honest acceptance that this is harder than it looks and not every PM will succeed at it

This is the extreme mode. Read the [case study](./case-study.md) and don't generalise from one data point.

## Who this is *not* primarily built for

- **Solo builders shipping greenfield prototypes.** The principles transfer (one spec, clickable prototype, edge cases enumerated), but the risk profile is different. You don't need the same review gates because you don't have users yet. Steal the spec template and the prototype checklist, skip the rest.
- **Teams without code review or testing culture.** The workflow assumes a real engineering org with reviewers who can fix AI-written code in the MR themselves. If you don't have that, the workflow won't work.
- **"Vibe coding" enthusiasts looking for a way to ship without engineering rigor.** The whole playbook is the opposite — it's about adding *more* rigor, not less, so that AI-assisted code is shippable.

## When the workflow fits

| Situation | Fit |
| --- | --- |
| New microservice, well-understood domain, internal team | **Excellent.** This is the case study's profile |
| Feature in an existing codebase, AI as pair-programmer with senior dev | **Excellent.** Spec discipline is what makes the AI session productive |
| Greenfield SaaS being built by a senior PM with team backing | **Good.** Run the case-study mode. Be ready to flex the gates |
| Bug fix in legacy code with poor test coverage | **Poor fit.** AI without tests on legacy code is a recipe for regressions. Add tests first, *then* iterate |
| Pure research / exploration / prototype | **Skip the workflow.** Don't bureaucratise a spike |
| Refactor with no behavior change | **Partial fit.** Skip the prototype and spec-from-scratch. Keep the gates and review discipline |

## Modes of adoption

You don't have to adopt all 11 stages at once. The most valuable subset, in order:

1. **Spec-as-SSOT + Pre-Dev: QA Check.** If you do nothing else, do this. Stop having separate PRDs, AC docs, and UX tickets. Put it all in one spec. Have QA review the spec before code is written. This alone removes most AI-development surprises.
2. **Pre-Dev: Tech Review.** Before AI writes a line of code, the architecture is approved. This is the difference between "we got a working dashboard" and "we got a working dashboard that fits our infrastructure".
3. **Post-Dev: Code Review where the reviewer fixes the code.** Behavior change for the team, not the workflow. Saves the PM from technical iteration loops. Saves reviewer-PM round-trips. Lets the PM stay on product questions.
4. **The full pipeline.** All 11 stages, applied to a meaningful feature.

Most teams should start at level 1 and add upward as they prove value.

## Common failure modes

- **"We adopted the workflow but skipped the spec quality bar."** Result: AI-written code that nobody can review confidently. Roll back. The spec discipline isn't optional — it's the whole point.
- **"We let the PM write specs without QA review."** Result: edge cases caught in QA Testing instead of in writing. The Pre-Dev QA Check exists because spec-author blindness is real.
- **"We tried to fire devs once a PM could build features."** Result: lost institutional knowledge, slower review cycles, team morale collapse. The workflow is about *adding* leverage, not removing people.
- **"We applied this to a refactor of legacy code without tests."** Result: silent regressions. Add a test harness before letting AI touch legacy code.

## A note on the 5–10× headline

The case study reports ~10× less core-team capacity used vs the traditional estimate. That's one data point from one project with a fortunate fit:

- Standalone microservice (no legacy entanglement)
- Well-understood data domain
- PM with deep technical background
- Team willing to review AI-written code seriously

**Do not treat it as a planning multiplier.** Treat it as proof that the workflow *can* work at scale when conditions are right. Your project may see 1.5×, 3×, or 8×. The number is downstream of how well you apply the spec discipline, not a guarantee from adopting the workflow.

The more honest framing: this workflow lets you ship things you weren't going to ship at all. Not "the same work, 10× faster". Things that, under the old constraints (waiting on team capacity, full traditional cycle), would have stayed in the backlog for a quarter.
