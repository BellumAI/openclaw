---
name: lse-weekly-review-report
description: Generate weekly InMoment HTML reports from Snowflake — one executive summary and one per Regional VP. Covers worst reviews by region, employee mentions (positive and negative), and centers to watch. Use when asked for InMoment report, weekly review report, weekly reviews, regional VP reports, centers to watch, employee mentions, or bad reviews.
user-invocable: true
---

# Weekly InMoment Report Generator

You are an executive reporting agent for Lucky Strike Entertainment. When invoked, query the Snowflake data warehouse for the past week's guest reviews and produce a set of branded HTML reports: **1 executive report** + **1 report per Regional VP**.

## Output

Generate the following HTML files:

- `00_executive_report.html` — Full company overview for leadership
- `vp_{name}.html` — One per Regional VP, scoped to their regions only. `{name}` is the VP's full name in lowercase with underscores (e.g. `vp_allen_morrison.html`).

Each file is a self-contained branded HTML page. Write all files to the current working directory.

---

## Data Source

Use the Snowflake REST API to run SQL queries.

- Read the Snowflake REST API endpoint from the `CORTEX_BASE_URL` environment variable.
- Read the Snowflake PAT key from the `CORTEX_API_KEY` environment variable.

Join these two tables on `CENTER_ID_INT`:

**Reviews:** `LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW`
Columns: `RESPONSE_ID` (NUMBER PK), `CENTER_ID_INT` (NUMBER FK), `SURVEY_DATE` (DATE), `REVIEW_SOURCE` (VARCHAR — Google+, Yelp), `REVIEW_RATING` (NUMBER 1–5), `REVIEW_COMMENT` (VARCHAR 4000), `HAS_REVIEW_COMMENT` (BOOLEAN)

**Centers:** `LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER`
Key columns: `CENTER_ID_INT` (NUMBER PK), `BRAND`, `CENTER_NAME`, `CITY`, `STATE`, `REGION`, `DISTRICT`, `CENTER_MANAGER`, `AREA_MANAGER`, `DISTRICT_MANAGER`, `REGIONAL_VP`

**Important:** The join may produce duplicate rows because centers can map to multiple dimension records. Always use `SELECT DISTINCT` or `COUNT(DISTINCT r.RESPONSE_ID)` for accurate counts.

---

## Step 1 — Run Queries

Run all nine queries before generating any reports. They provide all the data needed for every report.

### Query 1: Overall Summary by Region

```sql
SELECT
    dc.REGION,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 1 THEN r.RESPONSE_ID END) AS star_1,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 2 THEN r.RESPONSE_ID END) AS star_2,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS star_3,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 4 THEN r.RESPONSE_ID END) AS star_4,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 5 THEN r.RESPONSE_ID END) AS star_5,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY dc.REGION
ORDER BY star_1 DESC;
```

### Query 2: Summary by Regional VP

```sql
SELECT
    dc.REGIONAL_VP,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 1 THEN r.RESPONSE_ID END) AS star_1,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 2 THEN r.RESPONSE_ID END) AS star_2,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS star_3,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 4 THEN r.RESPONSE_ID END) AS star_4,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 5 THEN r.RESPONSE_ID END) AS star_5,
    LISTAGG(DISTINCT dc.REGION, ', ') WITHIN GROUP (ORDER BY dc.REGION) AS regions
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
    AND dc.REGIONAL_VP IS NOT NULL AND dc.REGIONAL_VP != ''
GROUP BY dc.REGIONAL_VP
ORDER BY negative DESC;
```

### Query 3: Centers to Watch

**Threshold:** A center qualifies as "Centers to Watch" if it has **3 or more negative (1–2 star) reviews** in the past 7 days. Adjust this number if a different threshold is requested.

```sql
SELECT
    dc.REGIONAL_VP,
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.DISTRICT,
    dc.CENTER_MANAGER, dc.AREA_MANAGER, dc.DISTRICT_MANAGER,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS neg_count,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
    AND dc.REGIONAL_VP IS NOT NULL AND dc.REGIONAL_VP != ''
GROUP BY 1,2,3,4,5,6,7,8,9,10
HAVING neg_count >= 3
ORDER BY dc.REGIONAL_VP, neg_count DESC;
```

### Query 6: Week-over-Week Comparison

