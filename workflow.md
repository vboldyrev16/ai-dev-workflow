# The 11-Stage Workflow

## The pipeline

```
Backlog → Research & Spec → Design
       → Pre-Dev: QA Check → Pre-Dev: Tech Review
       → Development
       → Post-Dev: Design Review → Post-Dev: Code Review
       → QA Testing → Deploy → Done
```

Every feature passes through these 11 stages. Two of them — the Pre-Dev gates (QA Check and Tech Review) and the Post-Dev gates (Design Review and Code Review) — are the parts that make AI-built features ship at production quality.

## Core ideas

### 1. Spec = single source of truth

There is no separate PRD, AC document, or UX doc. One spec covers business logic, formulas, acceptance criteria, edge cases, UI behavior, data and sources, and the decision log. If a rule isn't in the spec, it doesn't exist. When the spec changes, the code, design, and tests follow. Not the other way around.

This matters more under AI development than under traditional development. Claude Code treats the spec as ground truth. Anything missing gets filled in with guesses. A tight spec means fewer guesses and fewer bounces from later stages.

The quality bar for the spec lives in [spec-requirements.md](./spec-requirements.md).

### 2. Pre-Dev validates spec and architecture, before coding

Two gates before development:

- **Pre-Dev: QA Check** — QA reads the spec, lists what's missing, makes sure every state is described and every acceptance criterion is testable.
- **Pre-Dev: Tech Review** — CTO and tech leads validate the proposed architecture, integrations, and auth flow.

The cost of getting these wrong shows up at the end of the cycle, when QA finds gaps that should have been caught in the spec, or when the architecture has to be redone after the code is written. Catching them upfront is what makes the speed real.

### 3. Post-Dev: reviewers fix the code themselves

The biggest behavior change in the workflow. In traditional review:

- Reviewer writes 4 comments
- PM reads them, makes 5 commits, writes a long "done on all 4 items" reply
- Reviewer reads, approves

In this workflow:

- Reviewer pushes commits directly to the same branch — refactor, fix, harden
- 0 comments bounced back to the PM
- MR merged

Why this works: the reviewer already knows how to fix the issue. Routing it through the PM only adds latency. The PM only gets the questions that genuinely need a product decision — spec ambiguity, UX trade-offs — through a separate channel (chat, Jira comment), not as a code-review bounce.

This freed the PM from technical iteration loops entirely. It also turned out to be the single most important organizational change in the case study.

---

## The stages

### 1. Backlog

| | |
| --- | --- |
| **Goal** | Capture the idea or feature |
| **Owner** | PM |
| **Output** | A ticket describing what and why |

### 2. Research & Spec

| | |
| --- | --- |
| **Goal** | Understand what to build: analyze data, identify use cases, edge cases, metrics, data sources |
| **Owner** | PM. Data research (analytics warehouse, customer interviews), system analysis, draft spec |
| **Output** | A spec covering logic, formulas, data sources, what we show, what we don't show. A rough functional prototype, without final design |
| **Exit criterion** | Spec covers all use cases. Data verified. Prototype demonstrates the logic |

### 3. Design

| | |
| --- | --- |
| **Goal** | Turn the rough prototype into a final visual. Produce a UI spec for development |
| **Owner** | PM hands the prototype and spec to the designer → iteration → designer returns final design + UI spec → PM updates the prototype to match |
| **Output** | 1) Final working prototype with approved design. 2) UI spec from designer: design-system components, tokens, spacing, responsive breakpoints. 3) Spec updated if the design changed the logic |
| **Participants** | PM + designer |
| **Quality bar for the prototype** | Clickable, realistic mock data, all states (empty, error, edge), deployed to a URL, approved by the designer. See [spec-requirements.md](./spec-requirements.md) |
| **Exit criterion** | Designer approves. Prototype is deployed. UI spec is ready. Spec is current |

### 4. Pre-Dev: QA Check

| | |
| --- | --- |
| **Goal** | Validate that the spec and prototype are complete, *before* development |
| **Owner** | PM runs the spec through a structured `task-review` exercise with QA. PM prepares acceptance criteria and edge values |
| **Output** | QA confirms the spec is complete, use cases covered, ACs understandable. Comments are folded back into the spec |
| **What QA checks** | All states described (empty, error, edge cases). ACs are clear. "Out of scope" is explicit. Enough information for test design |
| **Exit criterion** | task-review passed. QA signs off on the spec |

### 5. Pre-Dev: Tech Review

| | |
| --- | --- |
| **Goal** | Validate the technical architecture: stack, integrations with existing services, auth, deploy |
| **Owner** | PM proposes an architecture sketch (data flow, API contracts, auth, deploy). Team adjusts or proposes an alternative |
| **Output** | Approved architecture. Spec finalized including technical decisions |
| **Participants** | CTO, backend lead, frontend lead |
| **What they review** | Stack, interaction with existing services, API format, auth flow, deploy strategy, alignment with infrastructure standards |
| **Exit criterion** | Written sign-off from CTO |

### 6. Development

