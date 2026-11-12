# Marketplace Leak Audit

An Excel and Power BI audit of 99,441 orders from a Brazilian online marketplace (Olist). It asks one business question: where does the marketplace lose money and customer trust?

This is a portfolio project built on a public dataset. It is not company data and it is not work experience. All money values are in Brazilian real (BRL) and are not converted.

Author: Sreenidhi reddy
Contact: sreenidhikreddy@gmail.com

---

## Top findings

| # | Finding | Number |
|---|---|---|
| 1 | Delivered orders that arrived after the promised date | 8.11% |
| 2 | Average review score, late orders vs on-time orders | 2.57 vs 4.29 (gap of about 1.7 points) |
| 3 | Highest late share by customer state | AL 23.93%, MA 19.67%, PI 15.97% |
| 4 | Lowest late share among large states | SP 5.89% |
| 5 | Orders where amount paid does not match items plus shipping | 303 orders (0.3%), 3,269.22 BRL in total gaps |
| 6 | Orders with a payment recorded but no item recorded | 775 orders, 162,591.95 BRL |

What these mean:

- Late delivery is where review scores collapse, and it is concentrated in a few northern and northeastern states, not spread evenly.
- Payment mismatches are small in money terms. The larger issue is a data gap: 775 orders have money recorded against them but no item rows. That is about 98% of the money sitting in non-matching orders (162,591.95 of about 166,005 BRL), and it looks like a data-quality problem rather than a payment error. I did not investigate the cause.

Late orders and low scores go together. This project does not prove that lateness caused the low scores.

---

## Business questions

1. **Delivery.** What share of delivered orders arrive late, and how much lower are review scores for late orders?
2. **Money match.** For each order, does the total paid match item price plus shipping? How many do not, and by how much in total?
3. **Where it concentrates.** Which product categories and states carry the most revenue, and which carry the most late orders?
4. **Lost orders.** What share of orders are canceled or unavailable, and is it higher in some places?

Question 4 is only partly answered. The Flag column and the "missing items" group show orders that never reached an item record, and their statuses include canceled, created, invoiced, shipped and unavailable. A full breakdown of lost orders by place was not built in v1.

---

## Data

