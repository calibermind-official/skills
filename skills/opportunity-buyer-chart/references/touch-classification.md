# Classifying touches

CaliberMind instances do not label events consistently. The same kind of touch can sit under
different `event_class` values in different instances, and one class can mix several kinds of
touch. For example, an "Operational" class might hold intent data, sales email threads, and
system noise side by side. So never classify by class name alone. Read the evidence in each
signature and decide what the touch actually is.

Everything below is guidance for reasoning. None of it is a lookup table. If an instance does
something unexpected, trust the samples over these notes.

## The buckets

| Bucket | What it means | Where it shows on the chart |
|---|---|---|
| `marketing` | A person engaged with something marketing produced or ran: web pages, content, forms, webinars, events, email clicks, product usage in a product-led (PLG) motion, community activity | Marketing marks on that person's lane; counted as marketing |
| `account_marketing` | Marketing engagement tied to the account but no person: ad impressions or clicks reported at company level, account-level web sessions from reverse IP | Marketing marks on the Account signals lane |
| `intent` | Third-party or first-party research signals, usually account level: topic surges, review-site research, intent scores | Intent marks on the Account signals lane; not counted as touches |
| `sales` | A rep reaching the buyer, or the buyer replying: emails, calls, LinkedIn messages, sequence steps, tasks that record real contact | Sales marks; counted as sales |
| `meeting` | A meeting that actually happened with the buyer | Meeting marks; counted as sales |
| `milestone` | A state change worth pinning on the time axis: MQL, MQA, SQL, stage changes, became customer | Candidate milestones (you pick the final ones) |
| `noise` | Anything else: record created or updated, identity stitching, syncs, score changes, auto-replies, out of office, calendar declines or holds, bounces, unsubscribes, internal-only activity | Dropped |

When you are unsure between `sales` and `noise`, ask whether a human on either side did
something meaningful. A logged auto-reply is noise. A rep's personal follow-up is sales.

## How to read a signature

Each row from query 2 has class, type, system, counts, a date range, and samples of
`event_name`, `page_url`, and `event_detail`. Look at all of it:

- **System** often tells you the motion. A marketing automation or web analytics system usually
  means marketing, a sales engagement tool usually means sales, and an intent provider means
  intent. But a CRM can hold all three, so keep reading.
- **Name prefixes and tool tags** are strong cues. Sales engagement tools often stamp names with
  their own tag, direction arrows, or step labels. Calendar tools stamp "Invitation", "Updated
  invitation", "Accepted", "Declined", or "Hold".
- **Detail text** can show direction (from the rep's domain to the buyer's, or the reverse) and
  status. Use it only to classify. It can contain email bodies, so never copy detail text onto
  the chart, into speaker notes, or into any file you save.
- **URLs** separate marketing content (guides, webinars, events, pricing, demo pages, docs) from
  generic pages.
- **Person coverage** matters. A signature with no people is account level, which usually means
  `intent`, `account_marketing`, or `noise`.
- **Volume and timing** help. Hundreds of identical rows at the same timestamp are usually syncs.

If one signature mixes kinds (sales emails plus auto-replies, or real meetings plus declines),
give the signature its main bucket and split the rest out with name or URL rules.

## Value tiers for marketing touches

Marketing touches also get a `value` of `high`, `medium`, or `low`. Value decides which touches
become key moments, so think about what a stakeholder would recognize as meaningful:

- `high`: hand raises and deep engagement. Demo, contact, pricing, or trial requests; live event
  or webinar attendance; booth visits; in-person meetings marketing set up; gated content
  downloads in content-led motions; product activation milestones in PLG motions.
- `medium`: deliberate engagement with a specific asset. Email clicks, webinar replays, content
  page views (guides, reports, case studies), ad clicks by a known person, community posts.
- `low`: generic browsing. Homepage, blog index, careers, generic product pages, routine logins.

What counts as high value depends on the go-to-market motion, so infer it from the instance:

- *Hypothetical event-heavy company:* trade show badge scans and dinner RSVPs are high; webinar
  replays are medium.
- *Hypothetical PLG company:* a workspace invite or a first integration connected is high;
  routine app sessions are low.
- *Hypothetical account-based (ABM-heavy) company:* account-level ad engagement is the main
  marketing story, so the Account signals lane carries much of the weight. Do not inflate its
  value tier; let the lane show it.
- *Hypothetical outbound-led company:* marketing may be thin. That is fine. Show thin marketing
  honestly rather than promoting page views to high value.

Upgrade page views that hit meaningful content by matching on `page_url`.

## Writing the `classified` CTE

The base block in `queries.md` has three `CASE` expressions. Fill them in from your decisions:

```sql
CASE
  -- 1. Exceptions first: rows that must leave their signature's bucket
  WHEN LOWER(event_name) LIKE "%automatic reply%" THEN "noise"
  WHEN LOWER(event_name) LIKE "%out of office%" THEN "noise"
  WHEN event_class = "<calendar class>" AND LOWER(event_name) LIKE "declined%" THEN "noise"
  -- 2. Specific signatures (class + type) before broad ones (class only)
  WHEN event_class = "<class>" AND event_type = "<type>" THEN "meeting"
  WHEN event_class = "<class>" THEN "marketing"
  WHEN event_system = "<intent provider>" THEN "intent"
  ELSE "noise"
END AS bucket,

CASE
  WHEN event_class = "<form class>" THEN "high"
  WHEN LOWER(page_url) LIKE "%/webinar%" OR LOWER(page_url) LIKE "%/guide%" THEN "medium"
  ELSE "low"
END AS value,

CASE
  WHEN event_class = "<lifecycle class>" AND LOWER(event_name) LIKE "%mql%" THEN "MQL"
  ELSE CAST(NULL AS STRING)
END AS milestone_label
```

Rules:

- `CASE` stops at the first `WHEN` that matches, so put exceptions and specific rules above broad
  ones.
- Anything that matches no rule falls to `"noise"`. Cover every signature that matters.
- Use exact `=` for class, type, and system values copied from query 2, since you saw the real
  strings. Use `LOWER(...) LIKE` for names and URLs. Use `REGEXP_CONTAINS` only when `LIKE`
  cannot express the pattern.
- `value` is only read for `marketing` rows, so its rules can ignore other buckets.
- Only the first occurrence of each `milestone_label` becomes a candidate (query 7).
- A `CASE` needs at least one `WHEN`. If you have no milestone rules, write
  `CAST(NULL AS STRING) AS milestone_label` instead.
- Null columns never match `=` or `LIKE`. If a signature's type or system is null, match it with
  `IS NULL`.

## Sanity checks before moving on

- Every signature with real buyer activity has a bucket. Large signatures left as noise should
  be noise on purpose.
- Meetings are only meetings that happened, not invitations, holds, or declines.
- Intent is kept even when the class it is logged under looks operational. Long-range intent is
  often the best evidence of how early marketing influence started.
- The ratio of marketing to sales looks plausible for the motion you inferred. If marketing is
  zero, check again before accepting it.