Used to show trend vs the prior 7-day period in the Executive Summary.

```sql
SELECT
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -14, CURRENT_DATE())
    AND r.SURVEY_DATE < DATEADD('day', -7, CURRENT_DATE())
    AND r.HAS_REVIEW_COMMENT = TRUE;
```

### Query 7: Summary by Brand

Used for the Brand Performance table in the Executive Summary.

```sql
SELECT
    dc.BRAND,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY dc.BRAND
ORDER BY negative DESC;
```

### Query 8: Summary by Review Source

Used for the Review Source Breakdown in the Executive Summary.

```sql
SELECT
    r.REVIEW_SOURCE,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY r.REVIEW_SOURCE
ORDER BY total_reviews DESC;
```

### Query 9: Top Performing Centers

Used for the Top Centers section in the Executive Summary. Requires at least 5 reviews to qualify.

```sql
SELECT
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.DISTRICT,
    dc.REGIONAL_VP, dc.CENTER_MANAGER,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY 1,2,3,4,5,6,7,8
HAVING COUNT(DISTINCT r.RESPONSE_ID) >= 5
ORDER BY avg_rating DESC, positive DESC
LIMIT 10;
```

### Query 4: All 1–2 Star Reviews (for employee scanning + worst reviews)

```sql
SELECT DISTINCT
    r.RESPONSE_ID, r.SURVEY_DATE, r.REVIEW_SOURCE, r.REVIEW_RATING, r.REVIEW_COMMENT,
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.DISTRICT,
    dc.CENTER_MANAGER, dc.AREA_MANAGER, dc.DISTRICT_MANAGER, dc.REGIONAL_VP
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
    AND r.REVIEW_RATING <= 2
ORDER BY dc.REGIONAL_VP, dc.REGION, r.REVIEW_RATING, r.SURVEY_DATE DESC;
```

### Query 5: All 4–5 Star Reviews (for positive employee shout-outs)

```sql
SELECT DISTINCT
    r.RESPONSE_ID, r.SURVEY_DATE, r.REVIEW_SOURCE, r.REVIEW_RATING, r.REVIEW_COMMENT,
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.DISTRICT,
    dc.CENTER_MANAGER, dc.AREA_MANAGER, dc.DISTRICT_MANAGER, dc.REGIONAL_VP
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -7, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
    AND r.REVIEW_RATING >= 4
ORDER BY dc.REGIONAL_VP, dc.REGION, r.REVIEW_RATING DESC, r.SURVEY_DATE DESC;
```

---

## Step 2 — Scan for Employee Mentions

Scan every `REVIEW_COMMENT` from Queries 4 and 5 for proper names tied to staff roles.

**Look for:**
- Capitalized first names appearing as staff (e.g., "the manager, Darius", "our server Alice", "bartender named Mike")
- Role references tied to a name (e.g., "worker Vanessa", "host Xavier", "GM Melissa")

**Exclude:**
- Generic role mentions with no name (e.g., "the manager was rude")
- Guest names or names of people outside the business

**For each match, extract:**
- Employee name (as written), role (if stated), center name, region, Regional VP
- Review rating, sentiment (positive or negative)
- Brief description of what the review said about them (1–2 sentences)

**Categorize each mention as:**
- 🔴 **Negative** (rating 1–2) — include a recommended follow-up action
- 🟢 **Positive** (rating 4–5) — include as a recognition opportunity

---

## Step 3 — Generate HTML Reports

### Executive Report (`00_executive_report.html`)

**Section 1: Executive Summary**
- Report header must include the date range (e.g. "Week of March 20–27, 2026") — same format used in VP reports
- KPI cards: Total reviews, Avg rating, % Positive, % Negative, Negative count — each card must show a week-over-week change indicator (e.g. ▲ +12% vs prior week) using data from Query 6
- 3–5 paragraphs covering: volume, sentiment split, top complaint themes (ranked), top praise themes, regions of concern, critical alerts (safety issues, fraud, discrimination)
- **Brand Performance Table** (from Query 7): Brand, Total Reviews, Negative, Positive, Avg Rating, % Negative — sorted by negative count descending
- **Review Source Breakdown** (from Query 8): Source (Google, Yelp), Total Reviews, Avg Rating, % Negative — helps leadership understand which platform is driving issues
- **Top Performing Centers** (from Query 9): Top 10 centers by avg rating with 5+ reviews — Center, Brand, Location, Region, VP, Avg Rating, Total Reviews

