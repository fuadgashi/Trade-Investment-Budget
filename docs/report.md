# The report

[← README](../README.md)

**Five pages and a tooltip**, in English. The report opens on `Calculation`.

| Page | Asks | Main visuals |
|---|---|---|
| `Calculation` | how is the price built, and what does it leave for marketing? | supplier price, cost price, marketing coefficient and regular price by week, this year against last; the price build-up product by product |
| `Bonus` | does the bonus stay within what the price set aside? | four KPI cards, the risks card, accumulated against spent by month and by division, the bonus customers, the customers without a bonus |
| `Position` | does the placement spend stay within what the price set aside? | four KPI cards, the checks card, accumulated against spent by month, positions by type, the customers with a position, the check by brand |
| `Budget` | is the marketing budget in balance? | accumulated and spent by month, marketing by outlet, marketing and extra discount by brand and product |
| `Total` | what does trade investment cost in all? | three KPI cards, the checks card, accumulated against spent by month and by division, the investment per customer, every pot side by side |
| `Brand Tooltip` | the brand or product under the pointer | one table: sales, margin, budget, discount, price and volume effects |

What each figure means is in [trade-investment.md](trade-investment.md).

## Page navigation

Each page carries a **tab row** at the top right: one button per page, reading Price · Bonus ·
Positions · Budget · Total, each its text from a label measure (`PageNav1 | Title` …). The
first tab opens `Calculation`; its label is the page's subject, the price.
The tab of the page you are on is filled. There is no menu page and no section drawer; the five
tabs are the whole navigation. Every page also has a phone layout — see [mobile.md](mobile.md).

