# Motors Analytics Dashboard — Spec (Anonymized Example)

> **Version**: 2.1 (anonymized) · **Originally written**: 2026-04-07 · **Originally shipped**: 2026-05-01

This is the actual spec that drove the case-study build, with internal identifiers replaced by placeholders. **Read this together with [spec-requirements.md](../spec-requirements.md)**, which is the quality bar this spec was written against.

---

## ⚠️ How this is anonymized

The original spec was written in Russian for an internal platform. To make it useful as a public example without leaking the source, identifiers are replaced according to these rules:

| What's replaced | Replacement format | Why |
| --- | --- | --- |
| Internal database / table names | `<analytics_warehouse>.<events_table>` and similar `<placeholder>` style | Don't disclose specific infrastructure |
| Internal API endpoints | `<core_api>/v1/...` | Don't disclose internal service shape |
| Internal field names where they reveal product specifics | `<vertical_subscription_code>` etc. | Don't disclose product internals |
| Real platform name and region | `<the platform>`, `<the market>` | Don't link this spec to a specific company |
| Specific person names | Roles (`the sales rep`, `the backend lead`) | Privacy |
| Internal ticket / page links | Removed | Internal links wouldn't resolve anyway |

**What is *not* anonymized — these are the parts worth reading:**

- Formulas, thresholds, constants (`VIP_LIFT = 4.24`, `THRESHOLD = 1.5`, etc.)
- Scenario logic (A / B+C / D / E / F)
- Trigger conditions (`No leads`, `No calls`, `Low interest`, etc.)
- Decision log entries with rationale
- Section structure (all 16 sections preserved)
- Acceptance criteria phrasing
- Backlog format

Where you see `<placeholder>`, look for an **"Originally:"** comment nearby. That comment tells you what kind of internal data lived there — so you can map this template to your own infrastructure.

This is meant as a SPEC EXAMPLE, not a SPEC TEMPLATE. For the template, see [../templates/spec-template.md](../templates/spec-template.md).

---

## 1. Target audience

### Definition

**~250 premium dealers** on `<the platform>` in the `<the market>` region, vertical: Motors.

Criterion: users who paid an annual subscription fee for the Motors vertical in the last 12 months.

```sql
-- ⚠️ Internal table names replaced. Originally: ClickHouse, project-specific.
SELECT user_id, max(complete_date) as last_fee_date, sum(sum_total_USD) as total_fees
FROM <analytics_warehouse>.<revenue_table>
WHERE project = '<platform>'
  AND account_type = '<premium_account_type>'
  AND service_name = '<vertical_subscription_code>'
  AND money_flow = 'Fees'
  AND complete_date >= '2025-04-01'
GROUP BY user_id
-- Result: 251 users
```

Average annual fee: ~$581 (~$49/month). Of these, 246 had active Motors listings; 5 paid but were inactive.

### Why prior counts differed

| Count | What it measured | Why wrong |
| --- | --- | --- |
| 314 | Premium dealers with ANY Motors transaction | Overcounted: included multi-vertical dealers who bought one VAS for Motors |
| 281 | Premium dealers with active Motors listings *now* | Included non-paying dealers |
| 255 | Sales-team count from a separate doc | Close to 251 — drift was time-slice differences |
| **251** | **Premium dealers who paid the Motors subscription fee** | **Final, verified figure** |

### Segmentation by inventory size

| Segment | Listings | Dealers | % |
| --- | --- | --- | --- |
| Large | 50+ | 22 | 9% |
| Medium | 20-49 | 79 | 31% |
| Small | 10-19 | 85 | 34% |
| Micro | 5-9 | 37 | 15% |
| Tiny | 1-4 | 23 | 9% |
| Inactive | 0 | 5 | 2% |

Median: ~15 listings. Mean: ~25.

### VAS usage (Q1 2026)

| Group | Dealers | % |
| --- | --- | --- |
| Using VIP (`premium_top`) | 69 | 27% |
| Using TOP only | 49 | 20% |
| Not using VIP or TOP | 133 | 53% |

**Key insight**: 72% of dealers (182 of 251) don't use VIP. They are the upsell target (see Scenario E in §6).

---

## 2. Prototype — UI source of truth

### Location

- **Pro-only (MVP)**: motors-analytics-demo.vercel.app — for production use and customer testing. Single tier, no locked states.
- **3-tier (full)**: separate variant with Starter/Business/Pro tiers and locked states. For internal demos only.

3 pages MVP: Overview, My Listings, Calls. Market page is phase 2, not MVP.

### MVP strategy

