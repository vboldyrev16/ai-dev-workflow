# Prototype Checklist

Before handing the prototype to the designer (and later to QA), run through this list. The prototype is the contract: if it's flaky or partial, every stage downstream gets harder.

## Required

- [ ] **Deployed to a URL** (Vercel, GitHub Pages, internal hosting). Not "clone and `npm install`"
- [ ] **Clickable, not static** — page transitions, filters, tabs all actually work
- [ ] **Mock data is realistic** — not Lorem Ipsum, not "User 1 / User 2". Real value ranges, real distributions, real edge cases
- [ ] **All key states present**:
  - [ ] loading state
  - [ ] empty state (no data at all)
  - [ ] error state (request failed, partial failure)
  - [ ] partial-data state (some sections empty)
  - [ ] one element (singular)
  - [ ] many elements (100+, scrolling, pagination if applicable)
  - [ ] very long string (truncation, wrapping)
- [ ] **Hover / active / focus states** for interactive elements (where the design demands it)
- [ ] **Responsive** for any UI with a mobile scenario
- [ ] **Spec link** visible in the prototype (footer, README, anywhere reachable)

## Required before Pre-Dev: QA Check

- [ ] **Approved by the designer.** Not "looked at it" — explicit sign-off, ideally in writing
- [ ] **Decision log** in the spec covers anything the prototype answered (e.g. "we chose carousel over grid because…")

## Not required

- [ ] ~~Connects to a real database~~ — mock data is fine
- [ ] ~~Working auth~~ — fake the logged-in state
- [ ] ~~Production-grade performance and animations~~ — that's Development

## Common mistakes

- **Only the happy path.** Empty / error states are skipped "because there's no data yet". This is exactly what causes Pre-Dev: QA Check to fail.
- **One screen per scenario.** A static collection of screens is a Figma export, not a prototype. The team can't test behavior on it.
- **Local-only.** "I'll show it on my laptop" doesn't scale. Deploy it.
- **Stale URL.** Prototype was updated; URL still points to the old build. Always re-deploy after a change.
- **Mock data that gives the wrong impression.** Five identical rows look fine. The same five rows in production with realistic variance look broken. Use real-shaped data.

## Tip — use AI to spin up the prototype

A functional prototype with all states, deployed to a URL, used to take days. With Claude Code (or similar) it can be done in a couple of hours. The spec drives the prototype, the prototype drives the design conversation, and the design conversation drives the final spec. Tight loop.
