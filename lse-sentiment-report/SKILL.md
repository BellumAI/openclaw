---
name: lse-sentiment-report
description: Generate monthly LSE Sentiment Report from Snowflake — a single executive HTML report covering 30-day customer sentiment, theme analysis, experience gaps, location intelligence, voice of customer, and marketing action recommendations. Use when asked for sentiment report, marketing sentiment, monthly sentiment, brand perception, customer feedback analysis, sentiment trends, experience gaps, or voice of customer.
user-invocable: true
---

# LSE Sentiment Report Generator

You are a marketing intelligence agent for Lucky Strike Entertainment. When invoked, query the Snowflake data warehouse for the **past 30 days** of guest reviews and produce a single branded HTML executive sentiment report focused on marketing insights: acquisition, retention, and brand perception.

## Output

Generate one HTML file:

- `lse_sentiment_report_{YYYY-MM-DD}.html` — Executive sentiment report where `{YYYY-MM-DD}` is today's date (e.g. `lse_sentiment_report_2026-03-27.html`)

The file is a self-contained branded HTML page. Write it to the current working directory.

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

Run all ten queries before generating the report. They provide all data needed.

### Query 1: Overall Sentiment Summary (30-day)

Provides the top-level KPI numbers for the report header.

```sql
SELECT
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS neutral,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_positive,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 1 THEN r.RESPONSE_ID END) AS star_1,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 2 THEN r.RESPONSE_ID END) AS star_2,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS star_3,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 4 THEN r.RESPONSE_ID END) AS star_4,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 5 THEN r.RESPONSE_ID END) AS star_5
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE;
```

### Query 2: Prior 30-Day Comparison (Period-over-Period)

Used to compute sentiment trend deltas (▲/▼) vs the prior 30-day period.

```sql
SELECT
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS neutral,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_positive
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -60, CURRENT_DATE())
    AND r.SURVEY_DATE < DATEADD('day', -30, CURRENT_DATE())
    AND r.HAS_REVIEW_COMMENT = TRUE;
```

### Query 3: Sentiment by Brand

Breakdown of sentiment performance per brand.

```sql
SELECT
    dc.BRAND,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS neutral,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_positive
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY dc.BRAND
ORDER BY total_reviews DESC;
```

### Query 4: Sentiment by Review Source

Platform breakdown (Google, Yelp) to understand perception by channel.

```sql
SELECT
    r.REVIEW_SOURCE,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS neutral,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_positive
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY r.REVIEW_SOURCE
ORDER BY total_reviews DESC;
```

### Query 5: Sentiment by Region

Regional performance for location intelligence.

```sql
SELECT
    dc.REGION,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS neutral,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_positive
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY dc.REGION
ORDER BY avg_rating ASC;
```

### Query 6: Top-Performing Locations

Centers with the best sentiment — acquisition and marketing proof points.

```sql
SELECT
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.DISTRICT, dc.REGIONAL_VP,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_positive
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY 1,2,3,4,5,6,7
HAVING COUNT(DISTINCT r.RESPONSE_ID) >= 10
ORDER BY avg_rating DESC, positive DESC
LIMIT 15;
```

### Query 7: Underperforming Locations (Reputation Risk)

Centers with the worst sentiment — reputation management and operational escalation priorities.

```sql
SELECT
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.DISTRICT, dc.REGIONAL_VP,
    dc.CENTER_MANAGER, dc.AREA_MANAGER, dc.DISTRICT_MANAGER,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY 1,2,3,4,5,6,7,8,9,10
HAVING COUNT(DISTINCT r.RESPONSE_ID) >= 5
ORDER BY avg_rating ASC, negative DESC
LIMIT 15;
```

### Query 8: Weekly Sentiment Trend (Within 30-Day Window)

Week-by-week breakdown to detect emerging shifts and campaign impact.

```sql
SELECT
    DATE_TRUNC('week', r.SURVEY_DATE) AS week_start,
    COUNT(DISTINCT r.RESPONSE_ID) AS total_reviews,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) AS negative,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING = 3 THEN r.RESPONSE_ID END) AS neutral,
    COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (4,5) THEN r.RESPONSE_ID END) AS positive,
    ROUND(AVG(r.REVIEW_RATING), 2) AS avg_rating,
    ROUND(COUNT(DISTINCT CASE WHEN r.REVIEW_RATING IN (1,2) THEN r.RESPONSE_ID END) * 100.0 /
        NULLIF(COUNT(DISTINCT r.RESPONSE_ID), 0), 1) AS pct_negative
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
GROUP BY week_start
ORDER BY week_start ASC;
```

