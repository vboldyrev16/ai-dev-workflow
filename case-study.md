# Case Study — Analytics Dashboard

The first feature shipped end-to-end with this workflow. An analytics dashboard for ~250 premium dealers on a classifieds marketplace. Released to production after 38 days, by 1 PM + AI for the build, and the same core team that would have built it traditionally for the review.

## Headline numbers

| Metric | Value |
| --- | --- |
| Time, customer-dev data to production | **38 days** |
| Build team | 1 PM + Claude Code |
| Review team | 1 BE lead + 1 FE lead + QA + Security + Designer + DevOps |
| Core-team story points consumed | **136 SP** (~174 person-hours) |
| Traditional team estimate (1 BE + 1 FE + 1 QA from scratch) | **3–5 months, ~780–1,300 SP** |
| Ratio | **5.7–9.6× less core-team capacity** |
| Live pair-call time during Development | **8h with backend lead + 1h with frontend lead** |
| Tooling cost | ~$600 in AI tokens |

The 5–10× number is one project, not a guarantee. It came from a feature with good fit: standalone microservice, well-understood data, a PM with deep technical background, and a core team willing to review aggressively rather than build from scratch.

## Timeline

| Date | Stage |
| --- | --- |
| Mar 24 | Customer-dev data delivered. Project effectively starts here |
| Mar 26 | Prototype + spec ready. Handed to designer. 2 days from data to working prototype |
| Mar 26 – Apr 3 | 3–4 design iterations |
| Apr 3 | Final design + UI spec from designer. Prototype v2 |
| Apr 6 | Dev kickoff with the backend lead. Architecture, repo, MR flow, CI agreed |
| Apr 6 – Apr 9 | 8 hours of joint work with the backend lead, 1 hour with the frontend lead. Full stack done |
| Apr 9 | Handed off to review. Microservice and monolith API both ready |
| Apr 9 – Apr 28 | **3 weeks stuck on code review.** Core team had no capacity. This is the bottleneck the rest of the workflow then fixed |
| Apr 28 | Code review passed. Security and functional review next |
| May 1 | Released to production. Closed beta 10 → 50 dealers |

## What worked

### Spec as SSOT

One document covered business logic, formulas (`VIP_LIFT = 4.24`, `ORGANIC_VPD = 312`, `THRESHOLD = 1.5`, five scenarios A/B+C/D/E/F), all UI states, auth/security with explicit GDPR Art. 15 references, and a decision log explaining *why* phone masking is on, why a particular spend metric was excluded from MVP, why UTC.

Result: by Pre-Dev: Tech Review, the backend lead had no "where does this data come from?" questions. By QA Testing, the test team ran exactly the trigger conditions the spec listed, no improvisation.

### Pre-Dev: QA Check caught 15 issues before any code was written

The spec was run through a structured `task-review` with QA. 15 questions closed in one pass. Each of those questions would have surfaced as a bug in QA Testing or as an iteration through the PM during Development.

### Pre-Dev: Tech Review reshaped the architecture

The PM proposed an architecture. The backend lead suggested JWT through a shared core-auth secret, which the PM would not have arrived at. Catching it before any code was written meant zero rework.

### Functional prototype carried the design conversation

A clickable, deployed prototype with realistic mock data and all states (empty, error, edge) let the designer review behavior, not just visuals. 3–4 iterations to an approved design, instead of 6–8.

### Reviewer fixes the code, no bounce-back

The big behavior change. Two MRs from the same project make the case:

**Before** (earlier MR, 7-day cycle):
- PM creates MR (23 commits)
- Reviewer writes 4 issues in comments
- PM reads, makes 5 commits with fixes themselves
- PM writes "Done on all 4 items" with mapping
- MR merged

**After** (later MR, 10-day cycle but more complex):
- PM creates MR + description
- Reviewer **pushes 5 commits directly** to the same branch: bug fix, layered auth, redirect fix, P0 tests
- 0 review comments bouncing back to PM
- MR merged straight to QA

The cycle time is similar in calendar days, but the PM spent ~0 hours on the second one versus ~3 hours of round-trips on the first. The reviewer spent comparable time either way.

## What got stuck

### 3 weeks on code review

The longest single delay in the project. Not an AI-workflow problem, an organizational one: the core team didn't have a reviewer slot. The fix (reviewer pushes commits directly, see above) came out of this incident.

### Three boards for one piece of work

Tasks tracked across PM's own board, design board, and core-team board. Status sync is manual. Known not-ideal, treated as transitional.

### Spec quality is now the bottleneck

When the PM can write a tight spec, the workflow is fast. When the PM can't, Claude Code fills in gaps with guesses and the cycle slows down. That makes spec quality a hard prerequisite, not a soft skill.

### Single PM, single data point

The workflow ran with one PM with a deep technical background. Whether it reproduces across 3–5 PMs with mixed backgrounds is an open question. A pilot is the right next step, not a rollout.

## Resource breakdown (the 136 SP, by department)

| Department | SP | ≈ hours | Role |
| --- | --- | --- | --- |
| Backend | 68 | ~87 h | 1 backend lead (review + refactor) |
| Frontend | 26 | ~33 h | 1 frontend lead (review + refactor) |
| QA + Security | 42 | ~54 h | QA + security engineers |
| DevOps / Infra | off-board | — | CI/CD, nginx, deploy, dashboard domain |
| Design | separate board | — | Designer: prototype iterations + final UI spec |
| UX research | separate board | — | UX researcher: dealer interviews and synthesis |
| **Total: MVP + follow-ups** | **136** | **~174 h** | 3 people across 38 days delivery + iteration after launch |

How the SP-to-hour conversion was derived: a recent sprint closed 437 SP with 7 active devs over 2 weeks (7 × 80 calendar hours = 560 person-hours). That's about 1 SP ≈ 1.28 person-hours on average. The team has no official SP-to-hour rate. This is sprint-capacity arithmetic, not policy.

Not counted in the 136 SP: PM time, AI tooling cost (~$600), pre-existing infrastructure.

## What this means

Not "AI replaces developers". The product still required a backend lead, frontend lead, designer, QA, security, devops, and a UX researcher. None of them were skipped.

What changed: who *built* the first version. Instead of pulling 3 people (1 BE + 1 FE + 1 QA) for 3–5 months to build a dashboard from scratch, the same team reviewed and hardened a PM + AI build over ~174 hours. The core team gets the parts only they can do. The PM ships a project that, in the older model, would have waited in a backlog for the team to free up.

The honest framing: this freed the core team to work on things the AI workflow can't reach (subscription engine, infrastructure migrations, payment flow). Not "we shipped more for less". "We shipped a thing we weren't going to ship at all this quarter, and the team kept doing their existing work."