`Brand Tooltip` is a report-page tooltip, filtered by the data point under the pointer
[[report-tooltips]](sources.md#report-tooltips). It is hidden from the tab row.

| Opens from | Filtered by | Shows |
|---|---|---|
| the two brand / product tables of `Budget` (marketing accumulated; extra discount) | `DimProduct[ProductName]`, `DimBrand[BrandName]` | for a brand or a product: sales, gross margin and profit, budget spent and accumulated and the difference, extra discount and its rate, the price and volume effects and the verdict — see [trade-investment.md](trade-investment.md#price--volume) |

Its figures are the `Lookup\Marketing` measures — see
[measures.md](measures.md#tooltip-measures).

## Slicers

Every page has **five slicers in a row** (x = 725, 965, 1205, 1445, 1685), each under a
title button that reads a `… | SlicerTitle` label measure. A field sits in the same slot on
every page, and every slicer is synced across the pages.

| Slot | Slicer | Fields | Sync group |
|---|---|---|---|
| 1 | Date Range (`Calculation`: Month) | `DimDate[Date]` (`DimDate[Month]`) | `Date` (`Month`) |
| 2 | Division / Product | `DivisionName` › `BrandName` › `ProductName` | `Product` |
| 3 | Channel / Region | `Channel` › `Region` › `City` | `Channel` |
| 4 | Customer / Outlet | `PartnerName` › `OutletName` | `Customer` |
| 5 | Unit / Warehouse | `DimOrgUnit[OrgUnitName]` | `OrgUnit` |

- **The date slicer saves no dates.** It opens on the whole calendar, so it never goes stale at
  New Year; the measures cut at today themselves.
- **The product and unit slicers list only items with sales** in the selected dates, through a
  hidden filter on `Slicer Has Data`.
- **What a channel or outlet pick reaches** is in
  [trade-investment.md](trade-investment.md#what-the-slicers-reach): the credit notes and the
  service invoices carry no outlet.

## KPI cards

The cards on `Bonus`, `Position` and `Total` are the **HTML Content** visual, which draws the
HTML a measure returns. The measures live in the `HTML` table, one per card.

**The markup lives in functions, not in each measure. One function per way of drawing**: a
function always draws the same parts, and only the data changes from one card to the next. A
function is named and described by the form it draws, never by the subject a card uses it for.
The function does what is the same for every card; the measure passes what differs — the state,
the share, the formatted value, the texts.

| Function | Draws |
|---|---|
| `HtmlBreakdownCard( ValueText, ValueState, CaptionText, Parts, NoteText )` | a value split into parts: value and caption, a bar of the parts, a legend, a note |
| `HtmlTargetCard( ValueText, ValueState, CaptionText, FillShare, FillText, TargetShare, TargetText, NoteText )` | a value against a target: value and caption, a bar filled to a share with a target line, a legend, a note |
| `HtmlInfoCard( CheckRows )` | a list of checks, one row each: icon, text, value |

- **Two cards are their own HTML.** `Total Accumulated | HTML` and `Total Spent | HTML` draw a
  value with a legend and no bar; nothing else does, so there is no function for them, and the
  measure writes the markup itself.
- **One order inside every function**: first every calculation (cleaned inputs, and per-row
  values as columns added to the input table), then the style, then the HTML built only from
  finished values. Each part of the HTML is commented in the function.
- **The card's title and subtitle are the visual's**, never in the HTML: the visual's title and
  subtitle are set to the label measures `<card> | VisualTitle` and `<card> | VisualSubtitle`,
  and the HTML starts below them.
- **The texts are labels.** Every word a card shows is a `Labels` row on the card's own owner
  number, with its own label type (`HTMLSoFar`, `HTMLYearEnd` …), read with `LabelText`.

### HtmlInfoCard

`HtmlInfoCard` takes one table of rows, each `( state, text, value, format, order )`. The
measure decides the state and reads the text with `LabelText`; it passes the value as a number,
and the function writes it by the format: `compact` (`NumberFormat`), `signed` (`NumberFormat`
with a plus sign), `count`, `percent` or `percent1`.

| The card guarantees | Rule |
|---|---|
| **States** | `good` (a tick), `bad` (`!`), `neutral` (`–`), `rise` (▲) and `info` (`i`). Every state has its own symbol, so none is told apart by colour alone [[wcag-use-of-color]](sources.md#wcag-use-of-color). A row whose rule is above or below a centre passes `RagColor(...)` as its state; a count keeps its own `IF`. |
| **Colours** | Only the icon is coloured, in the theme's sentiment colours. Values and text stay in the text colour: the theme's green and amber on white are under 4.5:1, the minimum for normal text [[wcag-contrast]](sources.md#wcag-contrast). |
| **Space** | The rows share the height with equal space above the first, between each and below the last [[css-justify-content]](sources.md#css-justify-content). Every row is two lines high, so the icons are evenly spaced whether a text wraps or not. |
| **Text** | 13 px, the visual title's size, shrinking to no less than 11 px only when the rows do not fit [[css-clamp]](sources.md#css-clamp); the value bold, with digits of equal width so values line up [[css-tabular-nums]](sources.md#css-tabular-nums). |
| **Long text** | At most two lines, then … [[css-line-clamp]](sources.md#css-line-clamp). A row's text in `Labels` is written to fit two lines on the phone; the … is a safety net. |
| **Bad input** | An unknown or blank state reads `neutral`; a blank value or an unknown format reads `–`. |

### HtmlBreakdownCard and HtmlTargetCard

Both draw four parts, top to bottom: the value with its caption, a bar, a legend and a note.
They differ only in the bar.

- **`HtmlBreakdownCard`'s bar splits the value into parts.** `Parts` is a table, one row per
  part: `( label, value, share, colour, order )`. A part with a blank share fills the rest of the
  bar; a part with no label stays out of the legend.
- **`HtmlTargetCard`'s bar is filled to `FillShare` with a line at `TargetShare`**, both 0 to 1
  of the bar's width; the measure sets the scale. A blank `TargetShare` draws no line.
- **Colours** of the parts are names from the theme: `dark`, `blue`, `orange`, `amber`,
  `green`, and `grey` for a neutral rest; any other name reads grey. A `ValueState` of `good`
  or `bad` colours the value; text stays in the text colours.
- **The note** is text the measure builds; `<b>` marks its value. A blank note draws none.
- **Space and size** follow the rules of `HtmlInfoCard`: no padding and no header in the HTML,
  the parts spread with equal space [[css-justify-content]](sources.md#css-justify-content), 13
  px text shrinking to no less than 11 px [[css-clamp]](sources.md#css-clamp).

## Labels

**No text is typed into a visual.** Page titles and subtitles, tab labels, visual titles and
subtitles, slicer titles, the captions under values, every alt text and every word of a card
are label measures in `LabelMeasures`, or `LabelText` calls in a card, each reading one row of
`Labels`.

- **Owner numbers are one sequence**: pages 1–5, then the slicers, then the visuals. A number
  is never shared across owner types.
- **A missing row shows ⚠**, never a quiet fallback, so a gap is seen in review.
- **A rate is never written into a text.** A subtitle that quotes the bonus or position rate
  holds a `{BonusRate}` or `{PositionRate}` placeholder that its label measure fills in.
- **One language, English.** `User Culture` returns `en-US`, and `Labels` holds one row per
  label. To add a language: add its rows to `Labels`, and make `User Culture` return the
  viewer's culture.
- **Alt texts** are `Alt … | VisualSubtitle` label measures: the visual's kind, then what it
  shows.
- **Field names** come from captions (`cultures/en-US.tmdl`): a field shows its caption in a
  visual, never a name typed into it.

## Number formats

- **`NumberFormat`** on the values — see [measures.md](measures.md#formats). A dynamic format
  string keeps the measure numeric [[dynamic-format-strings]](sources.md#dynamic-format-strings).
- **Colours** are theme sentiment names — good, neutral, bad — from the colour measures, so
  they follow the theme. The theme is `cortnes-light.json`.

## Sort order

Four names sort by a hidden column that comes from the source:

| Column | Sorted by |
|---|---|
| `DimProduct[ProductName]` | `ProductSortOrder` |
| `DimPartner[PartnerName]` | `PartnerSortOrder` |
| `DimOutlet[OutletName]` | `OutletSortOrder` |
| `DimOrgUnit[OrgUnitName]` | `OrgUnitSortOrder` |
