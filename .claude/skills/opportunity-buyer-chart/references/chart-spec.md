# The chart

This file says what the chart must show and how to judge whether it reads well. How to draw it
is up to you: pick the format, tools, and layout that work best in the environment you are in.

## Choosing a format

Use the best visual output your environment offers, in roughly this order of preference:

1. **A slide**, if you can make one: a slide or deck artifact type, or a presentation skill.
   Stakeholders usually put this chart in front of other people, so one 16:9 slide is the ideal
   shape.
2. **A web page or artifact**, such as HTML with inline SVG, if slides are not available.
3. **A rendered image**, if that is the only visual output you can produce.
4. **A text summary**, as a last resort: lanes, counts, milestones, and moments as a short
   table. Tell the user the environment could not render a chart.

If the user asks for a specific format, use it. For colors and fonts, use `theme.md` by default.
If the user names a different brand or design system, use that instead. If a data visualization
skill is available, follow its guidance on marks and legends, but keep the colors from the
theme.

Whatever the format, everything should fit in a single 16:9 frame without scrolling, and it
should be readable when projected.

## What it shows

Top to bottom:

1. **Header:** the header line, the title, and the 4 stats (`content-guide.md`).
2. **Time axis** from the first signal through the close (or today), with the milestones marked
   on it and labeled.
3. **Lanes**, one row each, in this order:
   - Account signals, if there are any account-level touches or intent.
   - Up to 5 key people, ordered by their first touch.
   - Everyone else, if anyone is left over.

   Each lane has its label and title on the left, and its marketing count and sales count
   (including meetings) in two columns on the right. Show a dash instead of 0.
4. **Key moment cards:** up to 5, numbered, along the bottom. Each one has the date, the
   person's first name, and the description.
5. **Footer:** a legend that lists only the mark types actually on the chart, and the source
   line.

Speaker notes go wherever the format supports them (slide notes, a caption, or a collapsed
section).

## Encoding

- **Marketing and sales** are two distinct colors. They must also stay distinct for color-blind
  readers and in grayscale prints: if their colors are close in lightness, separate them another
  way, such as an outline on one of them (`theme.md` says how). Within a lane, put marketing
  marks slightly above the lane's center line and sales marks slightly below, so they never sit
  on top of each other.
- **Volume:** mark size grows with the touches in that week or month. Use area, not radius, and
  cap the size so the busiest week does not swamp its neighbors.
- **Meetings:** a different shape (for example a diamond) on the sales side of the lane.
- **Intent:** a third color and shape on the Account signals lane.
- **Milestones:** a marker on the axis with a light vertical guide down through the lanes. Each
  milestone marker gets its own color from `theme.md`, with closed won always in the primary
  green. Make the close guide slightly stronger than the others.
- **Key moments:** a numbered badge on the chart at the moment's date and lane, with the same
  number on its card.

## Time axis

- If the first signal comes more than about 4 months before the first person touch (query 7),
  **compress the early period**. Give the long early stretch a short segment on the left with a
  visible axis break, label it by year, and give most of the width to the active period. Never
  squash early touches into a pile at the axis start.
- Otherwise use one continuous scale from the first signal to the end.
- Label the active period by month. If month labels would crowd (roughly more than 18 months),
  label by quarter ("Q1 26"). Show the year at least where it changes.
- Plot each week or month at its midpoint.

## Readability check

Before you hand it over, look at the actual output: view the rendered slide, page, or image if
your environment lets you take a screenshot or open the file. If you cannot view it, review your
layout carefully against this list instead, and tell the user you could not visually check it.

- No text overlaps other text, marks, or badges.
- No label wraps, is cut off, or runs outside its area or the frame.
- All text is large enough to read when projected. At 1920×1080, that means about 24px or more
  for everything, including axis labels and the legend.
- Text contrast is at least 4.5:1 against what is behind it (3:1 for very large text such as the
  title and the stat values).
- Milestone labels do not collide. If they do, shorten, merge, or drop milestones.
- Badges do not hide each other or sit on top of a dense cluster of marks. Move a badge slightly
  if needed, but keep it on its lane and inside the chart area.
- Every lane visibly has something in it. A lane that reads as empty is a sign that the
  classification or the people choice needs another look.
- Numbers on the chart match the query results.

Fix the cause (shorter text, fewer or merged milestones, different moment picks, a coarser time
bin) and check again. Repeat until it reads cleanly. Fonts that load from the web may not render
in your preview, so leave some slack in text widths.

## Revising later

Keep what you need to rebuild the chart in your working context: the opportunity ID, the
`classified` CTE, the key people, and the content you wrote. For a change, edit what changed,
rerun only the affected queries, rebuild, and check again. If you save any of it to files, apply
the privacy rule in SKILL.md: no event details, email subjects, meeting names, or email
addresses.
