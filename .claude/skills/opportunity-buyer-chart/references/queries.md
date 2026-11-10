# Queries

Run all of these with `execute_query`. They are BigQuery Standard SQL and read-only. They use
double-quoted string literals and `WITH` instead of temporary tables. Table and column names
are the usual CaliberMind ones. Confirm them against `get_semantic_schema` first, and adapt if
this instance differs.

Placeholders:

- `<TERM>` / `<term>`: what the user gave you to find the opportunity: an ID as given, and a
  name or company in lower case.
- `<OPP_ID>`: the opportunity's `cm_id` from query 1.
- `<KEY_PEOPLE>`: the person IDs you chose for their own lanes, as a quoted list, for example
  `"p1", "p2", "p3"`. Use `""` if there are none.

Every query returns aggregated rows, so results stay small. If one still comes back with
`truncated: true`, coarsen it (by month instead of week, or fewer columns) and run it again.

## 1. Find the opportunity

```sql
SELECT o.cm_id, o.cm_name, c.company_name, o.company_id, o.cm_amount, o.cm_type, o.cm_stage,
  FORMAT_DATE("%Y-%m-%d", DATE(o.cm_created_date)) AS created_date,
  FORMAT_DATE("%Y-%m-%d", DATE(o.cm_close_date)) AS close_date,
  o.cm_is_won, o.cm_is_closed
FROM cm.cm_opportunity o
LEFT JOIN cm.cm_companystats c ON c.company_id = o.company_id
WHERE NOT COALESCE(o.cm_is_deleted, FALSE)
  AND (o.cm_id = "<TERM>"
       OR LOWER(o.cm_name) LIKE "%<term>%"
       OR LOWER(c.company_name) LIKE "%<term>%")
ORDER BY o.cm_close_date DESC
LIMIT 25
```

## 2. Signatures

One row per class, type, and system on the account, with samples to classify from.

```sql
WITH opp AS (
  SELECT company_id,
    IF(cm_is_closed, DATE(cm_close_date), CURRENT_DATE()) AS end_date
  FROM cm.cm_opportunity WHERE cm_id = "<OPP_ID>"
)
SELECT e.event_class, e.event_type, e.event_system,
  COUNT(*) AS events, COUNT(DISTINCT e.person_id) AS people,
  FORMAT_DATE("%Y-%m-%d", MIN(DATE(e.event_datetime))) AS first_date,
  FORMAT_DATE("%Y-%m-%d", MAX(DATE(e.event_datetime))) AS last_date,
  ARRAY_AGG(DISTINCT SUBSTR(e.event_name, 1, 100) IGNORE NULLS LIMIT 8) AS sample_names,
  ARRAY_AGG(DISTINCT e.page_url IGNORE NULLS LIMIT 4) AS sample_urls,
  ARRAY_AGG(SUBSTR(REGEXP_REPLACE(e.event_detail, r"\s+", " "), 1, 160) IGNORE NULLS LIMIT 2)
    AS sample_details
FROM cm.cm_eventstats e
JOIN opp o ON e.company_id = o.company_id
WHERE DATE(e.event_datetime) <= o.end_date
GROUP BY 1, 2, 3
ORDER BY events DESC
```

## 3. Name patterns inside a mixed signature

Digits are masked so that similar names group together.

```sql
WITH opp AS (
  SELECT company_id FROM cm.cm_opportunity WHERE cm_id = "<OPP_ID>"
)
SELECT REGEXP_REPLACE(SUBSTR(e.event_name, 1, 60), r"\d", "#") AS name_pattern,
  COUNT(*) AS events
FROM cm.cm_eventstats e
JOIN opp o ON e.company_id = o.company_id
WHERE e.event_class = "<CLASS>" AND e.event_type = "<TYPE>"
GROUP BY 1
ORDER BY events DESC
LIMIT 25
```

## The base block

Queries 4 to 8 all start with this block. Paste in the `CASE` expressions you wrote in step 4
(see `touch-classification.md`). `lanes` assigns each touch to the account lane, a key person's
lane, or the "Everyone else" lane.

