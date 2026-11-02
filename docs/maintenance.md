# Maintenance

[← README](../README.md)

## Who maintains what

| Layer | Where |
|---|---|
| which document types are sales or purchases | `ti.DocumentType` flags, in the source |
| which products are goods, and which sales lines the sales rule keeps | `ti.Product[IsGoods]`, `ti.SalesLine[CountsAsSale]`, in the source |
| each brand's division and four rates | `ti.Brand`, in the source |
| which group a term type belongs to | `ti.TermType[TermGroup]`, in the source |
| which brand a supplier's service invoices are booked to | `ti.ServiceInvoice[BrandId]`, in the source |
| the outlets' channel, region and city | `ti.Outlet`, in the source |
| one row per trade term | `ti.TradeTerm`, in the source |
| which trade terms count, and the months each has run | the `TradeTerms` expression |
| every UI text | `Labels` rows |

---

## Engineering standards

**Source layer**
- Connection through `pServer` / `pDatabase`: no literal connection string
- Explicit `CommandTimeout` on every `Sql.Database` call
- All shaping in native SQL; M limited to typing and documented steps
- Every source query carries a header saying what it loads
- No code list or rate inside a query: classifications and rates are columns of the source
- A rule two queries share lives once, in a shared expression
- Culture passed explicitly to every M text function that depends on it (`Date.MonthName`)
- No credentials in code

**Model layer**
- Snowflake schema as [model.md](model.md) describes it, single-direction relationships, no
  bidirectional filtering
- One marked date table; time intelligence never touches raw transaction dates
- Key columns share a data type across every relationship
- `summarizeBy: none` on keys, prices and rates
- Fact columns hidden unless the report uses them
- A description on every table, column, measure and function
- Shared logic in user-defined functions, English PascalCase

**Report layer**
- No text typed into a visual: every text is a label measure
- No saved selection that goes stale: date slicers carry relative filters, not dates
- Every visual has a phone layout that meets [mobile.md](mobile.md)

**Practice**
- Every change verified against the live model and, where it touches a figure, against the
  source in read-only SQL — totals compared before and after
- The measurement behind a change recorded in its pull request
- Short-lived branches and pull requests; `main` is always deliverable
- `scripts/docs_check.py`, `scripts/model_check.py`, `scripts/rag_colors.py --check`,
  `scripts/dax_audit.py --strict` and `scripts/mobile_check.py --strict` run on every pull
  request and every push to `main` (`.github/workflows/checks.yml`). A red run is a finding to
  fix, not to merge past
- A DAX change is tested as [dax-quality.md](dax-quality.md#how-a-dax-change-is-tested) says

### Naming

**Everything is in English, in one vocabulary**: tables `Fact…` and `Dim…` after the source
views, columns as the views name them, measures by what they measure (`Bonus Spent`,
`Sales Net Price Actual Year`), UDFs in English PascalCase (`YearToDay`, `TermCustomers`), and
local `VAR`s in English snake_case with the `_` prefix (`_last_year_full`).

---

## Before you edit

**Desktop and an external edit can overwrite each other.** Desktop writes its changes to the
files only when you save [[pbip-external-editing]](sources.md#pbip-external-editing). So:

- **Save in Desktop before editing a file outside it.**
- **With the preview "Detect and reload external PBIP changes" on**, Desktop offers to apply
  an external change, which discards its unsaved changes
  [[pbip-external-editing]](sources.md#pbip-external-editing).
- **Without it, close Desktop**, edit, reopen.
- Desktop does not reload `cache.abf`: a changed query needs a refresh
  [[pbip-external-editing]](sources.md#pbip-external-editing).
- `diagramLayout.json`, `report.json` and `mobileState.json` are **not supported** for
  external editing [[pbip-overview]](sources.md#pbip-overview). Edit them only with Desktop
  closed, and check the project opens.

After any Desktop save, read the diff before committing: a save can carry unrelated changes.

**An invalid TMDL edit blocks loading.** Desktop shows an error with the location to fix when
the change is applied or the project opened
[[pbip-semantic-model]](sources.md#pbip-semantic-model).

**`cache.abf` holds the model with its data** and is git-ignored
[[pbip-semantic-model]](sources.md#pbip-semantic-model): never commit it.
