# Pre-Dev: QA Check — Gate Checklist

Run this at stage 4 of the workflow, *before* Development. The goal is to validate that the spec and the prototype are complete enough for the team to build without round-trips through the PM.

Soft gate: if **required** items aren't checked, don't send the task to QA — it will bounce. **Recommended** items raise the quality of later stages but aren't blockers.

## Required

### Spec coverage

- [ ] Spec covers every use case described in the original backlog item
- [ ] Each use case has an explicit acceptance criterion
- [ ] Edge cases are described: empty, error, zero values, long strings, no data, partial data
- [ ] What's *out of scope* is named explicitly (scope-out section or per-section "what we don't show and why")

### Data and logic

- [ ] Data sources are concrete: table names, endpoint names, field names
- [ ] Access has been verified (someone ran a test query / API call)
- [ ] Every formula and calculation is written out explicitly with constants
- [ ] Constants are pulled out into a list, not buried inside text

### UI

- [ ] UI states are described per page: loading, empty, error, partial
- [ ] Prototype is clickable and demonstrates the key states (for UI features)
- [ ] Prototype is deployed to a URL the team can open
- [ ] Designer has approved the prototype (for UI features)

### Security and compliance

- [ ] Auth model is described: who sees what, who can't see what
- [ ] GDPR / compliance aspects are called out with explicit regulation references where applicable

### Decision trail

- [ ] Decision log is filled in: what's in, what's deferred, what's in the backlog with rationale

## Recommended

- [ ] Technical proposals (stack, DB, auth mechanism) drafted — final call is at Tech Review
- [ ] Sequence diagrams for non-trivial interaction flows
- [ ] Example request / response payloads for each new API endpoint
- [ ] Backlog items numbered (B1, B2, …) for easy reference
- [ ] "What we don't show and why" section — preempts QA questions

## How to run the check

The PM walks through this list and resolves gaps. Then it goes to QA. QA does its own pass — typically running a structured `task-review` against the spec — and either signs off or returns a list of remaining issues.

Issues found at this stage become spec edits. They don't become bug tickets and they don't become code review comments later. That's the point of running the check upfront.

A good signal: QA closes 10–15 issues in a single pass. A bad signal: zero issues found (usually means the reviewer didn't go deep) or 50+ (usually means the spec wasn't ready for review).