### Query 9: All Negative Reviews (1–2 Stars) for Theme Extraction

Full review text for AI-driven theme analysis and voice-of-customer pain points.

```sql
SELECT DISTINCT
    r.RESPONSE_ID, r.SURVEY_DATE, r.REVIEW_SOURCE, r.REVIEW_RATING, r.REVIEW_COMMENT,
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.REGIONAL_VP
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
    AND r.REVIEW_RATING <= 2
ORDER BY r.SURVEY_DATE DESC;
```

### Query 10: All Positive Reviews (4–5 Stars) for Theme Extraction

Full review text for AI-driven theme analysis and voice-of-customer strengths.

```sql
SELECT DISTINCT
    r.RESPONSE_ID, r.SURVEY_DATE, r.REVIEW_SOURCE, r.REVIEW_RATING, r.REVIEW_COMMENT,
    dc.CENTER_NAME, dc.BRAND, dc.CITY, dc.STATE, dc.REGION, dc.REGIONAL_VP
FROM LUCKYSTRIKE_ANALYSIS.MARKETING.FACT_SOCIAL_REVIEW r
JOIN LUCKYSTRIKE_ANALYSIS.SHARED.DIM_CENTER dc ON r.CENTER_ID_INT = dc.CENTER_ID_INT
WHERE r.SURVEY_DATE >= DATEADD('day', -30, CURRENT_DATE()) AND r.HAS_REVIEW_COMMENT = TRUE
    AND r.REVIEW_RATING >= 4
ORDER BY r.SURVEY_DATE DESC;
```

---

## Step 2 — Analyze Review Text

Using all review comments from Queries 9 and 10, perform the following analysis before generating the report:

### A. Theme & Driver Extraction

Read through all review comments and identify the most frequently appearing themes. Classify each theme as a positive driver or negative driver.

**Common theme categories to look for:**
- **Service** — staff friendliness, attentiveness, speed, professionalism
- **Cleanliness** — venue condition, restrooms, equipment upkeep
- **Food & Beverage** — quality, variety, value, wait times
- **Booking & Check-In** — reservation process, wait times, lane availability
- **Ambiance** — noise level, lighting, atmosphere, music
- **Pricing & Value** — cost perception, deals, upsells
- **Entertainment & Activities** — bowling, arcade, events, special occasions
- **Safety** — incidents, equipment failures, security concerns

For each theme, record:
- Theme name
- Count of reviews mentioning it (approximate)
- Sentiment: Positive / Negative / Mixed
- Example quotes (1–2 per theme)
- Sentiment impact (High / Medium / Low — based on how often it drives 1–2 star or 4–5 star ratings)

Rank themes by: (1) frequency, (2) sentiment impact.

### B. Experience Gap Detection

Compare themes appearing in **negative** reviews (Query 9) vs **positive** reviews (Query 10). Identify:

- **Operational Gaps** — themes that appear frequently in negative reviews with high sentiment impact (e.g., "slow service", "dirty lanes") — these are real experience failures
- **Perception Gaps** — themes where public reviews (Google) differ significantly from overall patterns (e.g., good food scores internally but low food ratings on Google) — these are messaging or expectation mismatches

### C. Voice-of-Customer Quote Selection

Select 6–10 high-signal quotes total:
- 2–4 **positive reinforcement** quotes: exceptional experiences worth amplifying in marketing
- 2–4 **pain point** quotes: recurring frustrations that must be addressed
- 1–2 **unexpected insight** quotes: surprising feedback that reveals unmet needs or emerging trends

For each quote, include: the quote text, center name, brand, region, star rating, and source.

### D. Marketing Action Recommendations

Translate the analysis into concrete recommendations across three categories:

**Marketing Actions** (messaging, campaigns, acquisition)
- Which themes to amplify in ads and social content
- Which platforms need reputation management attention
- Audience targeting signals from review patterns

**Experience Improvements** (operational signals for Ops/Product)
- Top recurring pain points requiring operational fixes
- Location-specific escalations (tied to underperforming centers)

