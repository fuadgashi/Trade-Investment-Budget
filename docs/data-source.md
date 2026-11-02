# Data source, refresh and publishing

[← README](../README.md)

One source: **SQL Server**, reached through two Power Query parameters, `pServer` and
`pDatabase`, never a literal connection string. Both hold placeholders in the repository;
set them to the real server and database in Desktop.

## Source views

The queries read twelve views in schema `ti`. Each row below lists the columns the queries
select. Building these views over a source system is the only source-specific work: every
table, column and calculation of the model is named after them.

| View | One row is | Columns read |
|---|---|---|
| `ti.SalesLine` | a sales document line, with its header's fields | `DocumentTypeId`, `OrgUnitId`, `OutletId`, `DocumentDate`, `ProductId`, `Quantity`, `ListPrice`, `GrossPrice`, `NetPrice`, `UnitCost`, `VatRate`, `CountsAsSale` |
| `ti.PurchaseLine` | a purchase document line | `DocumentTypeId`, `OrgUnitId`, `DocumentDate`, `ProductId`, `Quantity`, `SupplierUnitPrice` |
| `ti.TradeTerm` | a trade term | `TermId`, `OutletId`, `BrandId`, `TermTypeId`, `IsPercentage`, `ValueExVat`, `Units`, `StartDate`, `EndDate` |
| `ti.TermType` | a term type | `TermTypeId`, `TermTypeName`, `TermGroup` |
| `ti.CreditNote` | a credit note line, split by brand | `DocumentDate`, `PartnerId`, `BrandId`, `Direction`, `ValueExVat` |
| `ti.ServiceInvoice` | a service or rent invoice to a supplier | `InvoiceDate`, `PartnerId`, `BrandId`, `ValueExVat` |
| `ti.Product` | a product | `ProductId`, `ProductName`, `SortOrder`, `BrandId`, `PiecesPerPack`, `PacksPerPallet`, `IsGoods` |
| `ti.Brand` | a brand | `BrandId`, `BrandName`, `DivisionName`, `DistributionCostRate`, `DistributionProfitRate`, `AnnualBonusRate`, `SecondaryPositionRate` |
| `ti.Partner` | a business partner | `PartnerId`, `PartnerName`, `SortOrder` |
| `ti.Outlet` | an outlet | `OutletId`, `OutletName`, `SortOrder`, `PartnerId`, `Channel`, `Region`, `City` |
| `ti.OrgUnit` | an organisational unit | `OrgUnitId`, `OrgUnitName`, `SortOrder` |
| `ti.DocumentType` | a document type | `DocumentTypeId`, `DocumentTypeName`, `IsSalesOut`, `IsPurchase` |

**Classifications and rates live in the source, not in the queries.** Which document types are
sales or purchases, which products are goods, which lines the sales rule keeps, which group a
term type belongs to, which brand a supplier's invoices are booked to, and every brand's four
rates: each is a column of a view, so no query carries a list of codes or a number. A new code
is classified where it is created; until it is, it counts in no total.

**What the measures expect of the views.** The queries pass these columns through, so the views
must hold them as the measures read them:

- **Prices on `ti.SalesLine` include VAT**: `ListPrice` before any discount, `GrossPrice` after
  the regular discount, `NetPrice` after the extra discount too. `VatRate` is a percentage
  (`20` for 20 %). `UnitCost` is without VAT.
- **`SupplierUnitPrice`** is what the supplier charged per unit, after its discounts and
  without VAT — not a landed cost.
- **The four brand rates are fractions** of the net price without VAT (`0.05` for 5 %).
- **`TermGroup`** is one of `Bonus`, `Position`, `Campaign` and `Promotion`; every placement
  type is a `Position`.
- **On a trade term, `ValueExVat`** is the percentage for a term billed on sales
  (`IsPercentage` = 1, `3` for 3 %), and the value per unit and month for a fixed term, with
  `Units` the number of units (shelves, displays).
- **`ti.TradeTerm` holds one row per term**: an outlet, a brand, a term type, a billing method
  and a period, each once. A term the register holds on several lines, or one amount entered on
  many outlets or brands, is made one row in the view.
- **`Direction`** on a credit note is `Issued` (to a customer) or `Received` (from a
  supplier). Values are without VAT and positive.

## Which model table reads which view

| Model table | Views |
|---|---|
| `FactSalesLine` | `ti.SalesLine`, `ti.Product` (goods) |
| `FactPurchaseLine` | `ti.PurchaseLine`, `ti.Product` (goods) |
| `FactTradeTerm`, `FactTradeTermMonth` | `TradeTerms` (`ti.TradeTerm`, `ti.TermType`, `ti.Outlet`), and for a percentage term's base `ti.SalesLine`, `ti.Product`, `ti.DocumentType` |
| `FactCreditNote` | `ti.CreditNote` |
| `FactServiceInvoice` | `ti.ServiceInvoice` |
| `DimProduct` | `ti.Product` |
| `DimBrand` | `ti.Brand` |
| `DimPartner` | `ti.Partner` |
| `DimOutlet` | `ti.Outlet`, `ti.Partner` |
| `DimOrgUnit` | `ti.OrgUnit` |
| `DimDocumentType` | `ti.DocumentType` |

`DimDate` and `Labels` are built inside the model, from no source.

## Queries

Every source table is a **native SQL query** inside `Sql.Database(..., [Query = "..."])`
with an explicit one-hour `CommandTimeout`. The SQL does the work: joins, filters, the date
window, the trade-term rules. The M after it sets types and marks keys.

- **A native query does not fold.** The mashup engine can't fold steps after a native SQL
  query [[pq-folding]](sources.md#pq-folding), so every filter lives inside the SQL text.
- **A rule used by two queries is written once.** `TradeTerms` is a text expression: the body
  of a common table expression that `FactTradeTerm` and `FactTradeTermMonth` both open with
  `WITH term AS (…)`. The window and the months each term has run are decided there.
- **The facts join `ti.Product` with INNER on purpose**, to leave out what is not goods. An
  inner join discards unmatched rows [[sql-from]](sources.md#sql-from): a line whose product is
  missing from the catalogue drops out with its value.
- **The fact window** is `DocumentDate >= DATEFROMPARTS(YEAR(GETDATE()) - 1, 1, 1)`: the prior
  and the current year, rolling on 1 January. `DimDate` restates the same window; change the
  two together.

## Running it on your own data

The repository ships with no data and placeholder connection parameters. To run it:

1. **Build the twelve views** in schema `ti`, with the columns above and the expectations
   they carry.
2. **Set `pServer` and `pDatabase`** in Desktop to the real server and database. Never commit
   them.
3. **Refresh**, then test as [dax-quality.md](dax-quality.md#how-a-dax-change-is-tested) says.

## Refresh and publishing

The model is refreshed in Desktop, tested, and published from `TradeInvestment.pbip`:
publishing creates a semantic model with the name of the file, and republishing replaces the
model of that name [[desktop-publish]](sources.md#desktop-publish). The order is always:
commit → refresh → test → publish.

**Credentials** live in each developer's Desktop and in the service, never in the repository.
