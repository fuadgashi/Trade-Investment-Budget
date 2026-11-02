# Core measure rules

[← README](../README.md)

## Sales is one measure

Sales anywhere in this model means **`Sales Net Price`**: quantity × net price across
`FactSalesLine` under one filter, `DimDocumentType[IsSalesOut] = TRUE`. The sales rule — goods
only, the lines the source counts as sales — is applied in the source query, so no measure
repeats it. **Never rebuild this measure inline; call it.**

## Two sales bases

The prices on `FactSalesLine` include VAT, as the source stores them: `ListPrice` (before any
discount), `GrossPrice` (after the regular discount) and `NetPrice` (after the extra discount
too).

Beside `Sales Net Price` runs a family named **`ex VAT`**, which takes the VAT out line by line
with `FactSalesLine[ExVatFactor]`, `1 / (1 + VAT rate / 100)` from the source query.

- **Every trade-investment rate is charged on `Sales Net Price ex VAT`**, and every pot is
  without VAT — see [trade-investment.md](trade-investment.md#words-and-rates).
- **`Gross Margin` divides by the ex-VAT figure.** A margin read against sales with VAT would
  not reconcile.
- Putting both bases in one visual compares two things silently; nothing errors.

## Cost and prices

- **Cost is without VAT.** `FactSalesLine[UnitCost]` is the average cost the source posts to
  the cost of sales; `Sales Cost Price` is quantity × it. It is never grossed up.
- **Every per-unit figure is value over quantity**, never an average over lines: unit × quantity
  must give back the money. An average over lines counts a one-piece line as much as a
  thousand-piece one. `Cost Price`, `Regular Price`, `Gross Price ex VAT`, `Supplier Price` and
  the four per-unit rates all follow it.
- **The supplier price is what the supplier charged**, after its discounts and without VAT
  (`FactPurchaseLine[SupplierPrice]`), not a landed cost.

## Time windows are functions

A time window is a call to a user-defined function, never written out in a measure.

| Function | Window |
|---|---|
| `YearToDay` | the date slicers' window within the current year, cut at today |
| `PrevYear` | the window one year back, inside last calendar year |

A prior year **to date** is `YearToDay` over `PrevYear`: last year, cut at today's date one year
back. The price and volume effects of the brand tooltip compare like for like on it.

- **Actual Year** in a measure name is `YearToDay`; **Prior Year** is `PrevYear`;
  **Prior Year YTD** is both.
- **The trade-investment pots are year to date**: the bonus, position and total figures are
  `YearToDay` of their base measure, so the date slicers narrow them within this year.

## Where a measure lives

Measures are foldered by job, not by page, so a new page adds no folder:

| Folder | Holds |
|---|---|
| `Calculation Roots\…` | the building blocks: sales, prices, the price calculation, profit |
| `Sales\…`, `Prices\…`, `Margin\…` | the roots in a time window: `Actual Year`, `Prior Year` |
| `Trade Marketing\…` | the pots: bonus, positions, the budget, credit notes, discounts, the total |
| `Lookup\…` | thin views of a root for the tooltip ([below](#tooltip-measures)) |
| `Formatting\…` | the colour measures and the slicer filter, hidden |

## Tooltip measures

The brand tooltip reads its own measures, in `Lookup\Marketing`: each is a root or a small
calculation over roots (`Marketing Product Sales` is `Sales Net Price Actual Year`,
`Budget Spent | Tooltip` is `Budget Spent Actual Year`). A view carries its own caption and its
unit in the format — "of sales", "of accumulated", "M" — without moving a measure the pages
depend on. **Never point a page visual at a `Lookup` view, and never move a root into
`Lookup`.** The value stays a number, so sorting, totals and colour still work.

## Colours

`RagColor` turns a value into a theme colour name — `good`, `bad` or `neutral` — so colours
follow the theme. An index's colour measure is generated from an annotation — see
[dax-quality.md](dax-quality.md#colour-measures).

## Formats

- **`NumberFormat`** shows at most three significant digits with K, M or B, as the measure's
  whole dynamic format string; it returns a quoted literal, so the measure stays a number
  [[dynamic-format-strings]](sources.md#dynamic-format-strings).
- **No currency symbol.** Values are plain numbers; the report shows no currency.
- **The HTML cards return text.** The KPI cards are measures of the `HTML` table that return
  the HTML the card draws, built by `HtmlBreakdownCard`, `HtmlTargetCard` and `HtmlInfoCard`;
  they carry no format — see [report.md](report.md#kpi-cards).
