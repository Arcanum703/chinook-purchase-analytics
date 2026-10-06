# Customer Purchase Analytics

A SQL analysis of customer re-engagement patterns in the Chinook database — identifying the time gap between each customer's first and second purchase to support data-driven retention decisions.

Built as part of the **Database Systems** course at Corvinus University of Budapest.

---

## Objective

For each customer who made at least two purchases, calculate the number of days between their first and second invoice. This helps:

- Understand how quickly customers return after their first purchase
- Identify at-risk customers with unusually long gaps
- Inform follow-up marketing and re-engagement campaigns

## Approach

The query uses a single CTE (`InvoiceSequence`) combining two window functions:

- **`LAG()`** — retrieves the previous invoice date for each customer, enabling pairwise date comparison
- **`ROW_NUMBER()`** — assigns a sequential number to each invoice per customer, allowing us to isolate the second purchase specifically

The CTE is then joined with the `Customer` table, filtered to `InvoiceOrder = 2`, and the day gap is computed with `DATEDIFF()`.

## Tables Used

| Table | Purpose |
|-------|---------|
| `Invoice` | Invoice dates, linked to customers |
| `Customer` | Customer names and IDs |

## Final Query

```sql
WITH InvoiceSequence AS (
    SELECT
        CustomerId,
        InvoiceDate,
        LAG(InvoiceDate) OVER (
            PARTITION BY CustomerId
            ORDER BY InvoiceDate
        ) AS PreviousInvoiceDate,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerId
            ORDER BY InvoiceDate
        ) AS InvoiceOrder
    FROM Invoice
)
SELECT
    C.CustomerId,
    ISNULL(C.FirstName + ' ', '') + ISNULL(C.LastName, '') AS CustomerName,
    I.InvoiceDate AS SecondInvoiceDate,
    I.PreviousInvoiceDate AS FirstInvoiceDate,
    DATEDIFF(day, I.PreviousInvoiceDate, I.InvoiceDate) AS DaysBetweenFirstAndSecond
FROM InvoiceSequence I
JOIN Customer C ON I.CustomerId = C.CustomerId
WHERE I.InvoiceOrder = 2
ORDER BY DaysBetweenFirstAndSecond;
```

## Key Concepts Demonstrated

- Common Table Expressions (CTEs)
- Window functions — `LAG()` and `ROW_NUMBER()`
- `PARTITION BY` for per-customer sequencing
- `DATEDIFF()` for date arithmetic
- `ISNULL()` for NULL-safe string concatenation
- Filtering derived results with `WHERE` on window function output

## Database

This project uses the [Chinook database](https://github.com/lerocha/chinook-database), a sample dataset representing a digital media store with customers, invoices, tracks, and more.

## Tech Stack

- **SQL Server** (T-SQL syntax)
- **Chinook sample database**

## License

MIT

---

*Built by [Arcanum241](https://github.com/Arcanum703)*