```sql
WITH opp AS (
  SELECT company_id, DATE(cm_created_date) AS created_date,
    IF(cm_is_closed, DATE(cm_close_date), CURRENT_DATE()) AS end_date
  FROM cm.cm_opportunity WHERE cm_id = "<OPP_ID>"
),
ev AS (
  SELECT DATE(e.event_datetime) AS event_date, e.person_id,
    e.event_class, e.event_type, e.event_system, e.event_name,
    REGEXP_REPLACE(e.page_url, r"[?#].*$", "") AS page_url
  FROM cm.cm_eventstats e
  JOIN opp o ON e.company_id = o.company_id
  WHERE DATE(e.event_datetime) <= o.end_date
),
classified AS (
  SELECT ev.*,
    CASE /* bucket rules */ ELSE "noise" END AS bucket,
    CASE /* value rules, marketing only */ ELSE NULL END AS value,
    CASE /* milestone label rules */ ELSE NULL END AS milestone_label
  FROM ev
),
named AS (
  SELECT c.*, p.name AS person_name, p.title AS person_title, p.job_level
  FROM classified c
  LEFT JOIN cm.cm_person p ON p.id = c.person_id
  WHERE c.bucket != "noise"
),
lanes AS (
  SELECT n.*,
    CASE
      WHEN bucket IN ("intent", "account_marketing") THEN "account"
      WHEN bucket = "marketing" AND person_id IS NULL THEN "account"
      WHEN person_id IN (<KEY_PEOPLE>) THEN person_id
      ELSE "rest"
    END AS lane
  FROM named n
)
```

## 4. People

Use this to choose the key people (SKILL.md, "Choosing key people"). For this query you have
not chosen anyone yet, so use `""` for `<KEY_PEOPLE>`.

```sql
<BASE BLOCK>
SELECT person_id, person_name, person_title,
  ANY_VALUE(job_level) AS job_level,
  COUNTIF(bucket = "marketing") AS marketing_touches,
  COUNTIF(bucket = "sales") AS sales_touches,
  COUNT(DISTINCT IF(bucket = "meeting", event_date, NULL)) AS meetings,
  FORMAT_DATE("%Y-%m-%d", MIN(event_date)) AS first_touch,
  FORMAT_DATE("%Y-%m-%d", MIN(IF(bucket = "marketing", event_date, NULL))) AS first_marketing
FROM lanes
WHERE person_id IS NOT NULL AND bucket IN ("marketing", "sales", "meeting")
GROUP BY person_id, person_name, person_title
ORDER BY meetings DESC, first_touch
LIMIT 200
```

People with a null `person_name` are anonymous and belong in "Everyone else".

## 5. Activity per lane

Weekly counts for the dots. Weeks start on Monday. If the active period is longer than about two
years, or the result is truncated, use `MONTH` instead of `WEEK(MONDAY)`.

```sql
<BASE BLOCK>
SELECT FORMAT_DATE("%Y-%m-%d", DATE_TRUNC(event_date, WEEK(MONDAY))) AS period, lane,
  COUNTIF(bucket IN ("marketing", "account_marketing")) AS marketing,
  COUNTIF(bucket = "sales") AS sales
FROM lanes
WHERE bucket IN ("marketing", "account_marketing", "sales")
GROUP BY period, lane
ORDER BY period, lane
```

Totals per lane for the count columns, and for the header stats. `sales_with_meetings` counts
each meeting once per person per day, which matches query 6.

```sql
<BASE BLOCK>
SELECT lane,
  COUNTIF(bucket IN ("marketing", "account_marketing")) AS marketing,
  COUNTIF(bucket = "sales")
    + COUNT(DISTINCT IF(bucket = "meeting", CONCAT(COALESCE(person_id, ""), CAST(event_date AS STRING)), NULL))
    AS sales_with_meetings,
  COUNTIF(bucket = "intent") AS intent,
  COUNTIF(bucket = "account_marketing") AS account_marketing
FROM lanes
GROUP BY lane
```

## 6. Intent and meetings

Intent dates go on the account lane. Meetings are counted once per person per day. The meeting
names are only there to help you spot a deal moment such as a pilot kickoff (see
`content-guide.md`). Describe the moment in your own words. Never copy a meeting name onto the
chart or into a file.

