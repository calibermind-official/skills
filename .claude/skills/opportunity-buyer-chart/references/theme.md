# Theme

The default look for the chart, taken from the CaliberMind Style Guide. Use it unless the user
asks for a different brand or names a design system. Contrast figures are measured against the
white background.

## Colors

| Role | Hex | Notes |
|---|---|---|
| Background | `#FFFFFF` | Keep the background light. |
| Alternating lane band | `#F8FAFC` | Brand off-white, behind every other lane. |
| Card background | `#FFFFFF` | Key moment cards. |
| Card border, grid lines, axis | `#DDE2E8` | Not in the style guide, which has no light neutral. Use muted text for the close guide. |
| Heading text | `#000C1A` | 19.7:1. Title, stat values, lane names, milestone labels. Also milestone markers and number badges. |
| Body text | `#4D5865` | 7.2:1. Header line, lane subtitles, card body, legend. |
| Muted text | `#6A7889` | 4.5:1. Axis labels and the source line only. |
| Title accent | `#69E2AE` with a `#000C1A` outline | The emphasized title words ("closed won") in the closed won green, directly on the white background (no panel behind them). Green alone is too faint on white, so give the letters a thin `#000C1A` outline drawn behind the fill (about 3px stroke at a 54px title, SVG `paint-order: stroke`). Only for the large title words, never body text. |

### Data and milestone colors

These follow the style guide's chart palette in its order: `#10BCE1`, `#23D186`, `#4D5865`,
`#F154A3`, `#9165F7`, `#F8E963`, `#56F0FF`, `#76FFC4`, `#6A7889`, `#FFBDF7`, `#B392FF`,
`#FFF7AE`. The data series take the first four. The milestones continue from the fifth.

| Role | Hex | Notes |
|---|---|---|
| Marketing | `#10BCE1` | Palette 1. A circle with no outline. |
| Sales | `#23D186` | Palette 2. A circle with a `#000C1A` outline (see below). |
| Intent | `#4D5865` | Palette 3. A square on the Account signals lane. |
| Meeting | `#F154A3` | Palette 4. A diamond on the sales side of the lane. |
| Closed won | `#69E2AE` | Brand primary green, with a `#000C1A` outline. Always this color, whatever its position. |
| Other milestones | Palette 5 onward | In date order: `#9165F7`, `#F8E963`, `#56F0FF`, `#6A7889`, and so on. Skip `#76FFC4`, so closed won stays the only green milestone. A closed lost or "Today" close milestone takes its place in this order. |

Marketing and sales are close in lightness (luminance 0.42 and 0.48), so color alone will not
separate them in grayscale or for color-blind readers. Keep marketing marks above the lane's
center line and sales marks below it, and give every sales mark a thin `#000C1A` outline.

Rules from the style guide:

- **No green on white for text or thin lines.** The brand greens (`#69E2AE`, `#23D186`,
  `#76FFC4`) only appear as filled marks with a `#000C1A` outline, the outlined title accent, or on
  the navy `#001022`, where `#69E2AE` reads at 11.9:1.
- **Light fills need an outline.** Any mark or milestone filled with a palette color lighter than
  `#9165F7` (luminance above 0.3) gets a thin `#000C1A` outline so it stays visible on white.
  Exception: marketing marks (`#10BCE1`) get no outline, so the outline stays the cue that tells
  sales apart from marketing.
- **Data colors are not text colors.** None of the palette colors except `#4D5865` and `#6A7889`
  are dark enough for small text. Label the "Marketing" and "Sales" columns, the milestones, and
  the legend in heading text, next to a swatch or mark in the color.
- Badge numbers sit in white text on `#000C1A`.

## Type

| Use | Font | Notes |
|---|---|---|
| Title, stat values, lane names, milestone labels | Circular Std Medium | `#000C1A`. |
| Body text, card text, legend, axis labels | Circular Std Book | `#4D5865`. |
| Header line, column headers, small caps labels | Roboto Mono, all caps | `#000C1A`. The style guide's H6. |
| A callout, if the format has one | Courier Prime | `#69E2AE` or white on a `#001022` panel. Optional; most charts do not need one. |

Circular Std is a licensed font, not on Google Fonts. Use it if it is installed or the format
provides it. Otherwise fall back to DM Sans (Google Fonts, weights 400 and 500), then Helvetica,
Arial, sans-serif. Roboto Mono and Courier Prime are on Google Fonts.

Use weight 500 (Medium) for headings and stat values, and 400 (Book) for everything else.

## Logo

Do not redraw the CaliberMind logo. Use an official logo file only if the user provides one or
the format already includes it.

## Using it with other formats

- **Slide or deck types:** if the format has its own theme settings, map the roles above onto
  them rather than overriding each element.
- **A design system the user chooses:** it replaces this file. Keep this file's rules: marketing
  and sales stay distinct without color (position and outline), data colors are not used for
  small text, closed won gets its own color, and intent gets its own shape.
