# Spec — [feature name]

> Drop-in template. Required sections are marked **required**. Delete this blockquote, the section descriptions, and unused optional sections before sharing.

## 1. Header [required]

| Field | Value |
| --- | --- |
| Status | draft / approved / released |
| Last updated | YYYY-MM-DD |
| Owner | [name + role] |
| Prototype URL | [url] |
| Security review | [link or "not applicable"] |

## 2. Target audience and problem [required]

Who needs this. What pain it solves. Segments with numbers where available.

Not "we improve UX" — "X% of segment Y don't use feature Z, this is the upsell path".

## 3. Prototype [required for UI features]

- URL: [deployed prototype]
- Approved by: [designer name + date]
- State: [draft / approved / final]

For backend-only work, replace this section with a sequence diagram and an API contract.

## 4. Data sources and dependencies [required]

Concrete table / API / service names. Field-level detail.

| Source | Owner | Status | Notes |
| --- | --- | --- | --- |
| `analytics.dealer_listings` | data team | exists | columns: ... |
| `core-api/v1/me` | platform team | needs new field | requested via ... |

External dependencies (other teams):

| Task | Status |
| --- | --- |
| Add field X to /me endpoint | requested 2026-MM-DD |

## 5. Metrics and formulas [required if calculations exist]

Each metric with an explicit formula and definition. Constants pulled out separately.

**Constants:**

- `VIP_LIFT = 4.24` — average views uplift from VIP listing, derived from [source]
- `ORGANIC_VPD = 312` — baseline organic views per day per listing, derived from [source]

**Metrics:**

- `effective_views = organic_views * (1 + has_vip * VIP_LIFT)`
- `roi = (vip_revenue_uplift - vip_cost) / vip_cost`

## 6. UI by page or section [required for UI features]

For each page:

### Page: [name]

- **What we show:** [KPIs, charts, filters]
- **What we don't show, and why:** [explicit list]
- **States:**
  - loading: [behavior]
  - empty: [behavior, copy]
  - error: [behavior, copy]
  - partial data: [behavior]

## 7. Triggers / smart logic [if applicable]

Each trigger condition explicit, expressible as code.

- **No leads**: `views > 0 AND leads = 0 AND days_active >= 14`
- **Underperforming VIP**: `has_vip AND vip_lift_actual < VIP_LIFT * 0.5`

## 8. Auth / security [required]

- **Who can access:** [user types, role checks]
- **What they see vs don't see:** [data scoping rules]
- **PII handling:** [masking, audit log]
- **Compliance references:** [GDPR Art. 15, regional regs]

## 9. Decision log [required]

| Decision | Why | Date |
| --- | --- | --- |
| Include phone masking by default | GDPR + dealer concern about scraping | 2026-MM-DD |
| Exclude promotion spend from MVP | Data source not stable yet, deferred to v2 | 2026-MM-DD |
| All timestamps in UTC | Avoid timezone bugs at the API boundary | 2026-MM-DD |

## 10. Versions and backlog [required]

### Current version

[MVP / v1.x / v2.0]

### Backlog

- **B1.** [item] — [why deferred]
- **B2.** [item] — [why deferred]
- **B3.** [item] — [why deferred]

## 11. Key project files [after development]

| Area | Path | Notes |
| --- | --- | --- |
| API entry | `src/api/main.py` | FastAPI app, routes |
| Data layer | `src/data/` | ClickHouse, Django Core, Redis adapters |
| Auth | `src/auth/jwt.py` | JWT through shared core-auth secret |
| Frontend pages | `frontend/src/pages/` | Overview, Listings, Calls |
| Tests | `tests/` | pytest, ~80% coverage |