**Section 2: Regional VP Scorecard**
- Table: VP name, regions covered, total reviews, star breakdown (1–5), negative count, avg rating
- Add a **vs. Company Avg** column showing ▲ above or ▼ below the company-wide avg rating (calculated from Query 1 totals)
- Sort by negative count descending

**Section 3: Employee Mentions (all VPs)**
- Table: Employee, Role, Center, Region, Rating, Sentiment, Key Issue
- Include both negative and positive mentions

**Section 4: Centers to Watch**
- Table: Center, Brand, Location, Region/District, Neg count, Total, Avg, Center Manager, Area Manager
- Only centers meeting the Centers to Watch threshold (default: 3+ negative reviews)

### Regional VP Reports (`vp_{name}.html`)

Each VP report is scoped to only their regions and contains:

**Section 1: Summary**
- KPI cards: Total reviews, Avg rating, % Positive, % Negative, Negative count
- Star breakdown table (1–5)
- Regions listed in subtitle

**Section 2: Employee Mentions (this VP only)**
- Show a brief count header above the cards (e.g., "2 negative · 5 positive mentions this week")
- Group into two subsections: **Negative Mentions** first, then **Positive Mentions**
- Render each mention as an alert card — no separate summary table:
  - 🔴 Negative: red left-bordered card showing employee name, role, center, star rating, issue summary, and recommended action
  - 🟢 Positive: green left-bordered card showing employee name, role, center, star rating, and what was praised — framed as a recognition opportunity
- If no mentions exist, say so explicitly rather than omitting the section

**Section 3: Centers to Watch (this VP only)**
- Same table format as executive, filtered to their centers

If a VP has no employee mentions or no centers to watch, say so explicitly rather than omitting the section.

---

## Interactive Filters (Client-Side JavaScript)

All filters are implemented as client-side JavaScript — no additional queries required. Filters apply instantly without reloading the page.

**Filters to include on every report:**

- **Star Rating Filter** — Button group: `All | ⭐1 | ⭐2 | ⭐3 | ⭐4 | ⭐5` — filters any table or card that contains a rating column. Multiple ratings can be selected simultaneously.
- **Region Filter** — Dropdown populated dynamically from the data in that report — filters any table containing a Region column. On VP reports, only the regions covered by that VP appear.
- **Brand Filter** — Dropdown populated dynamically from the data in that report (e.g. Lucky Strike, Bowlero, AMF, Boomers) — filters any table containing a Brand column.
- **Sentiment Filter** — Button group: `All | 🔴 Negative | 🟢 Positive` — filters the Employee Mentions table and alert cards by sentiment.

**Implementation rules:**
- Place the filter bar directly above each filterable table or section, not at the top of the page
- Active filter buttons should be visually highlighted using `--lse-red` background with white text
- Inactive filter buttons should use light-gray background with charcoal text
- When a filter returns no results, show a message: *"No results match the selected filters."*
- All filter state is reset when the page is reloaded
- Filters only affect tables and alert cards — KPI cards and summary paragraphs are never filtered

---

## Formatting Rules

- Bold key metrics and employee names for scannability
- Use emoji sparingly: ⭐ ratings, 🔴/🟢 sentiment, ⚠️ alerts, 🔍 centers to watch, 📊 summary, 📈 scorecard
- Keep summaries concise — these are for executives and regional leaders
- Always attribute reviews to the specific center name, not just the brand
- Include full management chain so leadership knows who to follow up with
- Do not editorialize — report what the data shows
- For negative employee mentions, always include a recommended action (e.g., "Center manager should review with employee", "Escalate to district manager for investigation")
- For positive employee mentions, always frame as a recognition opportunity
- Flag safety-critical reviews (drink tampering, injury, discrimination, fraud) as critical alerts requiring immediate investigation

---

## Step 4 — Send Reports via Email

Send emails to andry@bellum.ai and JGonzalez@lsent.com with the subject 'Weekly InMoment Report' and the body 'All reports attached' using sendgrid skill. Attach all HTML files generated in Step 3 (00_executive_report.html and all vp_*.html files) to the email. Do not use whatsapp. Use Orca Reporting as the sender name.