- Source: Brazilian E-Commerce Public Dataset by Olist, Kaggle. Real, anonymized orders from 2016 to 2018.
- License: CC BY-NC-SA 4.0 (https://creativecommons.org/licenses/by-nc-sa/4.0/). Non-commercial use with credit and the same license on shared adaptations.
- The raw data is not included in this repository. Download it from the Kaggle page to reproduce the work.

Files used (7 of the 9 in the dataset):

| File | Rows |
|---|---|
| orders | 99,441 |
| order_items | 112,650 |
| order_payments | 103,886 |
| order_reviews | 99,224 |
| customers | 99,441 |
| products | 32,951 |
| product category translation | 71 |

Not used on purpose: geolocation and sellers (outside the scope of the four questions).

---

## Method

### Excel (Power Query, formulas, pivots)

1. Loaded orders, items and payments in Power Query.
2. Grouped items by order and summed price and freight. Grouped payments by order and summed payment value. Grouping came before merging, because merging first repeats each order once per item and inflates every total.
3. Merged both groups onto orders with a left outer join, keeping every order. The result is the table `OrderRecon`.
4. Checked the result: 99,441 rows, 99,441 unique order IDs, matching the orders file.
5. Added formula columns: items total (price plus freight), difference (paid minus items total), late flag (delivered orders only), review score and state (XLOOKUP), product category, and a mismatch indicator.
6. Added a Flag column that labels each order: Match, Small gap, Large gap, Missing items or Missing payment. Large gaps are red, small gaps are amber, missing data is grey.
7. Built two pivot tables with slicers (late share and review by state; revenue, late share and mismatches by category).
8. Built a Brief sheet with six KPIs, a state dropdown that drives them, one chart and a short summary.

### Power BI

- Loaded `OrderRecon` as the main table with a date table linked on purchase date (one-to-many, single direction).
- Wrote 7 DAX measures: Orders, Revenue, Late %, Avg Review, Avg Review (Late), Mismatch Orders, Mismatch Amount.
- Built 3 pages: Overview, Delivery and reviews, Reconciliation.

### Cross-tool check

Key totals match between Excel and Power BI.

| Measure | Excel | Power BI |
|---|---|---|
| Orders | 99,441 | 99,441 |
| Late % | 8.11% | 8.11% |
| Mismatch Amount (BRL) | 3,269.22 | 3,269.22 |

Revenue (items only, BRL) is 13,591,643.70 in Excel and 13.59M in Power BI.

---

## Definitions and decisions

- **Late:** a delivered order whose delivery date is after the estimated delivery date. Orders that were never delivered are left blank and are not counted as on time or late. Late % is late orders divided by delivered orders.
- **Match tolerance:** differences of about 1 cent or less count as a match. The Flag formula rounds the difference to 2 decimals first, so the effective tolerance is about 1.5 cents. I chose a cent because rounding is fractions of a cent, and 138 orders sat between 0.01 and 1 BRL, which is too many to dismiss as rounding. A small gap is up to 1 BRL, and a large gap is over 1 BRL.
- **Mismatch amount:** the sum of absolute differences for small and large gaps, so overpayments and underpayments do not cancel out.
- **Missing items:** orders with no item rows. They are kept out of the money-match count so their full payment does not appear as a fake mismatch.
- **Reviews:** the reviews file has 99,224 rows but only 98,673 unique orders. I kept the latest review per order. Orders with no review (768) stay blank and are not treated as zero.
- **State:** the customer's state, not the seller's.
- **Category:** each order gets the category of its first item, in English. I chose this to keep one main table and keep Excel and Power BI comparable. Orders with no category are labeled unknown, and orders with no items are labeled no items.

---

## Reconciliation result

| Flag | Orders | Net difference (BRL) |
|---|---|---|
| Match | 98,362 | -0.67 |
| Small gap | 54 | 3.87 |
| Large gap | 249 | 2,867.19 |
| Missing items | 775 | 162,591.95 |
| Missing payment | 1 | -143.46 |
| Total | 99,441 | |

Row counts at each stage: 112,650 item rows grouped to 98,666 orders; 103,886 payment rows grouped to 99,440 orders; 99,441 orders in the final table.

---

## Limits

- Public data from 2016 to 2018. Results do not describe any company today.
- Late and low scores are associated, not shown to be causal.
- Vouchers and installments may explain some payment gaps. I did not test this, and a payment-type breakdown was not built.
- Category revenue uses the first item only. Orders with more than one item can contain products from different categories, and only the first item's category is counted.
- Small states (for example RR with 46 orders, AC with 81) and small categories swing a lot on a few orders.
- If two reviews for one order share the same timestamp, one is chosen arbitrarily.
- The last two months in the data are near zero because the dataset ends in 2018, not because orders collapsed. I did not verify the exact end date.
- Lookup ranges in some Excel formulas are fixed to the current row counts.

---

## Recommendation

- Logistics: set longer promised delivery dates for AL, MA and PI, and review carrier performance there. A side effect is that longer promises may reduce conversion.
- Data team: find out why 775 orders have payments but no item records.
- Next test: compare actual and promised delivery days by state to see where the promise is least realistic.

---

## Repository contents

```
marketplace-leak-audit/
  README.md
  memo/
    memo.pdf
  report/
    MarketplaceLeakAudit_report.pdf
  screenshots/
      overview.png
      excel_brief.png
  docs/
    data_dictionary.md
```

Not included, because of the dataset license: the raw CSV files, the Excel workbook and the Power BI file.

## How to reproduce

1. Download the dataset from the Olist page on Kaggle and put the CSVs in a folder.
2. In Excel, load orders, items and payments with Power Query. Group items and payments by order ID, then merge onto orders with a left outer join.
3. Check that the result has 99,441 rows and no duplicate order IDs.
4. Add the formula columns and the Flag column as described in the Method and Definitions sections.
5. Load the Excel tables into Power BI, link a date table to purchase date, and add the 7 measures.
6. Confirm Orders, Late % and Mismatch Amount match between the two tools.

## Tools

Excel (Microsoft 365: Power Query, XLOOKUP, COUNTIFS, SUMPRODUCT, pivot tables, slicers, conditional formatting, data validation) and Power BI Desktop (data model, DAX).

## Credit

Data: Brazilian E-Commerce Public Dataset by Olist, Kaggle, licensed CC BY-NC-SA 4.0 (https://creativecommons.org/licenses/by-nc-sa/4.0/). Modified: aggregated, joined and analyzed by Sreenidhi reddy. Not endorsed by Olist. This repository's write-up and screenshots are shared under the same license.