MVP ships with **Pro tier only** — full access to all features. The architecture supports 3 tiers (Starter / Business / Pro), but the tier toggle is removed from the UI. To add Starter and Business later, search for `TIER_GATE` markers in the code.

### What is *not* in the prototype

The following were removed (not in MVP):

- Onboarding tour (deferred)
- Notifications dropdown
- Global date range picker (replaced by inline 7d/30d toggles on charts)
- Detail panel (slide-out) — click goes directly to the listing on `<the platform>`
- Audio player (call recordings)
- Monthly Report, Health Score, VAS ROI — all dropped as low-signal

### Interactive prototype states (dev mode)

- **Empty-states toggle** in the bottom-right shows empty states for each section
- **VIP-block states** — 5 keyboard shortcuts:

| Key | State | Description |
| --- | --- | --- |
| (default) | Normal | Real data, VIP working (Scenario A) |
| `0` | VIP weak | Category averages instead of personal data (Scenario B+C) |
| `9` | No organics | Comparison against market average (Scenario D) |
| `8` | No listings | Green CTA (Scenario F) |
| Empty toggle | No VIP | Upsell CTA (Scenario E) |

### Listings table behavior (`listings.html`)

- 9 columns: #, Car, Price, Calls, Msg, WA, Views, Days, Status (Market column dropped — phase 3)
- **Status column** = Needs Attention triggers (§8). Same logic on Overview and My Listings.
- **Desktop (>1024px)**: `table-layout: fixed` with percent widths (#=3%, Car=28%, Price=10%, etc.)
- **Mobile (≤1024px)**: horizontal scroll, pixel widths
- **Calls table on mobile**: column # hidden, Result badges compact

### Table functionality

| Function | Description |
| --- | --- |
| Search | Filter by listing title, debounce 250ms |
| Sort | By Price, Calls, Messages, WhatsApp, Views, Days, Status |
| Export CSV | All listings |
| Pagination | 20 rows per page, Previous/Next |
| Row click | Navigate to the listing on `<the platform>` |

---

## 3. Data sources

> **⚠️ All tables in this section are internal database tables replaced with placeholders.** Originally: a ClickHouse cluster accessed via the team's BI tool. Each row below maps to a real internal table the team had access to.

### Tables and freshness

| Placeholder | What it contained | Originally | Freshness | Status |
| --- | --- | --- | --- | --- |
| `<analytics_warehouse>.<events>` | Phone clicks, messages, WhatsApp leads — all per-listing via `AdvertId` | A platform-wide event log table in ClickHouse | Real-time | ✅ Primary lead source |
| `<analytics_warehouse>.<ad_views>` | Per-listing impressions | Per-listing view counters | Real-time | ✅ |
| `<analytics_warehouse>.<call_tracking_raw>` | Calls per-dealer (NOT per-listing). Field `voxnumber` for mapping | Call-tracking provider's raw data table | Daily | ✅ |
| `<analytics_warehouse>.<revenue_table>` | Revenue, VAS spend, premium-dealer fees. Fields: `user_id`, `adv_id`, `service_name`, `money_flow` | Aggregated revenue table | Daily | ✅ |
| `<analytics_warehouse>.<active_ads>` | Dealer-listing relationship. Fields: `UserId`, `AdvertId`, `Date`, `Category` | Daily aggregate of active listings | Daily | ✅ |
| `<analytics_warehouse>.<personal_stats>` | Favorites per-listing | User-action stats | Real-time | ✅ |

### `service_name` values in the revenue table

| `service_name` | Meaning | Dashboard label |
| --- | --- | --- |
| `premium_top` | VIP placement | VIP |
| `top` | TOP placement | TOP |
| `update` | Listing bump / refresh | Update |
| `add` | Additional placement | — |
| `<vertical_subscription_code>` | Annual premium-dealer fee (`money_flow='Fees'`) | — (not shown) |

### What's NOT in the warehouse (needs an API)

| Data | Where it lives | Decision |
| --- | --- | --- |
| Listing metadata (title, price, photos, category) | PostgreSQL (Django) | New API endpoint (§4.1) |
| Call-tracking mapping (`user_id` → `voxnumber`) | Django `Phone` model | API or ETL (§4.2) |

---

## 4. Blockers and resolutions

### 4.1 API for listing metadata

**Status**: Required backend work (2–4 hours).

**Problem**: The listings table needs title, price, photo, category. None of this is in the warehouse — only in PostgreSQL (Django).

**Decision**: New endpoint on the internal API.

```
GET <core_api>/internal/adverts/metadata/?ids=1,2,3
```

**Response format** (per listing):

| Field | Type | Description |
| --- | --- | --- |
| `id` | int | AdvertId |
| `title` | string | Listing title |
| `price` | decimal | Price |
| `currency` | string | Currency code |
| `thumb_url` | string | Thumbnail URL |
| `in_top` | bool | Is TOP active |
| `in_premium` | bool | Is VIP active (premium_top) |
| `rubric_name` | string | Category name |

**Existing code**: An `AdvertLightSerializer` in the Django side already has all required fields. `in_top` / `in_premium` are existing booleans on the `Advert` model.

**Cache note**: Listing metadata changes rarely. Cache the API response in Redis with TTL 1–4 hours. Invalidate on update via webhook or TTL.

### 4.2 Call-tracking mapping (resolved)

**Status**: Resolved — no manual mapping needed.

**Problem**: The Calls page and the "Total Leads" KPI need to link the call-tracking provider's dealer record to the platform's `user_id`.

**Decision**: Django `Phone` model has `calltracking = voxnumber` with FK to `User`.

**Mapping chain**:
```
Django Phone.user_id → Phone.calltracking (strip "+") → <call_tracking_raw>.voxnumber
```

Existing code in the Django stats tasks.

**Implementation choices** (picked first):

| Option | Description | Recommendation |
| --- | --- | --- |
| API endpoint | `GET <core_api>/internal/calltracking/mapping/?user_ids=1,2,3` → `{user_id: voxnumber}` | Simpler, faster |
| ETL into the warehouse | Daily mapping export to a lookup table | Better at scale if call volume is high |

---

## 5. VIP vs Regular — methodology

### Approach: dealer's VIP listings vs their *own* organic listings

For each dealer, in the same period (e.g. March 2026):

```
organic_ads[] = dealer's listings WITHOUT active VIP
vip_ads[]     = dealer's listings WITH active VIP

avg_vpd_organic = sum(views of organic_ads) / count(organic_ads) / 30 days
avg_vpd_vip     = sum(views of vip_ads)     / count(vip_ads)     / 30 days

vip_lift = avg_vpd_vip / avg_vpd_organic
```

### Why NOT a "before/after" comparison

An earlier approach compared "before VIP activation / after VIP activation". Results:

- 16.5% false negatives due to serial reactivations, batch users, etc.
- 82% positive results vs **98.5% with the current approach**

The current approach eliminates these issues by comparing VIP and organic simultaneously.

### Results across all 251 premium dealers (VIP / `premium_top`)

| Metric | Value |
| --- | --- |
| Dealers with both VIP and organic (for comparison) | 67 |
| Positive lift (VIP > organic) | 66 (98.5%) |
| Negative lift | 1 (1.5%) |
| **Median lift** | **4.93×** |
| P10 | 2.27× |
| P25 | 3.10× |
| P75 | 7.16× |
| P90 | 14.51× |

**TOP excluded from MVP** — to be added later.

### Threshold: 1.5×

| Threshold | Dealers "all OK" (A) | Dealers "issue" (B+C) | % problematic |
| --- | --- | --- | --- |
| 1.0× | 62 | 1 | 1.5% |
| 1.3× | 60 | 3 | 4.6% |
| **1.5×** | **59** | **4** | **5.8%** |
| 2.0× | ~50 | ~13 | ~19% |

**Picked 1.5×** — only 4 dealers fall into the "issue" zone. Conservative enough not to scare dealers with false warnings, sensitive enough to catch real issues.

### Precomputed constants

```python
# Recomputed 2026-04-20 against March 2026 data with corrected methodology:
# organic = listing WITHOUT any active VAS (premium_top / top / update / add),
# not just without premium_top. Feb vs Mar drift < 1%.
CATEGORY_MEDIAN_ORGANIC_VPD = 274    # median views/day for organic premium-dealer Motors listings
CATEGORY_MEDIAN_VIP_LIFT    = 5.03   # median vip_lift among 45 dealers with comparable sample
VIP_LIFT_THRESHOLD          = 1.5    # threshold separating "VIP working" / "VIP weak"
CATEGORY_BOOST_PCT          = 403    # (5.03 - 1) × 100 — shown in scenarios B+C / E / F
```

These constants are derived from March 2026 data (recompute scripts live in the project repo). Recommendation: recompute monthly.

**Previous values (superseded):** `ORGANIC_VPD=312`, `VIP_LIFT=4.24`, `BOOST_PCT=324` — computed by an earlier methodology where "organic = not-VIP", which skewed medians due to listings with TOP/Update/Add.

---

## 6. Five scenarios — dashboard logic

Each of the 251 dealers falls into exactly one scenario. The scenario determines what the "VIP vs Regular" block on Overview shows.

### Distribution by scenario

Recomputed 2026-04-20 on March 2026 data with the new methodology.

| Scenario | Description | Dealers (was) | Dealers (now) | % | Prototype state |
| --- | --- | --- | --- | --- | --- |
| A | VIP working | 59 | **45** | 18% | Default (no key) |
| B+C | VIP weak / below organic | 4 | **0** | 0% | Key `0` — effectively doesn't fire |
| D | VIP exists, little/no organic | 6 | **20** | 8% | Key `9` — grew 3× (more ads have VAS → organic<3 more often) |
| E | No VIP | 182 | **176** | 70% | Empty toggle |
| F | No data at all | 0 | **5** | 2% | Key `8` — inactive dealers without Motors activity in March |

Total 246 of 251 active (5 inactive fell into F).

### Scenario A: VIP working (45 dealers, 18%)

**Condition**:
```
vip_ads > 0 AND organic_ads >= 3 AND vip_lift > 1.5
```

**Benchmark**: the dealer's own organic.

**Formula**:
```python
boost_pct = round((vip_lift - 1) * 100)
```

**Shown**:
- Green bars: VIP vs Regular
- Text: "+{boost_pct}% more views"
- Real numbers for this specific dealer (`avg_vpd_vip`, `avg_vpd_organic`)

**Example**: A dealer with `vip_lift = 5.2` sees "+420% more views" with a chart of their VIP listings vs organic.

### Scenario B+C: VIP weak or below organic (0 dealers post-recompute)

**Condition**:
```
vip_ads > 0 AND organic_ads >= 3 AND vip_lift <= 1.5
```

**Benchmark**: category average (NOT the dealer's own data — to avoid demotivating them).

**Formula**:
```python
boost_pct = round((CATEGORY_MEDIAN_VIP_LIFT - 1) * 100)  # = 403%
```

**Shown**:
- Text: "VIP is active"
- Text: "On average, Motors dealers with VIP get +403% more views"
- Personal comparison is NOT shown (to avoid negative impression)

**Logic**: VIP isn't working for this dealer, but we don't want to scare them off VIP. Show market-average instead of personal results.

### Scenario D: VIP exists, little/no organic (20 dealers, 8%)

**Condition**:
```
vip_ads > 0 AND organic_ads < 3
```

**Benchmark**: median organic views by category (274 vpd).

**Formula**:
```python
lift_vs_category = avg_vpd_vip / CATEGORY_MEDIAN_ORGANIC_VPD  # 274
boost_pct = round((lift_vs_category - 1) * 100)
```

**Shown**:
- Text: "Your VIPs: {avg_vpd_vip} views/day"
- Text: "Average organic listing in Motors: 274 views/day"
- Text: "+{boost_pct}% above average"

**Logic**: Dealer has almost all listings in VIP, nothing to compare to — use market benchmark.

### Scenario E: No VIP (176 dealers, 70%)

**Condition**:
```
vip_ads = 0 AND organic_ads > 0
```

**Benchmark**: average category lift (upsell).

**Formula**:
```python
potential_vpd = avg_vpd_organic * CATEGORY_MEDIAN_VIP_LIFT
boost_pct = 403  # (5.03 - 1) × 100, rounded
```

**Shown**:
- Text: "Your listings: {avg_vpd_organic} views/day"
- Text: "With VIP could be: ~{potential_vpd} (+403%)"
- CTA button: "Try VIP →"

**Logic**: The main upsell scenario. 70% of dealers see a forecast based on real data from other dealers.

### Scenario F: No data at all (5 dealers, 2%)

**Condition**:
```
organic_ads = 0 AND vip_ads = 0
```

**Shown**:
- Text: "Dealers with VIP get on average +403% more views"
- CTA button: "Activate VIP →"

### Summary: what to compute, where from

| Parameter | Source | Filter |
| --- | --- | --- |
| `vip_ads` | `<analytics_warehouse>.<revenue_table>` | `service_name = 'premium_top'`, `money_flow = 'VAS'`, in period |
| `organic_ads` | `<analytics_warehouse>.<active_ads>` minus VIP | `UserId = dealer_id`, `AdvertId NOT IN (vip_ads)` |
| `avg_vpd_vip` | `<analytics_warehouse>.<ad_views>` | Views for VIP listings / count(vip_ads) / days |
| `avg_vpd_organic` | `<analytics_warehouse>.<ad_views>` | Views for organic listings / count(organic_ads) / days |
| `vip_lift` | Computed | `avg_vpd_vip / avg_vpd_organic` |

---

## 7. Leads — per-listing metrics

### Per-listing (100% attribution via `AdvertId`)

| Metric | Table | Filter |
| --- | --- | --- |
| Phone clicks | `<analytics_warehouse>.<events>` | `ShowPhone = 1` |
| Messages | `<analytics_warehouse>.<events>` | `SendChatMessage = 1` |
| WhatsApp | `<analytics_warehouse>.<events>` | `EventName = 'Whatsapp_lead'` |
| Views | `<analytics_warehouse>.<ad_views>` | `AdvertId` |
| VAS Spend | `<analytics_warehouse>.<revenue_table>` | `money_flow = 'VAS'`, `adv_id = AdvertId` |
| Favorites | `<analytics_warehouse>.<personal_stats>` | `event_type = 'AddToFavorite'` |

### Per-dealer only (call tracking)

| Metric | Table | Notes |
| --- | --- | --- |
| Total calls | `<analytics_warehouse>.<call_tracking_raw>` | Via voxnumber mapping (§4.2) |
| Answered calls | `<analytics_warehouse>.<call_tracking_raw>` | `call_result = 'Answered'` |
| Missed calls | `<analytics_warehouse>.<call_tracking_raw>` | All minus answered |

**Important**: Calls are attributed at dealer level only, NOT at listing level. One dealer has one tracking number across all listings.

### Lead sparsity

Per-listing leads are very rare:
- Median premium dealer gets ~5 phone clicks/month across ALL listings
- Per-listing: ~0.3 phone clicks/month

**Consequence**: VIP ROI is computed against views, not leads.

### Deduplication

**Recommendation**: unique per `(visitor_id, advert_id, day)`. One visitor clicking "Show phone" 3× in one day = 1 lead. Next day = another lead.

**Status**: Final decision deferred to §13.

### Default period

**7 days** for KPI cards on Overview and Calls. Data refreshed daily — sliding 7-day window. Comparison: current 7 days vs previous 7 days.

Charts (Leads over time, Calls per day) support **7d / 30d** toggle.

---

## 8. Needs Attention — smart notifications

### Concept

Block on the Overview page. Not a table with columns — a list of human-language cards. Each card: listing title + one sentence explaining. Click → navigate to the listing on `<the platform>`.

Design driven by audience: 7/10 dealers don't use analytics tools; they track everything "in their head". Notification format doesn't require dashboard literacy.

### Base values (computed per dealer)

```python
dealer_listings = all active listings of the dealer (from <active_ads>)
dealer_median_views = median(views_30d across all listings)              # <ad_views>
dealer_avg_leads = sum(leads_30d across listings) / count(listings)      # <events>
dealer_avg_views_per_day = sum(views_30d) / count(listings) / 30
# leads = phone_clicks + messages + whatsapp (per-listing, from <events>)
```

### Triggers

The status names are written for the audience (dealers without analytics tooling experience). Same statuses appear in the Status column on My Listings.

#### 🔴 No leads

**What we catch**: a listing is getting views but nobody contacts — problem is in the listing itself.

**Condition**:
```python
active_days >= 14
AND views_30d >= dealer_median_views    # visibility is normal
AND leads_30d == 0                       # phone_clicks + messages + whatsapp = 0
```

**Wording (Overview)**: `"{title} — {views_30d} views but no leads in {active_days} days"`
**Status (My Listings)**: `No leads` (red badge)

**Sort within type**: by views DESC (more views = more lost potential)

**Why 14 days**: at ~2 leads/listing/month, the first 7 days without leads are normal. After 14 — signal.

#### 🟡 No calls

**What we catch**: listing is generating messages/WhatsApp but zero "Show phone" clicks. For dealers, calls are metric #1.

**Condition**:
```python
active_days >= 7
AND (messages_30d + whatsapp_30d) >= 2   # other leads exist
AND phone_clicks_30d == 0                 # no ShowPhone
```

**Wording (Overview)**: `"{title} — {messages+wa} messages but no calls yet"`
**Status (My Listings)**: `No calls` (orange badge)

#### 🟡 Low interest

**What we catch**: listing has been live for a long time with low return. From interviews: dealers change price after 5–20 days without leads, but many forget.

**Condition**:
```python
active_days >= 30
AND leads_30d < dealer_avg_leads     # below own average
AND leads_30d > 0                     # not 0 (otherwise it's "No leads")
```

**Wording (Overview)**: `"{title} — listed {active_days} days with only {leads_30d} leads"`
**Status (My Listings)**: `Low interest` (orange badge)

#### 🟡 VIP expiring (NOT in MVP — future)

**What we catch**: VIP placement is about to expire. Reminder to renew.

**Status**: Not implemented in prototype. Requires `in_premium` and `vip_expires_in` from Django — these are not in the warehouse. Will be added when backend implements.

**Condition**:
```python
listing.in_premium == true              # VIP active
AND vip_expires_in <= 3                  # days until expiry
```

#### 🟢 Top performer

**What we catch**: dealer's best listing for the week. Positive reinforcement.

**Condition**:
```python
leads_7d == max(leads_7d across dealer's listings)
AND leads_7d >= 2                          # minimum 2 leads
```

**Wording**: `"{title} — your top performer — {leads_7d} leads"`

#### 🟢 VIP working

**What we catch**: VIP is actually boosting views. Reinforce VAS value.

**Condition**:
```python
listing.in_premium == true                                       # VIP active
AND views_per_day >= 2.0 * dealer_avg_views_per_day            # views ≥2× higher
```

**Computation**:
```python
multiplier = round(views_per_day_listing / dealer_avg_views_per_day, 1)
```

**Wording**: `"{title} — VIP driving {multiplier}× more views"`

Shown only when `multiplier >= 2.0`.

#### Listings without problems

**Status (My Listings)**: `Active` (green badge) — listing is performing normally, no issues detected.

### Display rules (Overview)

| Rule | Value |
| --- | --- |
| Max items | **3** |
| Priority | Problems first: 🔴 → 🟡 → 🟢 (only if room left) |
| If 0 problems | 1 green + `"All listings performing well ✓"` |
| If 0 triggers | `"All listings performing well ✓"` |
| Click on item | Navigate to the listing on `<the platform>` |
| "View all →" | Navigate to My Listings (problems on top) |

### Display rules (My Listings)

| Rule | Value |
| --- | --- |
| Status column | Status badge for each listing |
| Default sort | By Status ASC — problems on top (No leads → No calls → Low interest → Active) |
| User can | Re-sort by any other column |

### Data sources for triggers

All data already in the warehouse — no extra backend work needed.

| Data | Table | Filter |
| --- | --- | --- |
| views_30d per listing | `<analytics_warehouse>.<ad_views>` | `AdvertId`, last 30 days |
| phone_clicks per listing | `<analytics_warehouse>.<events>` | `ShowPhone = 1`, `AdvertId` |
| messages per listing | `<analytics_warehouse>.<events>` | `SendChatMessage = 1`, `AdvertId` |
| whatsapp per listing | `<analytics_warehouse>.<events>` | `EventName = 'Whatsapp_lead'`, `AdvertId` |
| active_days | `<analytics_warehouse>.<active_ads>` | `Date` of first appearance of `AdvertId` |
| VIP status | `<analytics_warehouse>.<revenue_table>` | `service_name = 'premium_top'`, `adv_id` |

---

## 9. KPI cards — Overview and Calls

### Overview: 4 KPI cards

| Card | Metric | Period | Trend |
| --- | --- | --- | --- |
| Total Leads | `phone_clicks + messages + whatsapp` | 7 days | % vs previous 7d |
| Phone Clicks | `phone_clicks` (ShowPhone) | 7 days | % vs previous 7d |
| Messages | `messages` (SendChatMessage) | 7 days | % vs previous 7d |
| WhatsApp | `whatsapp` (Whatsapp_lead) | 7 days | % vs previous 7d |

**Update**: data recomputed daily. Sliding 7-day window.

**Trend formula**:
```python
trend_pct = round((current_7d - previous_7d) / previous_7d * 100)
# previous_7d = days 8-14 ago
# Sentinel: when previous_7d == 0, return "New" instead of inf%
```

### Calls: 3 KPI cards

| Card | Metric | Period | Trend |
| --- | --- | --- | --- |
| Total Calls | all calls | 7 days | % vs previous 7d |
| Answered | answered calls | 7 days | answer rate % |
| Missed | missed calls | 7 days | miss rate % |

### My Listings: 3 KPI cards

| Card | Metric | Formula |
| --- | --- | --- |
| Active Listings | Count of active listings | `count(<active_ads> WHERE UserId = dealer_id)` |
| Avg Leads / Listing · 7d | Avg leads per listing in 7 days | `sum(leads_7d across listings) / count(listings)` |
| Avg Days Active | Avg time active | `sum(daysActive) / count(listings)` |

---

## 10. Leads over time — chart

### Type: stacked area chart

Two datasets:
- **Calls** (blue, `#2563eb`) — `dailyLeads.calls` (phone clicks per day, NOT call-tracking calls)
- **Messages & WhatsApp** (light blue, `#38bdf8`) — `dailyLeads.messages + dailyLeads.whatsapp`

**Decision to merge messages + whatsapp**: 3 lines on the chart was too cramped. Both are text channels — logical grouping.

### Period toggle

- **7 days** — last 7 points
- **30 days** (default) — all 30 points

---

## 11. Promotion Spend — VAS spend

> **Note**: Dropped from MVP late in the cycle (decision in §13). Returns in v1.x.

### Visualization: horizontal bars (totals for period)

4 VAS categories:

| Category | `service_name` | Color |
| --- | --- | --- |
| TOP | `top` | `#f97316` |
| VIP | `premium_top` | `#fbbf24` |
| Add | `add` | `#164e63` |
| Update | `update` | `#2563eb` |

### Numbers shown

- **Total** — sum of VAS over 30 days
- **Daily avg** — total / 30
- Each bar shows the absolute amount in currency

### Source

`<analytics_warehouse>.<revenue_table>` → `money_flow = 'VAS'`, grouped by `service_name`, last 30 days.

---

## 12. Authentication and access

### Principle

Dashboard is accessible only to authenticated users of `<the platform>` with the right account type. No separate registration or login — reuse existing site session.

### Who has access

| Condition | Description |
| --- | --- |
| Authenticated on `<the platform>` | Valid JWT/session from shared auth service |
| `account_type = <premium_account_type>` | Premium dealer (current MVP). Future: subscription tiers |
| Vertical = Motors | Dealer with active Motors listings OR paid Motors subscription |

### Entry point (MVP)

**Direct link** — no button in the main app yet.

- URL: `<platform_domain>/analytics/motors/`
- Distributed by the sales rep to dealers manually or via email
- Later: button in the dealer profile (separate task on the main app)

### Auth flow

```
User opens dashboard URL
        ↓
Microservice validates JWT/session via shared auth
        ↓                          ↓
   Valid session             No session / invalid
        ↓                          ↓
   Check:                    Redirect to platform login
   - account_type in allowed set
   - Motors vertical (active listings OR Motors subscription)
        ↓                          ↓
   OK → dashboard            Not allowed → "Access restricted" page
```

### Implementation

**Decision**: Standalone JWT validation (same algorithm as the shared auth service, no dependency on the private package).

| Component | File | Status |
| --- | --- | --- |
| JWT validation | `backend/src/api/deps.py` | ✅ Implemented (PyJWT, HS256) |
| Auth gate | All endpoints except `/health` | ✅ 401 without token |
| Mock mode | `USE_MOCK_DATA=true` | ✅ Auth optional |
| Shared auth integration | — | Deferred (private package not installable via pip) |

**Note**: For shared-deployment, `AUTH_SECRET` must match the shared auth signing key. `DATABASE_URL` is the connection string for the future session-token lookup.

### "Access restricted" page

For users who failed the check (not premium, not Motors):
- Message: "Analytics is available for Motors dealers with the Pro subscription"
- CTA: "Learn about subscription →" (link to subscription page when it exists)

### Phone masking (GDPR Art. 15)

Call log shows phone numbers in masked form: `+1-555 9****234`. Full numbers are accessible via API only for the dealer's own calls. Audit log captures all data accesses.

---

## 13. Decisions made

| # | Question | Decision |
| --- | --- | --- |
| 1 | **Lead deduplication** | MVP: no dedup, count all clicks. Improvement (unique per visitor/advert/day) → backlog B3 |
| 2 | **VIP benchmarks** | MVP: hardcoded (`CATEGORY_MEDIAN_VIP_LIFT = 5.03`, `CATEGORY_MEDIAN_ORGANIC_VPD = 274`). Auto-recompute → backlog B4 |
| 3 | **Architecture** | FastAPI + React microservice ✅ |
| 4 | **Auth** | HttpOnly cookie + JWT via shared auth ✅ |
| 5 | **Phone masking** | ON (`+1-555 9****234`) — for GDPR Art. 15 compliance |
| 6 | **Error states** | Reuse empty state + "Unable to load data" + Retry |
| 7 | **Timezone** | UTC for MVP. Local-time conversion → backlog B15 |
| 8 | **Currency format** | Matches the platform's display format |
| 9 | **Call results** | Removed "blocked", `call_result = answered/missed/voicemail` |
| 10 | **Promotion Spend** | **Removed from MVP** (cutover decision). Returns in v1.x |
| 11 | **Sentry / Logstash** | Deferred, not in MVP |
| 12 | **Overview KPI** (2026-05) | Focus on leads. Views removed from Overview after custdev. 4 lead cards instead. Views remain on My Listings |
| 13 | **Messages & WhatsApp merged** in Leads chart (2026-05) | One line instead of two for chart readability. Revisit if beta feedback says split |

---

## 14. Open questions

| # | Question | Decision | Status |
| --- | --- | --- | --- |
| 1 | **Architecture**: microservice (Python/FastAPI) or Django module? | Microservice FastAPI + React | ✅ Implemented |
| 2 | **Shared-deployment**: URL, reverse proxy, CI/CD | Discuss with devops/backend | Open |
| 3 | **Call History API in Django** | Spec handed to Django team | Awaiting implementation |

---

## 15. Versions and backlog

### Versions

| Version | What's in | Status |
| --- | --- | --- |
| **1.0 — MVP** | 3 pages (Overview, My Listings, Calls). Pro-only. JWT auth. Direct link (no button in main app). Self-benchmark Needs Attention. VIP scenarios with hardcode. | ✅ Released 2026-05-01 |
| **1.x — beta feedback fixes** | Bug fixes, P0 tests | 🟡 In progress |
| **1.1 Starter & Business tiers** | Planned (after subscription engine) | Planned |
| **1.2 Button in main app** | Planned | Planned |
| **1.3 VIP expiring trigger** | Planned | Planned |
| **1.4 Lead deduplication** | Planned | Planned |
| **1.5 Promotion Spend** (return) | Planned | Planned |
| **2.0 Market** (market price benchmarks) | Phase 3 | Planned |

### Backlog

| # | Feature | Priority | Notes |
| --- | --- | --- | --- |
| B1 | Button in main app → navigate to analytics from dealer profile | High | Frontend task on main app |
| B2 | VIP expiring trigger | Medium | Needs VIP duration from `core/pay/models/` |
| B3 | Lead deduplication | Medium | unique per (visitor_id, advert_id, day) |
| B4 | Auto-recompute VIP benchmarks | Low | Monthly cron → warehouse lookup table |
| B5 | UX for small dealers | Medium | 90+ dealers with 0–3 leads/month. Maybe extend period to 90d |
| B6 | TOP comparison | Medium | TOP showed -15% lift — needs further analysis |
| B7 | Market price comparison | High (Phase 2) | Data pipeline: median prices by make/model/year |
| B8 | Market page | High (Phase 2) | Market data, demand, competition |
| B9 | Notifications | Medium | Push/in-app. Needs backend |
| B10 | Audio player | Low | Listen to call recordings |
| B11 | Onboarding tour | Low | Guide for new subscribers |
| B12 | Per-listing call attribution | Low | Only if call-tracking provider changes architecture |
| B13 | Competitor monitoring | Low | 0/10 dealers requested |

### Consciously rejected

| What | Why |
| --- | --- |
| **VIP ROI by leads** | Leads too rare (0–3 per listing) — irrelevant metric |
| **"Before/after" for VIP** | Replaced with organic benchmark (98.5% vs 82% positive) |
| **Call duration** | 10/10 dealers: "useless" |
| **Health Score (0–100)** | Not informative for target audience |
| **VAS ROI** | Requires hardcoded AVG_LEAD_VALUE — unreliable metric |
| **Detail Panel (slide-out)** | Click → navigate to platform is simpler and more useful |
| **Global Date Range Picker** | Replaced with inline 7d/30d toggles |

---

## 16. Key project files

### Implementation (analytics microservice repo)

| File / directory | Contains |
| --- | --- |
| `backend/src/api/deps.py` | DI: JWT auth + DataService toggle (mock/real) |
| `backend/src/services/real_data.py` | RealDataService — assembles data from warehouse + Django Core + Redis |
| `backend/src/services/mock_data.py` | MockDataService — synthetic data for development |
| `backend/src/clients/` | Clients: warehouse (7 queries), Django Core, Redis, Call History |
| `frontend/src/pages/` | 3 pages: Overview, Listings, Calls |
| `e2e/tests/dashboard.spec.ts` | 12 Playwright e2e tests |

### Prototype and UI

| File / directory | Contains |
| --- | --- |
| `prototype/` | Pro-only prototype (MVP) — UI source of truth |

---

## 17. Superseded documents

Prior versions of this analysis are superseded by this spec.

| Document | What's outdated | Use instead |
| --- | --- | --- |
| `VIP_PERFORMANCE_REPORT.md` | "Before/after" methodology (82% positive) | §5 of this spec: organic benchmark (98.5% positive) |
| `VIP_SCENARIOS_DEEP_DIVE.md` | 7 scenarios. Useful for understanding the cause of negatives, but the scenarios themselves were mostly eliminated by the new methodology | §6 of this spec: 5 scenarios |
| Early data about "314 dealers" or "281 dealers" | Inflated / imprecise counts | §1: **251 dealers** — final figure |

---

*End of spec. Version 2.1 (anonymized), originally written 2026-04-07.*
