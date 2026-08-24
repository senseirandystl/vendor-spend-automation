# Automated Multi-Currency Vendor Spend Reporting

Power Query project that turns monthly Accounts Payable exports into a refreshable leadership report.

**Problem:** Finance received a separate Excel/CSV AP export every month. Files used mixed currencies, had no department owner, and were combined with copy/paste plus VLOOKUPs. A new month meant repeating the whole pack.

**Solution:** One Power Query model that:
1. Combines every monthly AP file in `data/raw/`
2. Maps each vendor to a department and department head
3. Converts invoice amounts to USD using the exchange rate on the invoice date
4. Flags overdue invoices and aging buckets
5. Loads a tabular pivot for recurring and ad-hoc reporting

Drop a new month file into `data/raw/` and click **Data → Refresh All**.

## Demo the report

Rebuild it in Excel by following [`docs/power-query-walkthrough.md`](docs/power-query-walkthrough.md).

Source files are stored as CSV so the repo stays reviewable on GitHub. Power Query can load CSV or Excel from a folder the same way.

```
vendor-spend-automation/
├── README.md
├── docs/
│   ├── power-query-walkthrough.md
│   └── data-notes.md
├── data/
│   ├── raw/                 # monthly AP extracts — add new months here
│   └── lookups/
│       ├── Department_Heads.csv
│       └── Exchange_Rates.csv
└── output/
    └── AP_Final.csv         # expected fact table after refresh
```

## What the model produces

| Column | Description |
|--------|-------------|
| Company | Vendor |
| Invoice No. | Vendor invoice number (not globally unique — see data notes) |
| Invoice Date / Due Date | Dates from the extract |
| Department / Department_Head | From the vendor lookup |
| Currency / Amount | Original billed amount |
| USD Value | Amount × invoice-date FX rate |
| Days Overdue | Days past due as of 2024-12-31 |
| Overdue Flag | Yes / No |
| Aging Bucket | Current, 1-30, 31-60, 61-90, 90+ |

Leadership views from the same table: spend by owner, spend by vendor, overdue risk, monthly trend, ad-hoc slices.

## Skills demonstrated

- Power Query folder combine (new files auto-ingest)
- Merge queries (vendor → owner)
- Unpivot + composite-key merge (date + currency → FX rate)
- Custom columns for USD, aging, and overdue flags
- Connection-only staging queries vs one loaded fact table
- Tabular pivot + slicers for leadership reporting
- Data-quality check: invoice numbers are not a unique key

## Data notes

Sample data is synthetic and based on a public Power Query training pattern. Invoice numbers can repeat across vendors and months. The grain of the fact table is **Company + Invoice No. + Invoice Date**, not Invoice No. alone. Details in [`docs/data-notes.md`](docs/data-notes.md).

---

**About Me**  
Randall James | Data Coordinator / Data Analyst / Project Manager  
St. Louis, MO (O'Fallon area) | Open to remote, hybrid, or on-site within ~30 min commute  
[LinkedIn](https://www.linkedin.com/in/randall-james-stl) | [GitHub](https://github.com/senseirandystl) | randalljames34@pm.me

*This project was created as part of my professional portfolio to demonstrate data analysis capabilities.*