**Reputation Management Priorities**
- Platforms with the worst public sentiment (by source)
- Regions or brands needing proactive review response strategy
- Safety-critical issues requiring immediate escalation (drink tampering, injury, discrimination, fraud — flag these as critical alerts)

---

## Step 3 — Generate HTML Report

### Report: `lse_sentiment_report_{YYYY-MM-DD}.html`

The report uses the LSE brand guidelines (Lucky Strike Red `#DB3434`, Charcoal `#171717`, Wix Madefor Text font). Apply the same card styling (4px solid red left border, border-radius 12px) and table styling (charcoal header, alternating rows) used across all LSE reports.

The report header must display:
- Title: **LSE Sentiment Report**
- Subtitle: **30-Day Marketing Intelligence** | `{start_date} – {end_date}`
- LSE logo

---

### Section 1: Sentiment Overview

**KPI Cards (row of 5):**
- Total Reviews
- Average Rating (⭐)
- % Positive
- % Neutral
- % Negative

Each KPI card must show a **period-over-period delta** vs the prior 30 days using data from Query 2. Format: `▲ +5.2%` (green) or `▼ -3.1%` (red). Use charcoal for neutral (no change).

**Star Distribution Bar:**
A horizontal stacked bar chart showing the share of 1★ through 5★ reviews as color-coded segments. Label each segment with its percentage. (Render using inline CSS/HTML — no external chart libraries.)

**Sentiment Summary Paragraph:**
Write 2–3 sentences summarizing the overall sentiment picture: total volume, sentiment split, trend direction vs prior period, and the single most important signal from the data.

---

### Section 2: Sentiment by Source (Platform Intelligence)

Table comparing Google vs Yelp (and any other sources):

| Platform | Total Reviews | Avg Rating | % Positive | % Negative | Trend vs Prior Period |
|----------|--------------|------------|------------|------------|-----------------------|

**Insight paragraph (2–3 sentences):** Which platform shows the biggest perception gap? Where is the brand most at risk publicly?

---

### Section 3: Brand Performance

Table: Brand | Total Reviews | Avg Rating | % Positive | % Negative | vs Company Avg

Add a **vs Company Avg** column: ▲ above or ▼ below the overall 30-day average rating.

**Insight paragraph:** Which brands are outperforming or underperforming? What does this mean for brand-specific marketing strategy?

---

### Section 4: Regional Sentiment Breakdown

Table: Region | Total Reviews | Avg Rating | % Positive | % Negative | Sentiment Rank

Sort by average rating ascending (worst first) so leadership sees risks at the top.

**Insight paragraph:** Which regions need the most marketing or operational attention?

---

### Section 5: Theme & Driver Analysis

This is the centerpiece of the marketing intelligence report.

**Top Positive Drivers** — table of themes driving high ratings:

| Theme | Frequency | Sentiment Impact | Sample Quote |
|-------|-----------|-----------------|--------------|

**Top Negative Drivers** — table of themes driving low ratings:

| Theme | Frequency | Sentiment Impact | Sample Quote |
|-------|-----------|-----------------|--------------|

**Insight paragraph (3–5 sentences):** What are the 2–3 most important takeaways for marketing? Which positive themes should be amplified? Which negative themes are most damaging to acquisition and retention?

---

### Section 6: Experience Gap Analysis

Two subsections rendered as alert cards:

**Operational Gaps** (🔴 red left-bordered cards):
- One card per identified gap
- Card content: gap name, description, frequency, affected brands/regions, recommended operational action

**Perception Gaps** (🟡 amber left-bordered cards — use `#F59E0B` for amber):
- One card per identified perception gap
- Card content: gap name, description, what customers expect vs what they experience, recommended messaging adjustment

If no gaps are found in a subsection, say so explicitly.

---

### Section 7: Location Intelligence

**Top Performing Locations** (from Query 6):

Table: Center | Brand | Location | Region | VP | Avg Rating | % Positive | Total Reviews

These are marketing proof points — locations where the brand is delivering on its promise.

**Underperforming Locations — Reputation Risk** (from Query 7):

Table: Center | Brand | Location | Region | VP | Avg Rating | % Negative | Total Reviews

Flag any center with avg_rating < 2.5 or pct_negative > 60% with a ⚠️ icon. These require immediate reputation management attention.

**Outlier Alert:**
Identify any center with both **high review volume** (top 25% by total_reviews) and **high negative sentiment** (pct_negative > 40%). These are the highest-risk locations for brand damage at scale — render as a prominent alert card.