| | |
| --- | --- |
| **Goal** | Build the working product to the approved spec and architecture |
| **Owner** | PM + Claude Code |
| **Output** | Working code (frontend + backend), unit and integration tests. Self-test passed (spec checklist, screenshots) |
| **Participants** | PM. Assigned developers (backend / frontend) are *available on demand* during development for inline questions on architecture and integration, MR-flow help (branch naming, CI, pipelines), and light "are we doing the right thing" sanity checks on bigger tasks. This is availability, not a gate. It does not replace Post-Dev Code Review |
| **Exit criterion** | Code runs locally, tests pass, self-test confirms the spec checklist |

### 7. Post-Dev: Design Review

| | |
| --- | --- |
| **Goal** | Designer checks that the implementation matches the approved prototype |
| **Owner** | PM deploys to staging or sends screenshots / recordings for comparison |
| **Output** | Designer confirms visual fidelity. Issues fixed |
| **Participants** | Designer |
| **What they review** | Padding, fonts, colors, responsive behavior, states (empty, error, hover) |
| **Exit criterion** | "Design OK" from designer |
| **If rejected** | Back to Development |

### 8. Post-Dev: Code Review

| | |
| --- | --- |
| **Goal** | Bring code quality to a stable state through the reviewers. PM is freed from technical iteration loops |
| **Owner** | PM creates the MR with a description |
| **Output** | Reviewers *fix and refactor the issues themselves* directly in the same branch / MR and send the task on to QA. The MR is not bounced back to the PM with comments |
| **Participants** | Assigned reviewers — backend and frontend, by stack |
| **What they review and fix themselves** | Code standards, security, performance, refactoring, code smells, minor bugs, technical simplifications, integration with infrastructure |
| **What they don't touch** | Business logic and spec — that belongs to the PM. Contentious product calls are sent to the PM as a separate question, not as a code edit |
| **Why** | The reviewer already knows how to fix it. Looping through the PM on technical issues only adds latency |
| **Exit criterion** | 2 approvals on the MR, reviewer fixes merged, MR is stable and ready for QA |
| **What goes to the PM** | Only product questions: divergence from the spec, AC ambiguity, contentious UX. Through a separate channel (chat, ticket comment), not as a returned MR |

### 9. QA Testing

| | |
| --- | --- |
| **Goal** | Formal testing of the finished product |
| **Owner** | QA — test design and execution. ACs were approved at stage 4 |
| **Output** | QA sign-off. Any bugs go back to Post-Dev: Code Review — same reviewers / devs fix them, not the PM |
| **Participants** | QA engineer |
| **Exit criterion** | QA sign-off |
| **If bugs** | Back to Post-Dev: Code Review → back to QA Testing |
| **What goes to the PM** | Questions about the requirements themselves: divergence from expected behavior, spec ambiguity, mismatch between spec and UI. The fixes belong to the developers |

### 10. Deploy

| | |
| --- | --- |
| **Goal** | Ship to production via staging |
| **Owner** | Assigned developers + devops |
| **Output** | Product available to users |
| **Participants** | Assigned developers + devops. PM does not participate in the deploy itself, only answers business questions if they come up |
| **Steps** | 1) Deploy to staging/sandbox → 2) Smoke-test → 3) Deploy to production → 4) Verify in production |
| **Exit criterion** | Works in production, auth works, data renders |

### 11. Done

| | |
| --- | --- |
| **Goal** | Confirmed: the product is in production, working, ready for users |
| **Output** | Product ready for users. Sales / distribution can start |

---

## Pre-Dev / Post-Dev — clean split

- **Pre-Dev**: QA Check (spec completeness) + Tech Review (architecture) → *before* development
- **Post-Dev**: Design Review (visual fidelity) + Code Review (code quality, reviewer fixes it themselves) → *after* development

This split means:

- QA validates the spec early, not after a build cycle
- Architecture is approved before any code is written
- The designer sees the implementation, not a mockup, and catches drift
- Developers review code and not logic (the logic is already in the spec) and they bring the code to a stable state themselves

---

## PM ↔ developer responsibility

- **PM** owns the spec, business logic, acceptance criteria, and product decisions. Every product question from a developer or QA goes to the PM.
- **Developers** own code quality and bringing the code to a stable state. Technical fixes are theirs. They do not bounce them back to the PM.
- Principle: each role does what it's best at. The PM doesn't iterate on code style. Developers don't ask the PM how to name a variable.

---

## What happens when something is rejected

- **Pre-Dev** rejected → back to **Research & Spec** or **Design**
- **Post-Dev: Design Review** rejected → back to **Development**
- **Post-Dev: Code Review**: reviewers fix technical issues *themselves in the MR*. Only spec / logic problems are escalated to the PM, separately
- **QA Testing** bugs → back to **Post-Dev: Code Review** (developer fixes) → back to **QA Testing**
- **Any product question** (from dev or QA) → to the PM through a separate channel (chat, ticket comment), not as a returned MR or task

---

## Self-test before handoff

Before handing the MR to review (Post-Dev stages), the PM runs a checklist against the spec themselves: every AC covered, every edge case behaves correctly, screenshots captured. This saves the reviewers' time.

---

## When the workflow doesn't apply cleanly

- **Tiny tickets** (typo, copy change, config tweak): skip Pre-Dev gates, run the change through one reviewer
- **Pure infrastructure work**: no prototype, no Design Review. Replace the prototype with a sequence diagram and an API contract
- **Spike or experiment**: scope it as research, not feature work. Don't run it through the full pipeline

The pipeline is for production features. For everything else, use judgement.
