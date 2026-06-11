# CaliberMind SQL Examples

Worked BigQuery patterns. Table/column names follow common CaliberMind defaults
but **always confirm against the live `get_semantic_schema`** for the current org
before reusing — names and keys vary between customers.

## 1. Ad performance metrics (CPM / CPC / CTR / CPL)

`cm_performance` is the ad-performance table (cost, impressions, clicks, leads).
It can be joined to other campaign data (e.g. attribution) to relate spend to
outcomes.

```sql
WITH performance AS (
  SELECT
    cm_performance.source AS platform,
    IFNULL(SUM(cm_performance.cm_cost), 0)     AS cost,
    IFNULL(SUM(cm_performance.impressions), 0) AS impressions,
    IFNULL(SUM(cm_performance.clicks), 0)      AS clicks,
    IFNULL(SUM(cm_performance.leads), 0)       AS leads
  FROM cm.cm_performance
  LEFT JOIN cm.cm_campaign
    ON cm_campaign.cm_id = cm_performance.campaign_id
  WHERE CAST(cm_performance.cm_start_date AS DATE)
        BETWEEN "2025-01-02" AND "2026-01-02"
    AND cm_performance.source IN ("Facebook","G2","Google Ads","LinkedIn","RollWorks")
  GROUP BY 1
)
SELECT
  platform AS Platform,
  ROUND(cost, 0) AS Cost,
  impressions AS Impressions,
  ROUND(IFNULL(SAFE_DIVIDE(cost, (impressions / 1000)), 0), 2) AS cpm,
  clicks,
  ROUND(IFNULL(SAFE_DIVIDE(cost, clicks), 0), 2) AS cpc,
  ROUND(IFNULL(SAFE_DIVIDE(clicks, impressions), 0) * 100, 2) AS ctr,
  leads,
  ROUND(IFNULL(SAFE_DIVIDE(cost, leads), 0), 0) AS cpl,
  ROUND(IFNULL(SAFE_DIVIDE(leads, clicks), 0) * 100, 2) AS `Conversion Rate Pct`
FROM performance;
```

Notes: `SAFE_DIVIDE` avoids divide-by-zero. The platform list is illustrative —
validate the real `source` values first if the user named specific platforms.

## 2. Campaign ROI: cost vs. attributed revenue

Build cost per campaign and attributed value per campaign, then join. Keep both
sides inside the same time window. Discover the real `model_name` before
filtering on it (it is case-sensitive); `"Even-Weighted"` here is only an
example.

```sql
WITH cost AS (
  SELECT
    cm_performance.cm_start_date,
    cm_campaign.cm_name,
    cm_performance.campaign_id,
    SUM(cm_performance.cm_cost) AS cost
  FROM cm.cm_performance
  LEFT JOIN cm.cm_campaign
    ON cm_campaign.cm_id = cm_performance.campaign_id
  WHERE cm_performance.cm_cost > 0
    AND CAST(cm_performance.cm_start_date AS DATE)
        BETWEEN "2025-01-01" AND "2026-01-01"
  GROUP BY 1, 2, 3
),
attrib AS (
  SELECT
    cm_campaign.cm_name AS Campaign_Name,
    cm_campaign.campaign_id,
    DATE(cm_insights_attribution.response_datetime) AS response_date,
    -- Won status lives on cm_opportunity, NOT on the attribution table.
    SUM(CASE WHEN cm_opportunity.cm_is_won = true
             THEN cm_insights_attribution.touch_value ELSE 0 END) AS attributed_bookings,
    SUM(cm_insights_attribution.touch_value) AS attribution
  FROM cm.cm_insights_attribution
  LEFT JOIN cm.cm_campaign
    ON cm_campaign.cm_id = cm_insights_attribution.campaign_id
  -- The schema marks attribution -> opportunity as a mandatory INNER JOIN.
  INNER JOIN cm.cm_opportunity
    ON cm_opportunity.cm_id = cm_insights_attribution.opp_id
  WHERE CAST(cm_insights_attribution.response_datetime AS DATE)
        BETWEEN "2025-01-01" AND "2026-01-01"
    AND cm_insights_attribution.model_name = "Even-Weighted"
    AND DATE_DIFF(cm_insights_attribution.opp_create,
                  CAST(cm_insights_attribution.response_datetime AS DATE), DAY) <= 90
  GROUP BY 1, 2, 3
)
SELECT
  attrib.Campaign_Name AS `Campaign Name`,
  attrib.campaign_id,
  attrib.response_date AS `Response Date`,
  attrib.attribution AS `Attribution`,
  cost.cost AS `Cost`
FROM attrib
LEFT JOIN cost
  ON cost.campaign_id = attrib.campaign_id
 AND attrib.response_date = cost.cm_start_date;
```

Reminder: `cm_insights_attribution` requires an attribution `model_name` filter
and has join restrictions — read its `!>` notes in the semantic schema.

## 3. Apostrophe-safe string filtering

Use double quotes for string literals so apostrophes don't break the query, and
do not escape them.

```sql
SELECT
  campaign_id,
  cm_name AS `Campaign Name`
FROM cm.cm_campaign
WHERE cm_name = "Smith's Guide"   -- double quotes handle the apostrophe safely
  AND LOWER(campaign_status) LIKE "%active%";
```
