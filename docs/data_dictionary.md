# Data dictionary

Table: `OrderRecon` (one row per order, 99,441 rows). Money is in BRL, not converted.
Source files: Olist Brazilian E-Commerce Public Dataset (Kaggle, CC BY-NC-SA 4.0).

| Column | Meaning | How it was made | Check |
|---|---|---|---|
| order_id | Unique order ID | From orders file | 99,441 unique, matches the file |
| order_status | Delivered, canceled, shipped, etc. | From orders file | Used to decide Late |
| order_purchase_timestamp | When the customer bought | From orders file | Links to the date table in Power BI |
| order_delivered_customer_date | When the customer received it | From orders file | Blank if not delivered |
| order_estimated_delivery_date | Date promised to the customer | From orders file | Compared with the delivered date |
| ItemRows | Number of item rows on the order | Count of order_items grouped by order | Blank means no items: flag "Missing items" |
| ItemsPrice | Sum of item prices | Sum of price, grouped by order | 112,650 item rows became 98,666 orders |
| FreightTotal | Sum of shipping charges | Sum of freight, grouped by order | Same grouping as above |
| PaymentRows | Number of payment rows on the order | Count of order_payments grouped by order | Blank means no payment: flag "Missing payment" |
| PaymentTotal | Total paid | Sum of payment value, grouped by order | 103,886 payment rows became 99,440 orders |
| ItemsTotal | What the order should cost | ItemsPrice + FreightTotal | |
| Diff | Paid minus should-cost | PaymentTotal - ItemsTotal | Positive = overpaid, negative = underpaid |
| IsLate | 1 late, 0 on time, blank not delivered | Delivered order with delivery date after estimated date | Late % = 8.11% |
| ReviewScore | Latest review score, 1 to 5 | XLOOKUP on the reviews table | Blank = no review (768), not zero |
| State | Customer's state | XLOOKUP on the customers table | Customer state, not seller state |
| Category | English category of the first item | Lookup through items, products and translation | "unknown" and "no items" are labels, not real categories |
| Flag | Match, Small gap, Large gap, Missing items, Missing payment | Rules on Diff and row counts | Counts add up to 99,441 |
| IsMismatch | 1 for Small gap or Large gap, else 0 | From Flag | 303 orders |

Notes
- Column names above match the formulas used; verify them against your workbook before publishing.
- Match tolerance is about 1.5 cents because Diff is rounded to 2 decimals first.
- Small gap is up to 1 BRL. Large gap is over 1 BRL.
