# Trade investment

[← README](../README.md)

## Words and rates

**One word per concept.** Every title, card, caption and alt text uses three words, and
measure names follow them:

| Word | Means |
|---|---|
| **Accumulated** | what the selling price sets aside, and what suppliers give back |
| **Spent** | what was paid or given: bonus, positions, campaigns, promotions, credit notes issued, discounts |
| **Difference** | Accumulated − Spent; below zero, more was spent than set aside |

No synonym (reserved, accrued, cost, surplus, budget left) is used beside them. A measure is
named `<Pot> Accumulated`, `<Pot> Spent`, `<Pot> Difference`, `<Pot> Spent Rate` (spent as a
share of sales): `Position Accumulated`, `Bonus Spent`, `Budget Difference Actual Year`.

**The rates live in one place.** Each brand carries four rates of its net price without VAT,
read from the source by the `DimBrand` query: `DistributionCostRate`, `DistributionProfitRate`,
`AnnualBonusRate` and `SecondaryPositionRate`. No measure, label or document repeats a number:
the subtitles that quote a rate carry a `{BonusRate}` or `{PositionRate}` placeholder, and their
label measures fill in `Bonus Price Rate` and `Position Rate`. Both read the highest rate across
brands, so a card that quotes one rate assumes one rate for every brand; brands with their own
rate show it on the `Calculation` page.

