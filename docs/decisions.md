# Design decisions

[← README](../README.md)

The rules a future maintainer would otherwise re-litigate, each with the reason it holds.

- **Everything is named in English, after the source views.** Tables, columns, measures and
  functions read the same words as the `ti` views ([data-source.md](data-source.md)), so a
  figure traces from a visual to the SQL under one name. Mapping a source system to these
  names happens once, in the views, not in the model.
- **Classifications and rates are columns of the source, not lists in the queries.** Sales and
  purchase document types, goods, term groups, the supplier-to-brand booking and the four brand
  rates: each is a column of a view. A code list or a rate inside a query goes stale silently.
- **A rule two queries share is written once.** The trade terms are picked in one shared
  expression, `TradeTerms`, that both trade-term facts read: two copies of the window or the
  month count would drift ([data-source.md](data-source.md#queries)).
- **The rates are per brand**, and every rate measure iterates `DimBrand`, so a rate stays
  correctly weighted at every grain: a division total is each brand's rate on its own sales.
- **Every pot is without VAT.** The rates, the terms, the credit notes and the discount are all
  on one basis, so Accumulated and Spent add up ([measures.md](measures.md#two-sales-bases)).
- **A negative marketing coefficient stays.** It means the price does not cover cost and the
  four rates; hiding it would overstate the budget
  ([trade-investment.md](trade-investment.md#the-price-calculation)).
- **The bonus and position cost falls in the month it arises**, not on the term's start date.
  An annual term starts on 1 January: dated at its start, the whole year's cost so far would sit
  in January and the other months would be empty. Campaigns and promotions stay dated at
  their start: each is one amount, agreed once.
- **The source holds one row per term.** Cleaning the register — duplicate lines, restated
  values, one amount entered on many outlets or brands — belongs in the source view, where the
  data is entered, not in the model; the model reads each term once, as it is
  ([data-source.md](data-source.md#source-views)).
- **Each placement type is its own term.** A secondary position, a shelf and a basket on the
  same outlet and brand each charge their own rate on the brand's sales; averaging their rates
  into one would hide what each costs.
- **One threshold is fixed in the model, not read from the source**: more than 10 % of sales
  marks a high-spend customer. It decides how the data is read, not what the price sets aside,
  so it is a rule, documented where it applies
  ([trade-investment.md](trade-investment.md#the-trade-terms)).
- **Margin and budget are kept apart.** No margin measure takes budget or spend. A margin after
  trade spend was considered and not built: it mixes the margin with marketing spend, which are
  read separately ([trade-investment.md](trade-investment.md#the-price-calculation)).
- **The price and volume effects add up to the change exactly**: each holds the other factor at
  the opposite period, so no remainder belongs to neither
  ([trade-investment.md](trade-investment.md#price--volume)).
- **A tooltip reads its own views of the roots** (`Lookup\Marketing`), so its captions and units
  change without touching a measure the pages use ([measures.md](measures.md#tooltip-measures)).
- **Every position counts once in the total.** The seasonal fixed positions are part of
  `Position Spent`, so the total adds the campaigns and the promotions, not `Trade Terms Spent`.
- **A customer's accumulated bonus is its own sales'.** Comparing a customer's bonus spent with
  the company's accumulated would hide the customers whose terms cost more than their sales set
  aside ([trade-investment.md](trade-investment.md#annual-bonus)).
- **Credit notes and service invoices stay unsplit by outlet.** They carry a customer or
  supplier and a brand, never an outlet; spreading them over outlets would invent a split the
  source does not hold ([trade-investment.md](trade-investment.md#what-the-slicers-reach)).
- **Per-unit figures are quantity-weighted**, value over quantity, so unit × quantity gives
  back the money ([measures.md](measures.md#cost-and-prices)).
- **Sort keys come from the source, not DAX**, in the same view as the name they sort, so the
  name and its key arrive together on every refresh ([report.md](report.md#sort-order)).
- **No section navigation.** Five pages need no menu: the tab row on each page is the whole
  navigation, so there is no menu page, drawer or bookmark to keep in step.
- **One language, kept in `Labels`.** The report is English only, yet every UI text — the
  cards' words included — stays a `Labels` row: titles never sit hard-coded on the canvas, and a
  language is added as data ([report.md](report.md#labels)).
- **The calendar restates the fact window instead of reading it.** A calendar derived from a
  fact query depends on that query, and goes short with no error when the query returns
  nothing.
