# Spec & Prototype Requirements

The quality bar for the spec and the prototype. Used at Pre-Dev: QA Check (stage 4) to decide whether a feature is ready for development.

## Why this matters more under AI development

Claude Code treats the spec as ground truth. Anything not in the spec gets filled in with guesses or surfaces as PM iteration at the end of the cycle. A tight spec and a functional prototype mean fewer guesses, fewer surprises at Code Review or QA Testing, and a shorter path to production.

This is a guideline, not a hard gate. For large features it is mandatory in full. For small ones, apply the parts that fit.

---

## Three principles

### 1. Spec = SSOT (single source of truth)

No separate PRDs, AC documents, UX docs, or design tickets carrying logic. If a business rule lives in a Slack thread, it doesn't exist. When the spec changes, code, design, and tests follow. Never the other way around.

### 2. Prototype = functional, not static

A prototype is not a stack of Figma screens. It is a working, clickable artifact (HTML, Vercel, or similar) that demonstrates *logic and states*, not just visual. The team should be able to open a URL and try it.

### 3. All edge cases described before development

Empty, error, zero values, long strings, no data, partial data — all described at Research & Spec, validated at Pre-Dev: QA Check, *then* development. "Edge cases we'll figure out as we go" is almost guaranteed to cause bounces from QA Testing.

---

## Spec structure

Universal template. **Required** sections appear in every spec. Optional sections are added when applicable.

| Section | Status | What it contains |
| --- | --- | --- |
| 1. Header | required | Last update date, status (draft / approved / released), owner, link to prototype, link to security review (if applicable) |
| 2. Target audience and problem | required | Who needs this, what pain it solves, segments with numbers if available. Not "we improve UX" — "X% of dealers don't use VIP, this is an upsell scenario" |
| 3. Prototype | required (for UI features) | URL of the deployed prototype, who approved it, in what state |
| 4. Data sources and dependencies | required | Concrete tables / APIs / services: database names, table names, endpoints, field names. What is needed from other teams (separate "Task / Status" table) |
| 5. Metrics and formulas | required (if calculations exist) | Every metric with an explicit formula and definition. Constants pulled out separately. Never "we count something across listings" |
| 6. UI by page or section | required (for UI features) | Each page: what we show, what KPIs, what charts, filters, states (loading / empty / error / partial), what we do *not* show and why |
| 7. Triggers / smart logic | if applicable | Trigger conditions explicit. "No leads" is `views > 0 AND leads = 0 AND days_active >= 14`, not "no leads" |
| 8. Auth / security | required | How the user authenticates, what they see, what they don't. GDPR / compliance requirements (masking, audit log) with explicit references to the regulation (e.g. Art. 15) |
| 9. Decision log | required | What was decided and why. What was postponed (with rationale). What was moved to the backlog. This saves the review when "why did we drop that?" comes up |
| 10. Versions and backlog | required | What's in the current version (MVP / v1.x / v2.0). Backlog with numbered items (B1, B2, …) so they can be referenced from other places |
| 11. Key project files | after development | Map of "where things live" in the code. Helps reviewer onboarding and the next developer |

---

## Prototype requirements

For UI features the prototype is mandatory. For backend-only work (API endpoints, migrations, background workers, infrastructure), replace the prototype with a sequence diagram and a contract document with example requests / responses.

### What the prototype must do

- **Clickable, not static.** Page transitions, filters, tabs all work
- **Realistic mock data.** Not Lorem Ipsum, not "User 1 / User 2". Mimic real ranges and distributions
- **All key states**: empty, error, loading, edge cases (1 element, 100 elements, very long string), hover / active for interactions
- **Responsive** if a mobile scenario applies
- **Deployed to a URL** accessible to the team. Not "pls clone and npm install"
- **Approved by the designer** before Pre-Dev: QA Check

### What the prototype does *not* have to do

- Connect to real databases
- Implement working auth
- Match production performance and animations

---

## Anti-patterns

| Anti-pattern | Why bad | What to do instead |
| --- | --- | --- |
| "TODO: clarify" in input conditions | Will end up as a magic default in code, or surface as a bug in QA | Clarify *before* Pre-Dev. If there's no answer, open a separate ticket and block on it |
| "We count something across listings" | Everyone interprets differently. The result will not be what was expected | Explicit formula: `SUM(price) WHERE status='active' AND created_at > NOW() - 30 days` |
| Prototype = only Figma screens | QA can't validate behavior. Developer doesn't see the transitions | Deployed clickable HTML with mock data and states |
| Separate PRD / AC doc / UX doc | Documents drift apart. Nobody knows which is canonical | All in one spec. PRD = the first sections, AC = inside use cases, UX = a link to the prototype |
| Edge cases "we'll figure out as we go" | Guaranteed bounces from QA Testing | Describe before Development. Pre-Dev: QA Check will surface what's missing |
| Data sources not verified | Discovery mid-development: "that field doesn't exist in the table" | Run a test SQL / API call during Research |
| Decision log missing | A month later you can't answer "why did we drop that?" | Each non-trivial decision = one row in the decision log with rationale |
| "Spec ready" but prototype not approved | Designer finds gaps, the spec changes, ACs change | Prototype approved *before* Pre-Dev: QA Check |
