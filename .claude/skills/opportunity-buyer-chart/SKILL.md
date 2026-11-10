---
name: opportunity-buyer-chart
description: >-
  Build an Opportunity Buyer Chart from CaliberMind data: buying group,
  marketing and sales touches, intent, and key moments. Use for buyer charts,
  deal stories, or opportunity timelines.
---

# Opportunity Buyer Chart

One visual that tells the story of a single opportunity: how long marketing was involved with
the account before the deal, who on the buyer side engaged, and how marketing and sales worked
together to close it. Customers use it to report deal influence to their stakeholders.

This skill is instructions only. There are no scripts. All counting and grouping happens in
BigQuery through the CaliberMind connector, and you decide how to display the result.

## What you need

- **The CaliberMind connector**, logged in, with `get_semantic_schema` and `execute_query`. If
  it is not connected, tell the user and stop. Do not make up data or reuse data from another
  opportunity.
- **An opportunity** from the user: an ID, a name, or a company.
- **Optional skills:** if `calibermind-analyst` is available, follow its SQL rules for every
  query here. If a data visualization skill or an artifact or design skill is available, use it
  when you build the chart.

## Principles

- **Works on any instance.** Instances differ in event taxonomy, go-to-market motion, sales
  motion, and which titles matter. Work all of that out from the data on every run. Do not carry
  over class names, job titles, content names, or tool names from another instance, and do not
  treat the examples in these files as defaults. They are hypothetical.
- **Let BigQuery do the math.** Do not pull raw event rows and count them yourself. Each query in
  `references/queries.md` returns a small, aggregated result. If a result comes back with
  `truncated: true`, aggregate further (for example by month instead of week) and run it again.
  Never chart a truncated result.
- **Run straight through.** Do not stop to show the user the touch classification or ask them to
  approve it. Make the calls, build the chart, and mention any judgment calls briefly at the end.
  Only stop to ask when the opportunity itself is ambiguous (several matches).
- **Readable or it does not ship.** Look at what you built before you hand it over. See
  `references/chart-spec.md`.
- **Privacy.** `event_detail` can hold email bodies, and `event_name` on sales and meeting rows
  can hold email subjects and calendar titles. Use them only to classify. Never put that text, or
  any email address, on the chart, in speaker notes, or in any file you save.

## Workflow

### 1. Load the schema

Call `get_semantic_schema`. Confirm that the tables and columns in `references/queries.md` exist
in this instance (`cm.cm_opportunity`, `cm.cm_eventstats`, `cm.cm_person`). If a column is
missing or named differently, adapt the queries to what the schema shows.

### 2. Find the opportunity

Run query 1 in `references/queries.md`. If more than one opportunity matches, show the matches
and ask which one. Note the ID, name, company, amount, type, stage, created date, close date,
and won and closed flags.

The chart is built for **new business** opportunities. Deal types are not standardized across
instances, so do not filter strictly on type. If the opportunity is clearly an upsell, renewal, or
expansion, build it anyway on the full account history, label it as an expansion, and tell the
user the timeline includes earlier customer activity. Open opportunities run through today.
Closed lost opportunities work too; the title and the closing milestone change to match.

### 3. Profile the account's events

Run query 2 (signatures). It returns one row per event class, type, and system on the account
up to the close date, with counts, a date range, and sample names, URLs, and details. For any
large signature that looks mixed, run query 3 to see the name patterns inside it.

### 4. Classify

Read `references/touch-classification.md`. Decide the bucket and marketing value for every
signature, then write them as the `classified` CTE described there. Every later query starts
with that CTE, so you write the classification once and paste it in. Keep the mapping to
yourself; do not show it to the user.

### 5. Pull what the chart needs

Run queries 4 through 8 in `references/queries.md`, in order:

- **4. People:** each person's marketing, sales, and meeting counts, plus first touch dates.
  Use it to pick the key people (see "Choosing key people" below).
- **5. Weekly activity per lane:** counts by week, lane, and marketing or sales. Pass in the key
  people you chose.
- **6. Intent and meetings:** intent dates for the account lane, and meeting days per lane.
- **7. Milestones:** first signal, first known person, and the first time each milestone label
  appears.
- **8. Key moment candidates:** high and medium value marketing touches, each person's first
  marketing touch, and research bursts.

Check the results against what you saw in step 3. The totals should look plausible, the first
signal date should be real (not a record-created artifact), and the moment candidates should be
actual marketing touches. If something is off, fix the `classified` CTE and rerun. This loop is
cheap.

### 6. Write the content

Read `references/content-guide.md`, then write every piece of text: header, title, four stats,
lane labels, milestones, key moments, source line, and speaker notes.

### 7. Build, look, and fix

Read `references/chart-spec.md` and `references/theme.md`. Choose how to present the chart in
this environment, build it,
then look at it and fix what is hard to read. Repeat until it reads cleanly.

### 8. Hand off

Give the user the chart (or its link) and a short summary: the story in two or three sentences,
plus any judgment calls worth knowing. For example: "treated the intent surges logged under an
operational class as intent", "expansion deal, so the timeline includes earlier customer
history", or "only 3 marketing touches were strong enough for key moment cards". Do not list the
classification.

## Choosing key people

Show up to 5 people with their own lanes. Everyone else goes into one "Everyone else" lane.

1. Always keep the person whose first **marketing** touch came earliest. They usually start the
   marketing story.
2. Rank the rest by meetings first, then seniority, then total touches.
3. Give extra weight to personas that matter in this deal: ones the user names, or a function the
   instance clearly sells to whose key buyers are not senior by title (for example engineers in a
   technical sale).
4. Only named people get their own lane. Anonymous visitors stitched to the account go into
   "Everyone else".

Judge seniority from `job_level` first, then the title, and read titles as a person would.
"Vice President" is VP, not C-level, even though it contains "President". "Product Owner" and
"Account Owner" are not business owners. "Chief of Staff" is not a C-level buyer. Titles can be
non-standard or in another language. Use what the title means, not the words in it.

## Edge cases

- **No intent or early signal:** no compressed time axis; the timeline starts at the first touch.
- **No person-level marketing:** no key moment cards. Say so plainly in the hand off. Do not
  promote page views to fill the row.
- **Many people:** still 5 key lanes at most. Say in the hand off how many are in "Everyone else".
- **Very long active periods:** label the axis by quarter instead of by month.
- **Meetings logged twice:** count one meeting per person per day (query 6 does this).
- **Several opportunities on one account:** take milestones from the chosen opportunity only. Do
  not pin another opportunity's stage changes.

## Files

- `references/queries.md`: every SQL query, in workflow order.
- `references/touch-classification.md`: buckets, value tiers, and how to write the `classified`
  CTE.
- `references/content-guide.md`: how to write every piece of text on the chart.
- `references/chart-spec.md`: what the chart must show, how to choose a format, and the
  readability check.
- `references/theme.md`: the default colors and fonts, taken from the CaliberMind Style Guide.
