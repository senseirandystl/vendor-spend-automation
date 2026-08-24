# Power Query walkthrough

Rebuild `output/AP_Spend_Report.xlsx` from the files in `data/`.

CSV and Excel both work. If you use the CSVs in this repo, choose **From Folder** and combine the files in `data/raw/`.

## 1. Combine monthly AP files

1. New workbook → save as `AP_Spend_Report.xlsx`.
2. **Data → Get Data → From File → From Folder** → `data/raw`.
3. **Combine & Transform Data**.
4. In Power Query:
   - Set `Invoice Date` and `Due Date` to **Date**.
   - Set `Amount` to **Decimal Number**.
   - Set `Invoice No.` to **Text**.
   - Filter out any leftover header rows (`Company` should not equal `"Company"`).
5. Rename the query **AP_Monthly**.
6. **Close & Load → Only Create Connection**.

## 2. Load lookups (connection only)

**Department_Heads**

- **Data → Get Data → From File → From Text/CSV** → `data/lookups/Department_Heads.csv`
- Types: Text
- Connection only

**Exchange_Rates**

- Import `data/lookups/Exchange_Rates.csv`
- Set `Date` to Date and the four rate columns to Decimal Number
- Select EUR, CAD, GBP, USD → right-click → **Unpivot Columns**
- Rename `Attribute` → `Currency`, `Value` → `Exchange Rate`
- Connection only

## 3. Merge department onto invoices

1. Open **AP_Monthly**.
2. **Home → Merge Queries → Merge Queries as New**.
3. Join `Company` to `Department_Heads[Company]`, Left Outer.
4. Expand `Department` and `Department_Head`.
5. Rename **AP_Enriched**. Connection only.

## 4. Merge FX on date + currency

1. Merge AP_Enriched to Exchange_Rates.
2. Hold Ctrl and select two columns on each side, same order:
   - AP: `Invoice Date`, `Currency`
   - Rates: `Date`, `Currency`
3. Expand `Exchange Rate`.
4. **Add Column → Custom Column**
   - Name: `USD Value`
   - Formula: `[Amount] * [Exchange Rate]`
5. Type: Decimal Number.

## 5. Overdue Flag and Aging Bucket

Report date is fixed at 2024-12-31 so the file refreshes the same way every time.

**Days Overdue**

```powerquery
Duration.Days(#date(2024, 12, 31) - [Due Date])
```

**Overdue Flag**

```powerquery
if [Due Date] < #date(2024, 12, 31) then "Yes" else "No"
```

**Aging Bucket**

```powerquery
let
    d = Duration.Days(#date(2024, 12, 31) - [Due Date])
in
    if d <= 0 then "Current"
    else if d <= 30 then "1-30"
    else if d <= 60 then "31-60"
    else if d <= 90 then "61-90"
    else "90+"
```

Rename the query **AP_Final**. **Close & Load To → Table** (and/or Pivot Table Report).

## 6. Leadership pivot

Rows:

- Company
- Invoice No.
- Invoice Date
- Due Date
- Aging Bucket

Values: Sum of USD Value  
Filters: Department_Head  
Slicer: Department

Then:

1. Right-click any auto-grouped month → **Ungroup**.
2. **Design → Report Layout → Show in Tabular Form**.
3. **Design → Report Layout → Repeat All Item Labels**.
4. **Design → Subtotals → Do Not Show Subtotals**.
5. Turn off +/- buttons.

Keep **Company** next to Invoice No. Invoice numbers are reused across vendors. See [`data-notes.md`](data-notes.md).

## 7. Refresh test

Copy `AP_2024-12.csv` to `AP_2025-01.csv`, drop it in `data/raw`, **Data → Refresh All**. New rows should appear without changing the queries.
