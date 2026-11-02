# The model

[← README](../README.md)

**Snowflake schema.** Facts and dimensions are separate tables. Two dimensions also sit one
step behind another: `DimBrand` behind `DimProduct`, and `DimPartner` behind `DimOutlet`. Both
are joined directly too, by the facts that carry no product or outlet: the credit notes and
service invoices, and for brands the trade terms. Every relationship is many-to-one
and single-direction, from the many side to the one side; none filters both ways.

## Facts — grain and window

| Fact | One row is | Window |
|---|---|---|
| **`FactSalesLine`** | a sales document line, goods only, that the sales rule keeps: an outlet, a product, a quantity, the list, gross and net price, the unit cost | 1 January of the prior year onwards |
| **`FactPurchaseLine`** | a purchase document line, goods only: the supplier price per unit | 1 January of the prior year onwards |
| **`FactTradeTerm`** | a trade term: an outlet, a brand, a term group, a period, what it has cost up to today | terms that start in the prior or the current year |
| **`FactTradeTermMonth`** | a month of a bonus or position term, with that month's cost | the same terms |
| **`FactCreditNote`** | a credit note line, issued or received, booked on a brand | 1 January of the prior year onwards |
| **`FactServiceInvoice`** | a service or rent invoice to a supplier, booked on the supplier's brand | 1 January of the prior year onwards |

How the two trade-term facts are built is in
[trade-investment.md](trade-investment.md#the-trade-terms).

## Dimensions

| Dimension | Describes |
|---|---|
| `DimProduct` | products, with their brand and packing |
| `DimBrand` | brands, with their division and the four rates the price sets aside |
| `DimPartner` | business partners: the customers that own the outlets, and the suppliers on credit notes and service invoices |
| `DimOutlet` | outlets, with their customer, channel, region and city |
| `DimOrgUnit` | organisational units: the warehouses and vans documents are booked in |
| `DimDocumentType` | document types, with the flags `IsSalesOut` and `IsPurchase` |
| `DimDate` | the calendar: one row per day of the prior and the current year, marked as the date table |

## Infrastructure

| Table | Role |
|---|---|
| `Measure` | the measure host: every business measure lives here |
| `HTML` | the KPI cards: each measure returns the HTML of one card — see [report.md](report.md#kpi-cards) |
| `Labels` | every user-facing string of the report, one row per owner, label type and culture |
| `LabelMeasures` | one text measure per label, each a `LabelText` lookup against `Labels`; also `User Culture` — see [report.md](report.md#labels) |

## Relationships

| From (many) | To (one) |
|---|---|
| `FactSalesLine[Date]`, `FactPurchaseLine[Date]`, `FactCreditNote[Date]`, `FactServiceInvoice[Date]` | `DimDate[Date]` |
| `FactTradeTerm[StartDate]`, `FactTradeTermMonth[Month]` | `DimDate[Date]` |
| `FactSalesLine[ProductId]`, `FactPurchaseLine[ProductId]` | `DimProduct[ProductId]` |
| `FactSalesLine[DocumentTypeId]`, `FactPurchaseLine[DocumentTypeId]` | `DimDocumentType[DocumentTypeId]` |
| `FactSalesLine[OrgUnitId]`, `FactPurchaseLine[OrgUnitId]` | `DimOrgUnit[OrgUnitId]` |
| `FactSalesLine[OutletId]`, `FactTradeTerm[OutletId]`, `FactTradeTermMonth[OutletId]` | `DimOutlet[OutletId]` |
| `FactTradeTerm[BrandId]`, `FactTradeTermMonth[BrandId]`, `FactCreditNote[BrandId]`, `FactServiceInvoice[BrandId]` | `DimBrand[BrandId]` |
| `FactCreditNote[PartnerId]`, `FactServiceInvoice[PartnerId]` | `DimPartner[PartnerId]` |
| `DimProduct[BrandId]` | `DimBrand[BrandId]` |
| `DimOutlet[PartnerId]` | `DimPartner[PartnerId]` |

- **A single-direction relationship filters from the one side to the many side**
  [[relationships]](sources.md#relationships). A customer pick reaches its outlets, and through
  them the sales and the trade terms; it reaches its credit notes and service invoices
  directly.
- **An outlet pick does not reach the customer.** `DimOutlet` → `DimPartner` is single
  direction, so a channel, region or outlet pick leaves the credit notes and the service
  invoices whole — see [trade-investment.md](trade-investment.md#what-the-slicers-reach).
- **A brand reaches every fact**: sales and purchases through the product, the trade terms,
  credit notes and service invoices directly. The division and brand slicers bound them all.
- **`FactTradeTerm[PartnerId]` is not related.** The term reaches the customer through its
  outlet; the column is there for `TermCustomers`, which lists the customers with a term so the
  bonus and position measures can restrict sales to them with `TREATAS`.
- **Key columns share a data type** across every relationship: every key is a whole number.

## Query groups

Every query sits in one of five groups, declared in `model.tmdl`:

| Group | Queries |
|---|---|
| `Fact` | `FactSalesLine`, `FactPurchaseLine`, `FactTradeTerm`, `FactTradeTermMonth`, `FactCreditNote`, `FactServiceInvoice` |
| `Dimension` | `DimProduct`, `DimBrand`, `DimPartner`, `DimOutlet`, `DimOrgUnit`, `DimDocumentType` |
| `Model` | `DimDate`, `Measure`, `HTML`, `Labels`, `LabelMeasures` |
| `Parameters` | `pServer`, `pDatabase` |
| `Staging` | `TradeTerms`: the SQL of the trade terms, written once and read by both trade-term facts |

## Captions

`cultures/en-US.tmdl` gives fields short captions for the report — `Current Year`,
`Prior Year`, `Bonus %` — where the name is long. Captions translate object metadata, not data
values [[translations]](sources.md#translations); the report text that is data lives in
`Labels`.

## Data access options

`model.tmdl` sets two options of the M data engine, `legacyRedirects` and
`returnErrorValuesAsNull` [[data-access-options]](sources.md#data-access-options).

- **`returnErrorValuesAsNull`** makes an error in one cell of a query's result load as a
  null instead of failing the refresh, so a refresh can succeed with missing values. It does
  not touch DAX.
- **`legacyRedirects`** allows redirects from HTTPS to HTTP. The only data source is SQL
  Server, so there is nothing to redirect.
