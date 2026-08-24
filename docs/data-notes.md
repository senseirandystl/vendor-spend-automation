# Data notes

## Invoice numbers are not unique

The Power Query model did **not** assign one invoice to two departments. The source extracts reuse invoice numbers across vendors and months.

That is a data-quality issue in the raw files, not a merge error. Department comes only from **Company** via `Department_Heads`.

Examples:

| Invoice No. | Company | Department | Invoice Date |
|-------------|---------|------------|--------------|
| 11778 | Stripe | Finance | 2024-11-19 |
| 11778 | Apple | IT | 2024-12-02 |
| 11778 | Intuit | Finance | 2024-12-21 |
| 11778 | Xero | Finance | 2024-12-08 |
| 11751 | Intuit | Finance | 2024-12-03 |
| 11751 | Intuit | Finance | 2024-12-10 |
| 11751 | Airbase | Finance | 2024-12-15 |
| 11574 | Microsoft | IT | 2024-09-09 |
| 11574 | Apple | IT | 2024-09-27 |
| 11604 | American Express | Finance | 2024-10-11 |
| 11604 | Sage | Finance | 2024-10-26 |

Correct grain for an invoice row:

`Company + Invoice No. + Invoice Date`

Always keep **Company** on the pivot rows with Invoice No. If you put Invoice No. alone, the same number will appear under more than one vendor or department.

In a real AP system this would be flagged as a duplicate-key risk. In this portfolio dataset it is leftover from synthetic invoice numbering.

## FX conversion

USD Value = Amount × rate on **Invoice Date** for that currency.

Check: Stripe 11563 on 2024-09-02, EUR 30,000 × 1.106623 = **33,198.69 USD**.

## Aging as-of date

Days Overdue, Overdue Flag, and Aging Bucket use a fixed report date of **2024-12-31** so refresh results stay reproducible.
