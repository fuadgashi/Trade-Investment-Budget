# DAX quality

[← README](../README.md)

How the model's DAX is kept to its standards: nothing unused, no duplicate, one folder per
subject, every measure described and captioned, and every change proven not to move a number.

**Everything here is measured from the files.** `scripts/dax_audit.py` reads the model and the
report: every measure and function, and every place a visual, page or filter binds a measure.
It sees this report only: it cannot see Analyze in Excel or another report built on the
model.

| Command | Does |
|---|---|
| `python scripts/dax_audit.py` | prints every count below |
| `python scripts/dax_audit.py --strict` | the same, and exits 1 if a count that must be zero is not; runs on every pull request |
| `python scripts/dax_audit.py --register` | regenerates the [measure register](#measure-register) in this document |
| `python scripts/dax_audit.py --baseline baseline.dax` | writes the regression query of [How a DAX change is tested](#how-a-dax-change-is-tested) |
| `powershell -File scripts/dax_baseline.ps1 -QueryFile baseline.dax -OutCsv before.csv` | runs it against the model open in Desktop |
| `python scripts/dax_audit.py --compare before.csv after.csv` | compares two runs, measure by measure |
| `python scripts/dax_audit.py --json out.json` | writes every name behind every number |
| `python scripts/rag_colors.py` | writes the colour measure of every index that asks for one — see [Colour measures](#colour-measures) |

## What counts as used

A measure is **used** when a visual, page or filter of the report binds it, when a used
measure references it, or when a function a used measure calls reads it (`User Culture`, read
by `LabelText`). A KPI card's measures in `HTML` count like any other. A measure no visual binds is still **kept when the docs name it** in
backticks, or name its folder. Everything a kept measure calls is kept with it. Today every
analytical measure is bound by a visual or reached through one.

## Counts that must be zero

`--strict` fails the pull request when any of these is not zero.

| Count | What it catches |
|---|---|
| unresolved report references | a visual bound to a measure that no longer exists |
| unused measures, unused label measures, unused functions | DAX nothing reaches: removed, not kept "in case" |
| exact duplicates | two measures with the same expression, ignoring spaces and comments; a measure that differs only in the group it filters (`"Bonus"`, `"Position"`) is not a duplicate |
| text typed into visuals | a field renamed inside a visual instead of captioned in the model — see [report.md](report.md#labels) |
| fields sharing one caption in a visual | two columns of one table or matrix under the same heading |
| colour measures out of date | a colour measure that no longer matches its index's annotation |
| measures without a description, format or folder | the standards in [maintenance.md](maintenance.md#engineering-standards); a colour measure is text and carries the text format hint instead of a format |
| variables without the `_` prefix | a `VAR` not in English snake_case with the `_` prefix — see [maintenance.md](maintenance.md#naming) |
| colours not from `RagColor` | a theme colour name typed into a measure instead of returned by `RagColor`; a KPI card's row state (`HtmlInfoCard`) names a style of the card and is not counted |
| `IFERROR` | an error hidden instead of prevented: the model divides with `DIVIDE` and tests with `ISBLANK` |
| `FILTER` over a whole table | a filter argument built on a whole table: the model's filter arguments name columns |
| sales rebuilt inline | quantity × net price summed outside `Sales Net Price` and its ex-VAT twin — see [measures.md](measures.md#sales-is-one-measure) |
| visible measures without a caption | a field that would show its model name in a visual |

The audit also prints, without failing, the **shapes** that three or more measures share: the
expression with each measure reference replaced by a placeholder.

| Shape | Measures | Why they stay named measures |
|---|---|---|
| `YearToDay( [x] )` | 27 | the year-to-date window is already one function; each measure is a figure a card, chart or table shows under its own caption |
| `[x]` | 12 | the brand tooltip's copies (`Budget Spent | Tooltip` …): each carries the tooltip's own caption and format, while the base measure keeps the page's |
| `DIVIDE( [x], [y] )` | 9 | each rate pairs two named figures and carries its own format |
| `PrevYear( [x] )` | 6 | the same window, one year back |
| `RagColor( [x], 0 )` | 4 | the colour measures `rag_colors.py` writes — see [Colour measures](#colour-measures) |
| `[x] - [y]` | 4 | each difference pairs two named figures |
| `YearToDay( CALCULATE( [x], KEEPFILTERS( TREATAS( TermCustomers( … ), … ) ) ) )` | 4 | the bonus and position customers' sales and accumulated: one term group each |

## Colour measures

An index asks for a colour with one annotation on the measure:

- `annotation RagColor = UpIsGood`: a rise is good.
- `annotation RagColor = UpIsBad`: a rise is bad.

`python scripts/rag_colors.py` then writes its colour measure, hidden, in
`Formatting\RAG Color`: `RagColor( [index], centre )`, or with the sign turned for `UpIsBad`.
Edit the annotation, never the colour measure; `--check` runs on every pull request and fails
when a colour measure is out of step. A colour measure with its own rule — anything but one
`RagColor` call on one index — is written by hand and left alone by the script: today that is
`Marketing Verdict Color`, which colours the verdict by the signs of both the price and the
volume effect.

**What is coloured.** The marketing coefficient on `Calculation`, and on the brand tooltip the
budget difference, the volume and price effects and the verdict. Only rows with a clear
direction are coloured: a verdict takes the same test as its wording — both effects up is good,
both down is bad, a mixed one is neutral — so colour and text never disagree. Margin and the
shares are left uncoloured on purpose: whether they are good depends on the brand mix, a
business call rather than a number.

## How a DAX change is tested

Run before the change and after it, on the same refresh.

| Test | How | Passes when |
|---|---|---|
| **Numbers unchanged** | `--baseline`, then `dax_baseline.ps1` on the model open in Desktop, before and after; `--compare` | every kept numeric measure, every year: no value appears or vanishes, and a number differs by less than one part in 10¹² |
| **No broken binding** | `python scripts/model_check.py` and `python scripts/dax_audit.py` | 0 findings; `unresolved report references: 0` |
| **The audit's counts** | `python scripts/dax_audit.py --strict` | every count that must be zero is zero |
| **Visuals render** | open the pages listed under *Used on* in the register | no visual shows an error or a blank |
| **Documents** | `python scripts/docs_check.py` | 0 findings |

**The baseline needs data.** `dax_baseline.ps1` reads the model open in Power BI Desktop, so
it runs only once the model is connected to a source and refreshed — see
[data-source.md](data-source.md#running-it-on-your-own-data). The baseline query itself is generated from the files and
needs nothing: `--baseline` writes one query per folder, every measure by `DimDate[Year]`, and
the PowerShell script runs them as one query and saves one row per year. The comparison allows
a relative difference of 10⁻¹², so two runs may differ only in their last digits; a real change
moves a figure by far more.

## Measure register

Every analytical measure in its place: its folder, whether it is visible, and the visuals
that use it. Regenerated by `python scripts/dax_audit.py --register`; edit the model, not
this table.

<!-- register:start -->
151 analytical measures.

### Calculation Roots

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Bonus Per Unit | Price Calculation | yes | inside 1 measure (Marketing Coefficient) |
| Bonus Product Rate | Price Calculation | yes | inside 1 measure (Bonus Product Rate Actual Year) |
| Distribution Cost | Price Calculation | yes | inside 2 measures (Mark Up, …) |
| Distribution Cost % | Price Calculation | yes | inside 1 measure (Distribution Cost Actual Year %) |
| Distribution Profit | Price Calculation | yes | inside 2 measures (Mark Up, …) |
| Distribution Profit % | Price Calculation | yes | inside 1 measure (Distribution Profit Actual Year %) |
| Mark Up | Price Calculation | yes | inside 1 measure (Mark Up Actual Year) |
| Marketing Coefficient | Price Calculation | yes | inside 1 measure (Marketing Coefficient %) |
| Marketing Coefficient % | Price Calculation | yes | inside 2 measures (Marketing Coefficient Actual Year %, …) |
| Position Per Unit | Price Calculation | yes | inside 1 measure (Marketing Coefficient) |
| Position Product Rate | Price Calculation | yes | inside 1 measure (Position Product Rate Actual Year) |
| Cost Price | Prices | yes | Calculation matrix |
| Gross Price ex VAT | Prices | yes | inside 2 measures (Marketing Coefficient, …) |
| Packs per Pallet | Prices | yes | Calculation matrix |
| Pieces per Pack | Prices | yes | Calculation matrix |
| Regular Price | Prices | yes | Calculation matrix |
| Regular Price ex VAT | Prices | yes | Calculation matrix |
| Supplier Price | Prices | yes | inside 2 measures (Supplier Price Actual Year, …) |
| VAT Rate | Prices | yes | inside 1 measure (VAT Rate Actual Year) |
| Gross Margin | Profit | yes | inside 1 measure (Gross Margin Actual Year) |
| Gross Profit | Profit | yes | inside 2 measures (Gross Margin, …) |
| Sales Cost Price | Sales | yes | inside 1 measure (Gross Profit) |
| Sales Net Price | Sales | yes | inside 3 measures (Sales Net Price Actual Year, …) |
| Sales Net Price ex VAT | Sales | yes | inside 12 measures (Bonus Accumulated Period, …) |
| Sales Quantity | Sales | yes | inside 2 measures (Sales Quantity Actual Year, …) |
| Sales Selling Price | Sales | yes | inside 1 measure (Sales Selling Price Actual Year) |

### Formatting

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Budget Difference Color \| Tooltip | RAG Color | no | Brand Tooltip matrix |
| Marketing Coefficient Actual Year % Color | RAG Color | no | Calculation matrix |
| Marketing Price Effect Color | RAG Color | no | Brand Tooltip matrix |
| Marketing Quantity Effect Color | RAG Color | no | Brand Tooltip matrix |
| Marketing Verdict Color | RAG Color | no | Brand Tooltip matrix |
| Slicer Has Data | Slicer Filter | no | Bonus slicer, Budget slicer, Calculation slicer +2 |

### KPI Cards

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Bonus Accumulated \| HTML |  | yes | Bonus htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Bonus Customers \| HTML |  | yes | Bonus htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Bonus Risks \| HTML |  | yes | Bonus htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Bonus Spent Rate \| HTML |  | yes | Bonus htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Bonus Spent \| HTML |  | yes | Bonus htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Position Accumulated \| HTML |  | yes | Position htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Position Checks \| HTML |  | yes | Position htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Position Register \| HTML |  | yes | Position htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Position Spent Rate \| HTML |  | yes | Position htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Position Spent \| HTML |  | yes | Position htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Total Accumulated \| HTML |  | yes | Total htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Total Checks \| HTML |  | yes | Total htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Total Difference \| HTML |  | yes | Total htmlContent443BE3AD55E043BF878BED274D3A6865 |
| Total Spent \| HTML |  | yes | Total htmlContent443BE3AD55E043BF878BED274D3A6865 |

### Lookup

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Brand Supplier Name | Brand | yes | Brand Tooltip matrix |
| Budget Accumulated \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Budget Difference \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Budget Spent \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Extra Discount Rate \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Extra Discount \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Marketing Budget Spent Of Accumulated | Marketing | yes | Brand Tooltip matrix |
| Marketing Budget Spent Rate | Marketing | yes | Brand Tooltip matrix |
| Marketing Price Effect | Marketing | yes | Brand Tooltip matrix |
| Marketing Product Gross Margin | Marketing | yes | Brand Tooltip matrix |
| Marketing Product Gross Profit | Marketing | yes | Brand Tooltip matrix |
| Marketing Product Sales | Marketing | yes | Brand Tooltip matrix |
| Marketing Quantity Effect | Marketing | yes | Brand Tooltip matrix |
| Marketing Verdict | Marketing | yes | Brand Tooltip matrix |

### Margin

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Gross Profit Actual Year | Profit | yes | inside 1 measure (Marketing Product Gross Profit) |
| Gross Margin Actual Year | Ratios | yes | inside 1 measure (Marketing Product Gross Margin) |

### Prices

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Bonus Product Rate Actual Year | Price Calculation | yes | Calculation matrix |
| Distribution Cost Actual Year % | Price Calculation | yes | Calculation matrix |
| Distribution Profit Actual Year % | Price Calculation | yes | Calculation matrix |
| Mark Up Actual Year | Price Calculation | yes | Calculation matrix |
| Marketing Coefficient Actual Year % | Price Calculation | yes | Calculation line chart |
| Marketing Coefficient Prior Year % | Price Calculation | yes | Calculation line chart |
| Marketing Product Rate Actual Year | Price Calculation | yes | Calculation matrix |
| Position Product Rate Actual Year | Price Calculation | yes | Calculation matrix |
| VAT Rate Actual Year | Price Calculation | yes | Calculation matrix |
| Cost Price Actual Year | Product Prices | yes | Calculation line chart |
| Cost Price Prior Year | Product Prices | yes | Calculation line chart |
| Regular Price Actual Year | Product Prices | yes | Calculation line chart |
| Regular Price Prior Year | Product Prices | yes | Calculation line chart |
| Supplier Price Actual Year | Product Prices | yes | Calculation line chart |
| Supplier Price Prior Year | Product Prices | yes | Calculation line chart |

### Sales

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Sales Net Price Actual Year | Net Price | yes | Budget matrix |
| Sales Net Price Prior Year | Net Price | yes | inside 1 measure (Sales Net Price Prior Year YTD) |
| Sales Net Price Prior Year YTD | Net Price | yes | inside 2 measures (Marketing Price Effect, …) |
| Sales Net Price ex VAT Actual Year | Net Price ex VAT | yes | inside 3 measures (Marketing Budget Spent Rate, …) |
| Sales Quantity Actual Year | Quantity | yes | inside 2 measures (Marketing Price Effect, …) |
| Sales Quantity Prior Year | Quantity | yes | inside 1 measure (Sales Quantity Prior Year YTD) |
| Sales Quantity Prior Year YTD | Quantity | yes | inside 2 measures (Marketing Price Effect, …) |
| Sales Selling Price Actual Year | Selling Price | yes | inside 1 measure (Extra Discount Rate Actual Year) |

### Trade Marketing

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Bonus Accumulated | Annual Bonus | yes | Bonus bar chart, Bonus column chart, Total matrix |
| Bonus Accumulated Period | Annual Bonus | yes | inside 4 measures (Bonus Accumulated, …) |
| Bonus Accumulated Year-End | Annual Bonus | yes | inside 1 measure (Bonus Accumulated \| HTML) |
| Bonus Customers Count | Annual Bonus | yes | inside 2 measures (Bonus Customers \| HTML, …) |
| Bonus Difference | Annual Bonus | yes | Bonus matrix |
| Bonus Not Given | Annual Bonus | yes | Bonus table |
| Bonus Price Rate | Annual Bonus | yes | inside 9 measures (Bonus Accumulated \| HTML, …) |
| Bonus Rate | Annual Bonus | yes | Bonus matrix |
| Bonus Spent | Annual Bonus | yes | Bonus bar chart, Bonus column chart, Bonus matrix +1 |
| Bonus Spent Above Accumulated | Annual Bonus | yes | inside 1 measure (Bonus Risks \| HTML) |
| Bonus Spent Above Accumulated Customers Count | Annual Bonus | yes | inside 3 measures (Bonus Customers \| HTML, …) |
| Bonus Within Accumulated Customers Count | Annual Bonus | yes | inside 1 measure (Bonus Customers \| HTML) |
| Price Bonus | Annual Bonus | yes | Bonus matrix |
| Sales ex VAT | Annual Bonus | yes | Bonus matrix |
| Sales Without Bonus | Annual Bonus | yes | Bonus table |
| Credit Notes Issued Actual Year | Credit Notes Issued | yes | Budget matrix, Total matrix |
| Credit Notes Received Actual Year | Credit Notes Received | yes | Budget matrix, Total matrix |
| Extra Discount Actual Year | Discounts | yes | Budget matrix |
| Extra Discount Rate Actual Year | Discounts | yes | Budget matrix |
| Fixed Positions | Secondary Position | yes | Position bar chart, Position matrix |
| Fixed Without Price | Secondary Position | yes | Position matrix |
| Position Accumulated | Secondary Position | yes | Total matrix, Position column chart |
| Position Accumulated Period | Secondary Position | yes | inside 3 measures (Position Accumulated, …) |
| Position Accumulated Year-End | Secondary Position | yes | inside 1 measure (Position Accumulated \| HTML) |
| Position Average Monthly Price | Secondary Position | yes | inside 1 measure (Position Checks \| HTML) |
| Position Credit Notes | Secondary Position | yes | Position matrix |
| Position Customer Difference | Secondary Position | yes | Position matrix |
| Position Customer Rate | Secondary Position | yes | Position matrix |
| Position Customers Count | Secondary Position | yes | inside 1 measure (Position Register \| HTML) |
| Position Difference | Secondary Position | yes | inside 1 measure (Position Checks \| HTML) |
| Position Rate | Secondary Position | yes | inside 7 measures (Bonus Customer \| VisualSubtitle, …) |
| Position Sales | Secondary Position | yes | inside 2 measures (Position Spent Rate, …) |
| Position Sales ex VAT | Secondary Position | yes | Position matrix |
| Position Spent | Secondary Position | yes | Position matrix, Position column chart |
| Position Spent Rate | Secondary Position | yes | inside 1 measure (Position Spent Rate \| HTML) |
| Position Spent Year-End | Secondary Position | yes | inside 1 measure (Position Spent \| HTML) |
| Position Suppliers Without Credit Note Count | Secondary Position | yes | inside 1 measure (Position Checks \| HTML) |
| Positions Count | Secondary Position | yes | inside 3 measures (Fixed Positions, …) |
| Positions Percentage Count | Secondary Position | yes | Position bar chart |
| Price Position | Secondary Position | yes | Position matrix |
| Total Accumulated | Total Investment | yes | Total bar chart, Total matrix, Total column chart |
| Total Accumulated Rate | Total Investment | yes | inside 1 measure (Total Accumulated \| HTML) |
| Total Coverage | Total Investment | yes | inside 1 measure (Total Difference \| HTML) |
| Total Difference | Total Investment | yes | Total matrix |
| Total Discounts | Total Investment | yes | Total matrix |
| Total High Spent Amount | Total Investment | yes | inside 1 measure (Total Checks \| HTML) |
| Total High Spent Customers Count | Total Investment | yes | inside 1 measure (Total Checks \| HTML) |
| Total Sales | Total Investment | yes | Total matrix |
| Total Spent | Total Investment | yes | Total bar chart, Total matrix, Total column chart |
| Total Spent Rate | Total Investment | yes | Total matrix |
| Budget Accumulated Actual Year | Trade Spend | yes | Budget matrix |
| Budget Difference Actual Year | Trade Spend | yes | Budget matrix |
| Budget Spent Actual Year | Trade Spend | yes | Budget matrix |
| Campaigns | Trade Spend | yes | inside 3 measures (Total Spent, …) |
| Credit Notes Issued | Trade Spend | yes | inside 1 measure (Credit Notes Issued Actual Year) |
| Credit Notes Received | Trade Spend | yes | inside 1 measure (Credit Notes Received Actual Year) |
| Extra Discount | Trade Spend | yes | inside 1 measure (Extra Discount Actual Year) |
| Extra Discount Without VAT | Trade Spend | yes | inside 2 measures (Budget Spent Actual Year, …) |
| Marketing Accumulated | Trade Spend | yes | inside 1 measure (Marketing Accumulated Actual Year) |
| Marketing Accumulated Actual Year | Trade Spend | yes | Budget matrix, Total matrix |
| Promotions | Trade Spend | yes | inside 3 measures (Total Spent, …) |
| Seasonal Fixed Positions | Trade Spend | yes | inside 1 measure (Trade Terms Spent) |
| Service Invoices | Trade Spend | yes | inside 1 measure (Service Invoices Actual Year) |
| Service Invoices Actual Year | Trade Spend | yes | inside 3 measures (Budget Accumulated Actual Year, …) |
| Trade Terms Spent | Trade Spend | yes | inside 1 measure (Trade Terms Spent Actual Year) |
| Trade Terms Spent Actual Year | Trade Spend | yes | Total matrix |
<!-- register:end -->
