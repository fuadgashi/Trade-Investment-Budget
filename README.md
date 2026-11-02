# Trade Investment Budget

Power BI trade investment budget: a semantic model and report (PBIP, DAX, TMDL) for a
distributor of fast-moving consumer goods — what the selling price sets aside for bonuses,
in-store placements and marketing, what the customer terms, discounts and credit notes spend,
and the difference, in total and customer by customer.

**One model, one report.** `TradeInvestment.SemanticModel` holds the tables, relationships and
calculations; `TradeInvestment.Report` holds five pages and a tooltip on it — see
[report.md](docs/report.md). Open `TradeInvestment.pbip` in Power BI Desktop.

**Anonymised.** Every table, column and calculation is named in English, after a generic
source schema of SQL Server views ([data-source.md](docs/data-source.md)). Nothing in the
repository names a business, a person, a place or a source system behind the data, and the
connection parameters are placeholders. No rate is written anywhere: the rates come from the
source. The only name is the author's, in the [licence](#licence). The report is in English.

**No figures in the documents.** Totals, balances and row counts move with every refresh.
These documents hold structure, rules and decisions; the measurement behind a change lives
in its pull request.

## Documents

Read [maintenance.md](docs/maintenance.md) before you edit anything.

**The business and the model**

| Document | Covers |
|---|---|
| [business.md](docs/business.md) | the four questions the model answers, customers and outlets, brands and their rates, where the money comes from |
| [model.md](docs/model.md) | the snowflake schema: the facts with their grain and window, dimensions, infrastructure tables, relationships, query groups |
| [data-source.md](docs/data-source.md) | the source views and their columns, what the measures expect of them, the shared trade-terms expression, the parameters, running it on your own data |
| [measures.md](docs/measures.md) | the core measure rules: one sales measure on two bases, cost and prices, the time functions, formats |
| [dax-quality.md](docs/dax-quality.md) | how the DAX is kept to its standards: the audit, the colour measures, how a DAX change is tested, the measure register — measured by `scripts/dax_audit.py` |

**The subject**

| Document | Covers |
|---|---|
| [trade-investment.md](docs/trade-investment.md) | the price calculation, the trade terms, the annual bonus, secondary positions, the marketing budget, the total, and what the slicers reach |

**The report**

| Document | Covers |
|---|---|
| [report.md](docs/report.md) | pages, page navigation, the brand tooltip, slicers, the HTML KPI cards, labels, number formats, sort order |
| [mobile.md](docs/mobile.md) | the phone layout standard and its checker |

**Working on the project**

| Document | Covers |
|---|---|
| [maintenance.md](docs/maintenance.md) | engineering standards, naming, editing with Desktop open, the checks |
| [decisions.md](docs/decisions.md) | design decisions a future maintainer would otherwise re-litigate |
| [glossary.md](docs/glossary.md) | the model's own terms |
| [sources.md](docs/sources.md) | the external references behind every statement on how Power BI, DAX, SQL Server or a standard behaves |

**Sources.** A statement about how something outside the project behaves cites an entry
in [sources.md](docs/sources.md); a statement with no source is not written.

**Checks.** `scripts/docs_check.py` checks every link, anchor and citation in these
documents and in `CLAUDE.md`, and that each model object, page and script they name in
backticks exists. `scripts/model_check.py` checks that every reference in the model and the
report resolves, that relationships join keys of one type, and that nothing publishable
carries an e-mail address (but the licence's contact), an IP address or a currency symbol.
`scripts/dax_audit.py` measures the DAX against its standards and `scripts/rag_colors.py`
keeps the colour measures in step with their indexes — see
[dax-quality.md](docs/dax-quality.md). `scripts/mobile_check.py` checks the phone layout
against [mobile.md](docs/mobile.md). All five run on every pull request and every push to
`main` (`.github/workflows/checks.yml`).

## Licence

© 2026 Fuad Gashi. All rights reserved. Published to show the work and the method; no licence
to use, copy, modify or distribute any part of it is granted. To use any part of it, email
fuaddi.g@gmail.com — permission is usually given. See [LICENSE](LICENSE).
