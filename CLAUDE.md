# Trade Investment Budget — notes for Claude

Power BI project (PBIP). [README.md](README.md) maps every document in `docs/`. The rules live
in those documents; this file only says where to look.

## Before any change

- Read [docs/maintenance.md](docs/maintenance.md): editing while Desktop is open, what
  blocks loading, the engineering standards.
- Open the document for the subject in the table below and follow it.
- Work on a short-lived branch and a pull request; `main` is always deliverable.

## Never

- Rebuild sales inline: sales is `Sales Net Price`, without VAT `Sales Net Price ex VAT` —
  [docs/measures.md](docs/measures.md).
- Write a rate or a code list into a query, a measure, a label or a document: the rates and
  classifications come from the source — [docs/trade-investment.md](docs/trade-investment.md).
- Write the trade-term rules twice: they live once, in `TradeTerms` —
  [docs/data-source.md](docs/data-source.md).
- Write figures, or the name of a business, person, place or source system, anywhere in
  the repository — [README.md](README.md). The author's name and contact address appear in
  `LICENSE` and the README's licence section only.
- Type text into a visual or a card: every UI text is a row in `Labels` —
  [docs/report.md](docs/report.md).
- State how Power BI, DAX, SQL Server or a standard behaves without an entry in
  [docs/sources.md](docs/sources.md). No source, not written.

## Where each subject lives

| Changing | Read |
|---|---|
| `FactTradeTerm`, `FactTradeTermMonth`, `TradeTerms`, `DimBrand`, the bonus, position, budget and total measures | [docs/trade-investment.md](docs/trade-investment.md) |
| `Measure.tmdl`, `functions.tmdl`, `DimDate` | [docs/measures.md](docs/measures.md), [docs/dax-quality.md](docs/dax-quality.md), then [docs/trade-investment.md](docs/trade-investment.md) |
| any other table, `relationships.tmdl`, `model.tmdl`, a source query | [docs/model.md](docs/model.md), [docs/data-source.md](docs/data-source.md) |
| report pages, slicers, formats, the `HTML` cards, `Labels`, `LabelMeasures` | [docs/report.md](docs/report.md) |
| a `mobile.json`, the phone layout | [docs/mobile.md](docs/mobile.md) |
| `README.md`, `docs/` | [docs/sources.md](docs/sources.md) |

Not sure what a term means: [docs/glossary.md](docs/glossary.md).

## Checks

- After changing `README.md`, `CLAUDE.md` or `docs/`: `python scripts/docs_check.py`.
- After changing the model or the report: `python scripts/model_check.py` and
  `python scripts/dax_audit.py --strict`; after a DAX change, the test in
  [docs/dax-quality.md](docs/dax-quality.md#how-a-dax-change-is-tested).
- After changing an index's `RagColor` annotation: `python scripts/rag_colors.py` —
  [docs/dax-quality.md](docs/dax-quality.md#colour-measures).
- After changing a page, a visual or a `mobile.json`: `python scripts/mobile_check.py --strict`.