**Everything here is without VAT** — the rates, the trade terms, the credit notes, the service
invoices and the discount — so every pot adds up on one basis
([measures.md](measures.md#two-sales-bases)).

## The price calculation

The `Calculation` page builds the selling price up from cost, product by product, and shows
what is left for marketing.

```
Marketing Coefficient = Gross Price ex VAT − Cost Price
                        − ( Distribution Cost + Distribution Profit + Bonus Per Unit + Position Per Unit )
Marketing Accumulated = sum over sales lines: Quantity × the line's coefficient
```

- **`Gross Price ex VAT`** is the price after the regular discount and before the extra one;
  `Cost Price` is the average cost. Both are per unit, weighted by quantity.
- **The four per-unit rates** (`Distribution Cost`, `Distribution Profit`, `Bonus Per Unit`,
  `Position Per Unit`) are each brand's rate on its `Sales Net Price ex VAT`, divided by the
  quantity. They iterate `DimBrand`: the rate is a brand attribute, so net sales per brand × rate
  equals the sum over the lines, without evaluating a measure per line.
- **Every part is weighted by quantity**, so the columns of the price table add up, and
  `Marketing Coefficient` × quantity gives `Marketing Accumulated` exactly.
- **A negative coefficient is a real result**: the price does not cover cost and the four rates.
  It stays in the totals, never hidden as an error. Distribution profit is the company's own
  target margin, so a negative coefficient means the sale missed the planned price structure —
  not that it lost money.
- **The coefficient starts from the price after the regular discount**, while the rates are
  charged on the net price, the money the company takes. The extra discount is therefore not
  taken out of the coefficient: it is counted as spend, in the budget
  ([Marketing budget](#marketing-budget)).
- **Margin and budget are kept apart.** No margin measure takes budget or spend; a margin after
  spend is not built ([decisions.md](decisions.md)).
- The per-product rates on the table (`Distribution Cost %`, `Bonus Product Rate`, …) read the
  product's brand, and are blank unless one product is in context.
- The page's charts set this year against last year week by week: supplier price, cost price,
  marketing coefficient and regular price. `Supplier Price` comes from the purchase lines
  (`FactPurchaseLine`), which carry no customer or outlet, so the customer and channel slicers
  leave it unchanged.

## The trade terms

The terms agreed with customers — bonuses, placements, campaigns, promotions — come from one
register, read once by the shared expression `TradeTerms` and loaded by two queries.

| Term group | Billed | Spent counts |
|---|---|---|
| **Bonus** | a percentage of the outlet's sales of the brand, or a fixed value per month | in the month it arises, the current month included |
| **Position** (every placement type) | the same | in the month it arises, the current month included |
| **Campaign** | a fixed amount | the amount agreed, dated at its start |
| **Promotion** | a fixed amount | the same |

The group comes from the source (`ti.TermType`), not from a list of codes in the query. The
source holds **one row per term**: an outlet, a brand, a term type, a billing method and a
period. `TradeTerms` reads them as they are and adds what the measures need: the window, the
months each term has run, and whether it covers a whole calendar year.

**Each placement type is its own term.** A secondary position, a shelf and a basket for the
same outlet and brand each charge their own rate on the brand's sales: the term type is part of
what makes a term one term.

**One threshold is a rule, not data.** The 10 % of sales that marks a high-spend customer on
`Total` is fixed in the model. It is a decision about how to read the data, not a rate of the
price, so it does not come from the source ([decisions.md](decisions.md)).

Two tables hold the result:

- **`FactTradeTerm`**: one row per term, dated at its start, with what it has cost up to today
  (`Spent`) and, for a fixed term, its monthly value. The counts, the campaigns and the
  promotions read it.
- **`FactTradeTermMonth`**: for every bonus and position term, one row per month it runs,
  with that month's cost — a percentage term's rate on the outlet's sales of the brand in that
  month, a fixed term's value × units in every month, the months still to come included. The
  bonus and position spent read it, so an annual term shows in every month, not all in January.

A term carries an outlet and a brand, never a product: the product slicer does not move it.

## Annual bonus

The `Bonus` page sets what the price **accumulates** for the annual bonus against what the
customers' bonus terms **cost**, this year to date.

```
Bonus Accumulated = sum over brands: AnnualBonusRate × Sales Net Price ex VAT, this year to date
Bonus Spent       = the bonus terms' cost in the months of this year to date
Bonus Difference  = Price Bonus − Bonus Spent        (per customer with a bonus term)
```

- **A bonus customer** has a bonus term running at some point between 1 January and today, by
  the term's own dates (`TermCustomers`).
- **What a customer accumulated is its own.** `Price Bonus` is the bonus rate on the sales of
  the bonus customers; `Bonus Difference` sets it against their bonus spent. Below zero, the
  customer's terms cost more than its sales set aside (`Bonus Spent Above Accumulated`).
- **The spent rate** (`Bonus Rate`) is the bonus spent over the sales of the customers it is
  agreed with (`Sales ex VAT`), so it reads directly against the rate in the price.
- **Accumulated without a bonus** (`Bonus Not Given`) is the bonus rate on the sales of the
  customers with no bonus term: set aside in the price and paid to nobody.
- **`Bonus Accumulated Year-End`** projects this year's accumulated to 31 December with last
  year's seasonality: so far × last year's full year ÷ last year to the same day.

**The cards** read left to right from what the price holds to what is paid:

| Card | Shows |
|---|---|
| Bonus accumulated | `Bonus Accumulated`, split into customers with and without a bonus term, and the projection to year end |
| Spent against accumulated | `Bonus Spent` ÷ `Bonus Accumulated`; the bar sets the spent rate on sales against the rate in the price, with the difference between the two rates |
| Bonus spent | `Bonus Spent` so far this year |
| Customers with a bonus | the bonus customers, spent above and within what their own sales accumulated, and their spent rate against the rate in the price |

Beside the monthly chart, the **Risks** card lists the customers spent above their
accumulation and by how much, the spent rate, and the bonus accumulated on customers without a
bonus, as a value and as a share. Below, one table lists the bonus customers (sales, spent rate,
bonus spent, accumulated, difference) and another the customers without one (`Sales Without
Bonus`, `Bonus Not Given`).

## Secondary positions

The `Position` page does the same for in-store placements.

```
Position Accumulated = sum over brands: SecondaryPositionRate × Sales Net Price ex VAT, this year to date
Position Spent       = the position terms' cost in the months of this year to date, the current month included
Position Difference  = Position Accumulated − Position Spent
```

- **`Position Spent Year-End`** takes every month of this year: the fixed positions for the
  months still to come, the percentage positions on the sales so far. It is a commitment from
  the register, not a forecast of payments.
- **Without a value**: a fixed position with no value in the register counts as a position
  with no cost (`Fixed Without Price`); the average monthly value is taken over the others.
- **Credit notes are the suppliers' budget, not position cost.** The page sets the credit
  notes received on a brand against its position cost as a check, per brand only
  (`Position Suppliers Without Credit Note Count`).
- **A customer with a position** has one running at some point between 1 January and today,
  whenever it started (`TermCustomers`, by the term's own dates), as for the bonus: a position
  that started last year and runs into this one counts, as its cost does.
- **Counts are terms**: `Positions Count`, `Fixed Positions` and `Positions Percentage Count`
  count the rows of `FactTradeTerm`, one per term.
- **`Position Accumulated Year-End`** works like the bonus one: last year's seasonality.

**The cards**, the same layout as `Bonus`:

| Card | Shows |
|---|---|
| Position accumulated | `Position Accumulated`, split into customers with and without a position, and the projection to year end |
| Spent against accumulated | `Position Spent` ÷ `Position Accumulated`; the bar sets the spent rate on sales against the position rate |
| Position spent | `Position Spent` as a bar of `Position Spent Year-End`: so far and still to come, the total for the year in the note |
| Positions in outlets | every position once: fixed with a value, fixed without one, percentage of sales; and the customers with a position |

Beside the monthly chart, the **Checks** card lists the brands with position cost and no credit
note, the credit notes received, the fixed positions without a value, the difference and the
average monthly value.

## Marketing budget

The `Budget` page sets what is **accumulated** for marketing against what is **spent**.

```
Budget Accumulated = Marketing Accumulated + Credit Notes Received + Service Invoices
Budget Spent       = Trade Terms Spent + Extra Discount Without VAT + Credit Notes Issued
Trade Terms Spent  = Campaigns + Promotions + Seasonal Fixed Positions
Budget Difference  = Budget Accumulated − Budget Spent
```

- **`Credit Notes Received`**: credit notes from suppliers, booked on a brand, not on products,
  so it is blank on a single product.
- **`Service Invoices`**: the service and rent invoices the company sends its suppliers, booked
  to the supplier's brand (`FactServiceInvoice`).
- **`Credit Notes Issued`**: credit notes to customers. Consideration payable to a customer
  reduces revenue unless it pays for a distinct good or service
  [[ifrs-15]](sources.md#ifrs-15); the report shows sales before them, and the credit notes as
  spend.
- **`Extra Discount Without VAT`**: the gap between the price after the regular discount and
  the net price, on every sales line.
- **`Seasonal Fixed Positions`**: the fixed positions that do not run a whole calendar year,
  in each month they have run up to today. The whole-year positions stay out of the budget;
  they are in `Position Spent`.

The brand tooltip (`Brand Tooltip`) shows, for the brand or product under the pointer, its
sales, margin, budget and the price and volume effects behind its change.

### Price × volume

The tooltip splits the year-on-year change in sales into what came from selling more or fewer
units and what came from a higher or lower price:

```
Sales  = Quantity × Price
Change = P0 × (Q1 − Q0)  +  Q1 × (P1 − P0)
         Marketing Quantity Effect   Marketing Price Effect
```

- **Price is net sales over quantity**, both this year to date and last year to the same day,
  from the same measures the pages use.
- **The two effects add up to the change exactly.** Each factor is held at the opposite period
  from the one that moves; held at the same one, a remainder would belong to neither.
- **`Marketing Verdict`** names the pattern by the signs of the two effects: growth from price
  and volume, growth from price only, higher volume at a lower price, or a decline in both. A
  zero effect takes the other's sign; both zero is no change, and the verdict is blank.
- The effects are shown in millions.

## Total trade investment

The `Total` page puts every pot side by side for the current year to date.

```
Total Accumulated = Bonus Accumulated + Position Accumulated + Marketing Accumulated
                    + Credit Notes Received + Service Invoices
Total Spent       = Bonus Spent + Position Spent + Campaigns + Promotions
                    + Credit Notes Issued + Extra Discount Without VAT
Total Difference  = Total Accumulated − Total Spent
```

- **Every position counts once.** The seasonal fixed positions are inside `Position Spent`, so
  `Total Spent` adds the campaigns and the promotions, not `Trade Terms Spent`.
- **The marketing budget stays in Accumulated when negative**: a negative value means the price
  does not cover cost and the rates.
- **`Total Coverage`** is how much of the spending the money set aside covers.

**The cards**:

| Card | Shows |
|---|---|
| Accumulated | `Total Accumulated` and its parts — bonus, positions, marketing, and what suppliers give back (credit notes received and service invoices) — with its share of sales |
| Spent | `Total Spent` and its parts — bonus, credit notes issued, discounts, and positions with the other trade terms — with its share of sales |
| Difference | `Total Difference`, with a bar of `Total Coverage` |

Beside the monthly chart, the **Checks** card shows the marketing budget the price leaves, the
customers above 10 % of their sales and what they cost, the position cost, and the bonus spent as
a share of the bonus accumulated.
- Per customer the page reads spent as a share of sales (`Total Spent Rate`); the checks card
  counts the customers above 10 %.

## What the slicers reach

The channel, region and customer slicers reach a figure only through an outlet or a customer.

| Follows a channel, region or outlet pick | Shows the company figure under such a pick |
|---|---|
| everything on sales, and every trade term: `Bonus Accumulated`, `Position Accumulated`, `Marketing Accumulated`, `Bonus Spent`, `Position Spent`, `Campaigns`, `Promotions`, `Seasonal Fixed Positions`, `Extra Discount Without VAT` | `Credit Notes Issued`, `Credit Notes Received`, `Service Invoices`, `Supplier Price` |

The credit notes and the service invoices carry a customer or supplier and a brand, but no
outlet. A customer pick reaches them; a channel, region or outlet pick does not. So under such
a pick `Total Spent`, `Total Difference`, `Budget Spent Actual Year` and
`Budget Difference Actual Year` add an outlet group's figures to company figures: read them
with no such pick.