```sql
<BASE BLOCK>
SELECT "intent" AS kind, "account" AS lane, FORMAT_DATE("%Y-%m-%d", event_date) AS day,
  COUNT(*) AS n, CAST(NULL AS ARRAY<STRING>) AS sample_names
FROM lanes WHERE bucket = "intent"
GROUP BY day
UNION ALL
SELECT "meeting", lane, FORMAT_DATE("%Y-%m-%d", event_date), 1,
  ARRAY_AGG(DISTINCT SUBSTR(event_name, 1, 80) IGNORE NULLS LIMIT 2)
FROM lanes WHERE bucket = "meeting"
GROUP BY lane, person_id, event_date
ORDER BY day
```

## 7. Milestone candidates

Opportunity created and closed come from query 1. `first_person_touch` is when the active period
starts, which is what you need to decide whether to compress the early part of the time axis
(`chart-spec.md`).

```sql
<BASE BLOCK>
SELECT "first_signal" AS kind, CAST(NULL AS STRING) AS label,
  FORMAT_DATE("%Y-%m-%d", MIN(event_date)) AS day
FROM lanes WHERE bucket != "milestone"
UNION ALL
SELECT "first_person_touch", NULL, FORMAT_DATE("%Y-%m-%d", MIN(event_date))
FROM lanes WHERE lane != "account" AND bucket != "milestone"
UNION ALL
SELECT "first_known_marketing", NULL, FORMAT_DATE("%Y-%m-%d", MIN(event_date))
FROM lanes WHERE bucket = "marketing" AND person_name IS NOT NULL
UNION ALL
SELECT "data", milestone_label, FORMAT_DATE("%Y-%m-%d", MIN(event_date))
FROM lanes WHERE bucket = "milestone" AND milestone_label IS NOT NULL
GROUP BY milestone_label
ORDER BY day
```

## 8. Key moment candidates

Marketing touches by named people, of three kinds:

- `touch`: high or medium value touches, one row per person, day, and touch.
- `first`: each person's first marketing touch, whatever its value.
- `burst`: weeks where one person viewed 10 or more low-value pages. This is a research burst.

```sql
<BASE BLOCK>,
mk AS (
  SELECT * FROM lanes WHERE bucket = "marketing" AND person_name IS NOT NULL
)
SELECT "touch" AS kind, FORMAT_DATE("%Y-%m-%d", event_date) AS day, person_id, person_name,
  person_title, value, event_type, SUBSTR(event_name, 1, 100) AS event_name, page_url,
  COUNT(*) AS n, CAST(NULL AS ARRAY<STRING>) AS top_pages
FROM mk WHERE value IN ("high", "medium")
GROUP BY day, person_id, person_name, person_title, value, event_type, event_name, page_url
UNION ALL
SELECT "first", FORMAT_DATE("%Y-%m-%d", event_date), person_id, person_name, person_title,
  value, event_type, SUBSTR(event_name, 1, 100), page_url, 1, NULL
FROM mk
WHERE TRUE
QUALIFY ROW_NUMBER() OVER (PARTITION BY person_id ORDER BY event_date) = 1
UNION ALL
SELECT "burst", FORMAT_DATE("%Y-%m-%d", DATE_TRUNC(event_date, WEEK(MONDAY))), person_id,
  person_name, person_title, "low", NULL, NULL, NULL, COUNT(*),
  ARRAY_AGG(DISTINCT page_url IGNORE NULLS LIMIT 8)
FROM mk WHERE value = "low" AND page_url IS NOT NULL
GROUP BY 2, person_id, person_name, person_title
HAVING COUNT(*) >= 10
ORDER BY day
LIMIT 300
```

How to pick from the candidates (up to 5):

1. Start with `high` touches, then each person's `first` touch, then the last marketing touch
   before the opportunity was created, then `medium` touches and `burst` weeks.
2. Prefer touches by senior people and by the personas that matter in this deal.
3. If a high or medium touch falls inside a burst week for the same person, merge them into one
   moment covering that week.
4. Keep one moment per person per day.
5. Never pad. A plain page view only qualifies if it is that person's first touch or the
   person is senior. If fewer than 5 touches qualify, show fewer. If none qualify, show none.