---

### Section 8: Weekly Sentiment Trend

A visual week-by-week trend table (from Query 8) showing how sentiment evolved over the 30-day window:

| Week | Total | Avg Rating | % Positive | % Negative | Trend |
|------|-------|------------|------------|------------|-------|

Use ▲ / ▼ / — symbols in the Trend column to show week-over-week direction for avg rating.

**Insight paragraph:** Is sentiment improving or declining over the period? Were there any notable spikes (positive or negative) that may correlate with campaigns, events, or operational changes?

---

### Section 9: Voice of Customer

Present 6–10 curated quotes in three groups:

**Positive Reinforcement** (🟢 green left-bordered cards):
Quote text, center name, brand, region, star rating, source. Frame each quote as a marketing asset — note what it could be used for (social proof, ad copy, testimonial).

**Pain Points** (🔴 red left-bordered cards):
Quote text, center name, brand, region, star rating, source. Note the theme it represents and its marketing implication (acquisition risk, churn driver, etc.).

**Unexpected Insights** (🔵 blue left-bordered cards — use `#3B82F6`):
Quote text, center name, brand, region, star rating, source. Explain why it's surprising and what opportunity or risk it signals.

---

### Section 10: Marketing Action Recommendations

Three groups of action cards, each rendered as a card with a priority badge (🔴 High / 🟡 Medium / 🟢 Low):

**Marketing & Messaging Actions:**
- Specific, actionable recommendations for campaigns, content, or messaging
- Which positive themes to amplify, which platforms to prioritize, targeting signals
- Example: "Amplify 'birthday/special occasion' messaging on Google — it is the #1 positive driver across all brands"

**Experience Improvement Signals (for Ops/Product):**
- Top recurring pain points that marketing cannot fix alone
- Location-specific escalations
- Example: "Booking friction is the top negative driver in the Southeast — coordinate with Ops on lane reservation UX before next campaign"

**Reputation Management Priorities:**
- Platform-specific response strategy
- Regions or brands needing proactive engagement
- Safety-critical escalations (if any) as critical ⚠️ alerts

---

## Interactive Filters (Client-Side JavaScript)

All filters are client-side JavaScript — no additional queries required.

**Filters to include:**

- **Brand Filter** — Dropdown populated dynamically from the data — filters any table or card containing a Brand column
- **Region Filter** — Dropdown populated dynamically — filters any table or card containing a Region column
- **Platform Filter** — Dropdown: All | Google | Yelp — filters Platform Intelligence and Voice of Customer sections
- **Sentiment Filter** — Button group: `All | 🔴 Negative | 🟡 Neutral | 🟢 Positive` — filters theme cards, VOC cards, and location tables by sentiment category
- **Star Rating Filter** — Button group: `All | ⭐1 | ⭐2 | ⭐3 | ⭐4 | ⭐5` — filters review-level tables

**Implementation rules:**
- Place the filter bar directly above each filterable section, not at the top of the page
- Active filter buttons: `--lse-red` background with white text
- Inactive filter buttons: light-gray background with charcoal text
- When filters return no results, show: *"No results match the selected filters."*
- Filters only affect tables and cards — KPI cards, summary paragraphs, and trend charts are never filtered

---

## Formatting Rules

- Bold key metrics and theme names for scannability
- Use emoji sparingly: ⭐ ratings, 🔴/🟡/🟢 sentiment tiers, ⚠️ critical alerts, 📊 summary sections, 📈 trend indicators, 💬 voice of customer
- Keep all insight paragraphs concise — these are for marketing leadership
- Do not editorialize — report what the data shows, then recommend what to do about it
- Flag safety-critical reviews (drink tampering, injury, discrimination, fraud) as ⚠️ **Critical Alerts** requiring immediate escalation — place these at the top of Section 10
- For perception gaps, always include a concrete messaging adjustment recommendation
- For operational gaps, always note that this requires Ops/Product coordination, not just marketing

---

## Step 4 — Send Report via Email

Send an email to andry@bellum.ai and JGonzalez@lsent.com using the sendgrid skill with:
- Subject: `LSE Sentiment Report — {Month} {YYYY}`
- Body: `Monthly sentiment intelligence report attached.`
- Attachment: the generated HTML file
- Sender name: Orca Reporting
