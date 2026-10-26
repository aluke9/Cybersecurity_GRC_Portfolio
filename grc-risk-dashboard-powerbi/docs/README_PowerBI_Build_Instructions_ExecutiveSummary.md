# Cyber Risk Dashboard: Power BI Build Instructions

Step-by-step instructions to rebuild the Power BI model from `Cyber_Risk_PowerBI_Dataset.xlsx`.

**Snapshot date for all status fields:** October 20, 2026
**Canvas assumed:** 16:9, 1280 x 720

---

## Contents

1. [Before you start](#1-before-you-start)
2. [Import the data](#2-import-the-data)
3. [Transform the data (Power Query)](#3-transform-the-data-power-query)
4. [Create the Dim_Date table](#4-create-the-dim_date-table)
5. [Model view: table relationships](#5-model-view-table-relationships)
6. [Measures](#6-measures)
7. [Verification values](#7-verification-values)

---

## 1. Before you start

- Save the Excel file to a permanent folder (for example `Documents\PowerBI\`). Do not open it directly from the browser's download bar: Excel opens a temporary copy (shown with a number such as `[18]` in the name) and Power BI refreshes will break later.
- Close the Excel file before refreshing Power BI. Power BI cannot read a file Excel has locked.
- Use a recent version of Power BI Desktop (2024 or later) so DAX query view and the `INFO.VIEW.MEASURES()` function are available.

## 2. Import the data

1. **Home > Get data > Excel workbook**, select the file.
2. In the Navigator, each data sheet appears twice: once as a **sheet** and once as an **Excel table** with the same name. Power BI adds a number to one of them (for example `Fact_Control_Test4`). They contain the same data.
3. Select **one version of each** of these 7 items (the table-icon versions are preferred):
   - Dim_Asset
   - Dim_Control
   - Fact_Asset_Control
   - Fact_Control_Test
   - Fact_Review_Schedule
   - Fact_Exceptions
   - Fact_Vulnerabilities
4. Do **not** select README, KRI_Definitions, DAX_Measures or Dashboard_Plan. These are guide sheets.
5. Click **Transform Data** (not Load).

## 3. Transform the data (Power Query)

### 3.1 Rename queries

In the **Queries** pane, right-click each query > **Rename** and remove any number suffix. The names must match exactly, because every measure refers to them:

`Dim_Asset`, `Dim_Control`, `Fact_Asset_Control`, `Fact_Control_Test`, `Fact_Review_Schedule`, `Fact_Exceptions`, `Fact_Vulnerabilities`

### 3.2 Check headers

If any table shows `Column1`, `Column2` instead of real column names, select it and click **Home > Use First Row as Headers**. (This happened with Dim_Control during the original build.)

### 3.3 Set data types

Click the type icon at the left of each column header. When asked, choose **Replace current** so the existing "Changed Type" step is edited instead of a second conversion being stacked on top. Tip: Ctrl+click several columns of the same type and set them at once.

**Dim_Asset**
- Text: `Asset_ID`
- Decimal Number: `Inherent_Risk_Score`
- Date: `Commission_Date`

**Dim_Control**
- Text: `Control_ID` (all columns in this table are Text)

**Fact_Asset_Control**
- Text: `Asset_Control_Key`, `Asset_ID`, `Control_ID`
- Decimal Number: `Implementation_Score`
- Date: `Last_Test_Date`, `Next_Review_Due`

**Fact_Control_Test**
- Text: `Test_ID`, `Asset_Control_Key`, `Asset_ID`, `Control_ID`
- Date: `Test_Date`

**Fact_Review_Schedule**
- Text: `Review_ID`, `Asset_Control_Key`, `Asset_ID`, `Control_ID`
- Date: `Due_Date`
- Whole Number: `Days_Overdue`

**Fact_Exceptions**
- Text: `Exception_ID`, `Asset_Control_Key`, `Asset_ID`, `Control_ID`
- Date: `Approval_Date`, `Expiration_Date`
- Whole Number: `Exception_Term_Days`, `Days_To_Expiration`

**Fact_Vulnerabilities**
- Text: `Vuln_ID`, `Asset_ID`
- Decimal Number: `CVSS_Score`
- Date: `Discovered_Date`, `Remediated_Date`
- Whole Number: `SLA_Days`, `Days_Open`, `Days_To_Remediate`

Use **Date**, not Date/Time, for every date column. Date/Time columns will not match the Dim_Date relationship.

### 3.4 Check data quality

1. **View** tab > tick **Column quality**.
2. **Error** must be 0% in every column. **Empty** values are expected and intentional:

| Table | Column(s) | Expected Empty | Why |
|---|---|---|---|
| Fact_Asset_Control | Implementation_Score, Test_Frequency, Last_Test_Date, Last_Test_Result, Next_Review_Due | about 22% | "Not Applicable" asset-control pairs (for example EDR on applications) |
| Fact_Control_Test | Finding_Severity | about 71% | Tests that passed cleanly have no finding |
| Fact_Vulnerabilities | Remediated_Date, Days_To_Remediate | about 16% | Vulnerabilities still open |

3. Do not replace these blanks with 0. Measures such as coverage and MTTR rely on them being blank.
4. Column quality profiles only the first 1,000 rows by default (note at bottom left), so small differences from the percentages above are normal.

### 3.5 Load

Click **Close & Apply**.

## 4. Create the Dim_Date table

### Purpose

Dim_Date is a calendar table with one row per day from January 1, 2025 to December 31, 2027 (1,095 rows). It lets every date-based visual (test pass rate by quarter, reviews due by month, vulnerabilities discovered by month) share one consistent set of years, quarters and months. It also enables DAX time-intelligence functions such as `DATESINPERIOD`, used by the 90-day test failure KRI.

It is created in DAX, not imported from Excel.

### Steps

1. **Table view** > **Table tools > New table**.
2. Replace `Table =` with:

```DAX
Dim_Date =
ADDCOLUMNS (
    CALENDAR ( DATE ( 2025, 1, 1 ), DATE ( 2027, 12, 31 ) ),
    "Year", YEAR ( [Date] ),
    "Quarter", "Q" & QUARTER ( [Date] ),
    "Year Quarter", YEAR ( [Date] ) & "-Q" & QUARTER ( [Date] ),
    "Month", FORMAT ( [Date], "MMM yyyy" ),
    "Month Sort", YEAR ( [Date] ) * 100 + MONTH ( [Date] )
)
```

3. **Table tools > Mark as date table**, choose the `Date` column.
4. Confirm data types: `Date` = Date, `Month Sort` = Whole Number.
5. Click the `Month` column > **Column tools > Sort by column > Month Sort**.
6. Optional: right-click `Month Sort` > **Hide in report view**.

### Columns

| Column | Example | Purpose |
|---|---|---|
| Date | 2026-10-20 | Key that relates to each fact table's date |
| Year | 2026 | Yearly grouping |
| Quarter | Q4 | Quarter label |
| Year Quarter | 2026-Q4 | Quarterly trend axis (sorts correctly as text) |
| Month | Oct 2026 | Monthly axis label |
| Month Sort | 202610 | Helper so `Month` sorts chronologically instead of alphabetically |

## 5. Model view: table relationships

### Step 1: Remove auto-detected fact-to-fact relationships

1. **Model view > Manage relationships**.
2. Delete any relationship where **both** tables start with `Fact_` (for example Fact_Control_Test to Fact_Asset_Control on `Asset_Control_Key`), whichever direction it points.
3. `Asset_Control_Key` is for drill-through and lookups only. Linking fact tables to each other creates ambiguous filter paths.

### Step 2: Create the 13 relationships

**Manage relationships > New.** Fact table on top, dimension table on the bottom, click the matching column in each. Every relationship is:
- **Cardinality:** Many to one (*:1)
- **Cross filter direction:** Single
- **Make this relationship active:** ticked

Notation is `Table[Column]`.

| # | From (many side) | To (one side) | Created by |
|---|---|---|---|
| 1 | Fact_Asset_Control[Asset_ID] | Dim_Asset[Asset_ID] | Auto-detected |
| 2 | Fact_Control_Test[Asset_ID] | Dim_Asset[Asset_ID] | Auto-detected |
| 3 | Fact_Review_Schedule[Asset_ID] | Dim_Asset[Asset_ID] | Auto-detected |
| 4 | Fact_Exceptions[Asset_ID] | Dim_Asset[Asset_ID] | Auto-detected |
| 5 | Fact_Vulnerabilities[Asset_ID] | Dim_Asset[Asset_ID] | Auto-detected |
| 6 | Fact_Asset_Control[Control_ID] | Dim_Control[Control_ID] | Manual (after fixing Dim_Control headers) |
| 7 | Fact_Control_Test[Control_ID] | Dim_Control[Control_ID] | Manual |
| 8 | Fact_Review_Schedule[Control_ID] | Dim_Control[Control_ID] | Manual |
| 9 | Fact_Exceptions[Control_ID] | Dim_Control[Control_ID] | Manual |
| 10 | Fact_Control_Test[Test_Date] | Dim_Date[Date] | Manual |
| 11 | Fact_Review_Schedule[Due_Date] | Dim_Date[Date] | Manual |
| 12 | Fact_Exceptions[Expiration_Date] | Dim_Date[Date] | Manual |
| 13 | Fact_Vulnerabilities[Discovered_Date] | Dim_Date[Date] | Manual |

Fact_Vulnerabilities has no Control_ID, so it relates only to Dim_Asset and Dim_Date. Fact_Asset_Control is a current-state table with no main date, so it relates only to Dim_Asset and Dim_Control.

### Step 3: Check

In Model view, every line runs from a `Dim_` table (marked **1**) to a `Fact_` table (marked **\***). If "Make this relationship active" is greyed out, a fact-to-fact relationship still exists somewhere.

## 6. Measures

### 6.1 Create the measures table

1. **Home > Enter data**, leave the grid empty, name it `_Measures`, click **Load**.
2. For each measure: right-click **_Measures** > **New measure**, paste the formula, press Enter.
3. Create them **in the order listed**, because later measures reference earlier ones.

### 6.2 Naming convention

- **Subject first** (plural), then the descriptor: `Assets ...`, `Controls ...`, `Date ...`, `Exceptions ...`, `Reviews ...`, `Tests ...`, `Vulns ...`
- **`%` at the end** of percentage measures
- KRI helper measures add a suffix: `<value measure> Status` and `<value measure> Color`
- Status and Color measures go in a **KRIs** display folder (Model view > select measures > Properties > Display folder)

To export the current list of measures at any time, open **DAX query view** and run:

```DAX
EVALUATE
SELECTCOLUMNS ( INFO.VIEW.MEASURES (), "Name", [Name], "Expression", [Expression] )
```

### 6.3 Date

| Measure | Purpose |
|---|---|
| Date As Of | Fixed snapshot date. Used by date-relative measures so results are reproducible. Swap to `TODAY()` only if the data is regenerated on a schedule. |

```DAX
Date As Of = DATE ( 2026, 10, 20 )
```

### 6.4 Assets

| Measure | Purpose |
|---|---|
| Assets Total | Count of assets in the current filter context. Base for other asset measures. |
| Assets Internet-Facing | Assets exposed to the internet; the highest-exposure population. |
| Assets Unsupported Platform | **KRI.** Assets on an operating system with no vendor patches (Windows Server 2012 R2, Ubuntu 18.04, CentOS 7). |
| Assets Internet-Facing Without Full MFA | **KRI.** Internet-facing assets where MFA (CTL-08) is not fully implemented. Top initial-access risk. |
| Assets Crown Jewels With Control Gap | Mission Critical assets with at least one control not fully implemented. Strict definition: one partial control counts. |
| Assets Crown Jewels With Control Gap % | **KRI.** The count above as a share of all Mission Critical assets. |

```DAX
Assets Total = COUNTROWS ( Dim_Asset )
```
```DAX
Assets Internet-Facing = CALCULATE ( [Assets Total], Dim_Asset[Internet_Facing] = "Yes" )
```
```DAX
Assets Unsupported Platform = CALCULATE ( [Assets Total], Dim_Asset[Unsupported_Platform] = "Yes" )
```
```DAX
Assets Internet-Facing Without Full MFA =
CALCULATE (
    DISTINCTCOUNT ( Fact_Asset_Control[Asset_ID] ),
    Fact_Asset_Control[Control_ID] = "CTL-08",
    Fact_Asset_Control[Applicable] = "Yes",
    Fact_Asset_Control[Implementation_Status] <> "Implemented",
    Dim_Asset[Internet_Facing] = "Yes"
)
```
```DAX
Assets Crown Jewels With Control Gap =
CALCULATE (
    DISTINCTCOUNT ( Fact_Asset_Control[Asset_ID] ),
    Fact_Asset_Control[Implementation_Status] IN { "Not Implemented", "Partially Implemented", "Risk Accepted (Exception)" },
    Dim_Asset[Criticality] = "Mission Critical"
)
```
```DAX
Assets Crown Jewels With Control Gap % =
DIVIDE (
    [Assets Crown Jewels With Control Gap],
    CALCULATE ( [Assets Total], Dim_Asset[Criticality] = "Mission Critical" )
)
```

### 6.5 Controls

| Measure | Purpose |
|---|---|
| Controls Applicable | Asset-control pairs where the control is required. Denominator for coverage. |
| Controls Implemented | Asset-control pairs where the control is fully implemented. |
| Controls Coverage % | **KRI.** Share of required controls that are fully implemented. Core posture measure. |
| Controls Gaps | Required controls that are not implemented, partially implemented, or covered only by an exception. |
| Controls EDR Coverage % (Servers) | **KRI.** Coverage % for EDR (CTL-05) only. EDR applies to servers only. The same pattern works for any control ID. |

```DAX
Controls Applicable = CALCULATE ( COUNTROWS ( Fact_Asset_Control ), Fact_Asset_Control[Applicable] = "Yes" )
```
```DAX
Controls Implemented = CALCULATE ( COUNTROWS ( Fact_Asset_Control ), Fact_Asset_Control[Implementation_Status] = "Implemented" )
```
```DAX
Controls Coverage % = DIVIDE ( [Controls Implemented], [Controls Applicable] )
```
```DAX
Controls Gaps =
CALCULATE (
    COUNTROWS ( Fact_Asset_Control ),
    Fact_Asset_Control[Implementation_Status] IN { "Not Implemented", "Partially Implemented", "Risk Accepted (Exception)" }
)
```
```DAX
Controls EDR Coverage % (Servers) = CALCULATE ( [Controls Coverage %], Fact_Asset_Control[Control_ID] = "CTL-05" )
```

### 6.6 Tests

| Measure | Purpose |
|---|---|
| Tests Performed | Number of control tests. Responds to date filters through Dim_Date. |
| Tests Pass Rate % | Share of tests that passed cleanly. Used for the quarterly trend chart. |
| Tests Failure Rate % | Share of tests that failed (all history in the current filter). |
| Tests Failure Rate 90 Days % | **KRI.** Failure rate for the 90 days up to Date As Of. A KRI should reflect current performance, not two years of history. |

```DAX
Tests Performed = COUNTROWS ( Fact_Control_Test )
```
```DAX
Tests Pass Rate % = DIVIDE ( CALCULATE ( [Tests Performed], Fact_Control_Test[Result] = "Pass" ), [Tests Performed] )
```
```DAX
Tests Failure Rate % = DIVIDE ( CALCULATE ( [Tests Performed], Fact_Control_Test[Result] = "Fail" ), [Tests Performed] )
```
```DAX
Tests Failure Rate 90 Days % =
VAR d = [Date As Of]
RETURN
    CALCULATE ( [Tests Failure Rate %], DATESINPERIOD ( Dim_Date[Date], d, -90, DAY ) )
```

### 6.7 Reviews

| Measure | Purpose |
|---|---|
| Reviews Overdue | Control reviews past their due date and not completed. Ignores date slicers on purpose (snapshot). |
| Reviews Overdue % | **KRI.** Overdue reviews as a share of applicable controls. Untested controls mean unknown assurance. |
| Reviews Due Next 30 Days | Upcoming workload for the GRC testing team. |

```DAX
Reviews Overdue =
CALCULATE (
    COUNTROWS ( Fact_Review_Schedule ),
    Fact_Review_Schedule[Review_Status] = "Overdue",
    REMOVEFILTERS ( Dim_Date )
)
```
```DAX
Reviews Overdue % = DIVIDE ( [Reviews Overdue], [Controls Applicable] )
```
```DAX
Reviews Due Next 30 Days =
VAR d = [Date As Of]
RETURN
    CALCULATE (
        COUNTROWS ( Fact_Review_Schedule ),
        Fact_Review_Schedule[Due_Date] >= d,
        Fact_Review_Schedule[Due_Date] <= d + 30,
        Fact_Review_Schedule[Review_Status] <> "Overdue",
        REMOVEFILTERS ( Dim_Date )
    )
```

### 6.8 Exceptions

| Measure | Purpose |
|---|---|
| Exceptions Active | Approved exceptions currently in force. |
| Exceptions Expired Not Remediated | **KRI.** Exceptions past their expiration date with the gap still open: risk accepted beyond its approved term. |
| Exceptions Expiring 30 Days | **KRI.** Forward-looking: active exceptions that need renewal or remediation within 30 days. |
| Exceptions High Residual Risk | Active exceptions rated High residual risk. |

```DAX
Exceptions Active =
CALCULATE ( COUNTROWS ( Fact_Exceptions ), Fact_Exceptions[Exception_Status] = "Active", REMOVEFILTERS ( Dim_Date ) )
```
```DAX
Exceptions Expired Not Remediated =
CALCULATE ( COUNTROWS ( Fact_Exceptions ), Fact_Exceptions[Exception_Status] = "Expired - Not Renewed", REMOVEFILTERS ( Dim_Date ) )
```
```DAX
Exceptions Expiring 30 Days =
CALCULATE (
    COUNTROWS ( Fact_Exceptions ),
    Fact_Exceptions[Exception_Status] = "Active",
    Fact_Exceptions[Days_To_Expiration] <= 30,
    REMOVEFILTERS ( Dim_Date )
)
```
```DAX
Exceptions High Residual Risk = CALCULATE ( [Exceptions Active], Fact_Exceptions[Residual_Risk] = "High" )
```

### 6.9 Vulnerabilities

| Measure | Purpose |
|---|---|
| Vulns Open | Vulnerabilities not yet remediated. |
| Vulns Open Crit/High Past SLA | **KRI.** Open Critical and High vulnerabilities older than their SLA (Critical 15 days, High 30). Directly exploitable exposure. |
| Vulns SLA Compliance % | **KRI.** Share of closed vulnerabilities remediated within SLA. Measures remediation discipline. |
| Vulns MTTR Critical (days) | Mean time to remediate Critical vulnerabilities (closed only; open ones are blank and excluded). |

```DAX
Vulns Open =
CALCULATE ( COUNTROWS ( Fact_Vulnerabilities ), Fact_Vulnerabilities[Vuln_Status] = "Open", REMOVEFILTERS ( Dim_Date ) )
```
```DAX
Vulns Open Crit/High Past SLA =
CALCULATE ( [Vulns Open], Fact_Vulnerabilities[Past_SLA] = "Yes", Fact_Vulnerabilities[Severity] IN { "Critical", "High" } )
```
```DAX
Vulns SLA Compliance % =
DIVIDE (
    CALCULATE ( COUNTROWS ( Fact_Vulnerabilities ), Fact_Vulnerabilities[Vuln_Status] = "Closed", Fact_Vulnerabilities[Past_SLA] = "No" ),
    CALCULATE ( COUNTROWS ( Fact_Vulnerabilities ), Fact_Vulnerabilities[Vuln_Status] = "Closed" )
)
```
```DAX
Vulns MTTR Critical (days) =
CALCULATE ( AVERAGE ( Fact_Vulnerabilities[Days_To_Remediate] ), Fact_Vulnerabilities[Severity] = "Critical" )
```

### 6.10 KRI Status measures (KRIs folder)

**Purpose:** each returns `Green`, `Amber` or `Red` for one KRI. Thresholds live only here, so a change in risk appetite means editing one formula, not every visual.

- Percentage KRIs return blank when there is no data, so a filtered-out slice does not falsely show Green.
- Count KRIs use `+ 0` so that "no rows" counts as 0 and shows Green.

| KRI | Green | Amber | Red |
|---|---|---|---|
| Controls Coverage % | >= 85% | 75% to 85% | < 75% |
| Controls EDR Coverage % (Servers) | >= 95% | 85% to 95% | < 85% |
| Reviews Overdue % | <= 5% | 5% to 10% | > 10% |
| Tests Failure Rate 90 Days % | <= 10% | 10% to 20% | > 20% |
| Vulns Open Crit/High Past SLA | <= 25 | 26 to 60 | > 60 |
| Vulns SLA Compliance % | >= 90% | 80% to 90% | < 80% |
| Assets Internet-Facing Without Full MFA | 0 | 1 to 10 | > 10 |
| Assets Unsupported Platform | 0 | 1 to 25 | > 25 |
| Assets Crown Jewels With Control Gap % | <= 5% | 5% to 15% | > 15% |
| Exceptions Expired Not Remediated | 0 | 1 to 50 | > 50 |
| Exceptions Expiring 30 Days | <= 25 | 26 to 75 | > 75 |

```DAX
Controls Coverage % Status = VAR v = [Controls Coverage %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v >= 0.85, "Green", v >= 0.75, "Amber", "Red" ) )
```
```DAX
Controls EDR Coverage % (Servers) Status = VAR v = [Controls EDR Coverage % (Servers)] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v >= 0.95, "Green", v >= 0.85, "Amber", "Red" ) )
```
```DAX
Reviews Overdue % Status = VAR v = [Reviews Overdue %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 0.05, "Green", v <= 0.10, "Amber", "Red" ) )
```
```DAX
Tests Failure Rate 90 Days % Status = VAR v = [Tests Failure Rate 90 Days %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 0.10, "Green", v <= 0.20, "Amber", "Red" ) )
```
```DAX
Vulns Open Crit/High Past SLA Status = VAR v = [Vulns Open Crit/High Past SLA] + 0 RETURN SWITCH ( TRUE (), v <= 25, "Green", v <= 60, "Amber", "Red" )
```
```DAX
Vulns SLA Compliance % Status = VAR v = [Vulns SLA Compliance %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v >= 0.90, "Green", v >= 0.80, "Amber", "Red" ) )
```
```DAX
Assets Internet-Facing Without Full MFA Status = VAR v = [Assets Internet-Facing Without Full MFA] + 0 RETURN SWITCH ( TRUE (), v = 0, "Green", v <= 10, "Amber", "Red" )
```
```DAX
Assets Unsupported Platform Status = VAR v = [Assets Unsupported Platform] + 0 RETURN SWITCH ( TRUE (), v = 0, "Green", v <= 25, "Amber", "Red" )
```
```DAX
Assets Crown Jewels With Control Gap % Status = VAR v = [Assets Crown Jewels With Control Gap %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 0.05, "Green", v <= 0.15, "Amber", "Red" ) )
```
```DAX
Exceptions Expired Not Remediated Status = VAR v = [Exceptions Expired Not Remediated] + 0 RETURN SWITCH ( TRUE (), v = 0, "Green", v <= 50, "Amber", "Red" )
```
```DAX
Exceptions Expiring 30 Days Status = VAR v = [Exceptions Expiring 30 Days] + 0 RETURN SWITCH ( TRUE (), v <= 25, "Green", v <= 75, "Amber", "Red" )
```

### 6.11 KRI Color measures (KRIs folder)

**Purpose:** each converts its Status into a hex color, used by conditional formatting (**fx > Format style: Field value**). Colors: Green `#2E7D32`, Amber `#F9A825`, Red `#C62828`. The Color measures never need editing when thresholds change.

```DAX
Controls Coverage % Color = SWITCH ( [Controls Coverage % Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Controls EDR Coverage % (Servers) Color = SWITCH ( [Controls EDR Coverage % (Servers) Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Reviews Overdue % Color = SWITCH ( [Reviews Overdue % Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Tests Failure Rate 90 Days % Color = SWITCH ( [Tests Failure Rate 90 Days % Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Vulns Open Crit/High Past SLA Color = SWITCH ( [Vulns Open Crit/High Past SLA Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Vulns SLA Compliance % Color = SWITCH ( [Vulns SLA Compliance % Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Assets Internet-Facing Without Full MFA Color = SWITCH ( [Assets Internet-Facing Without Full MFA Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Assets Unsupported Platform Color = SWITCH ( [Assets Unsupported Platform Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Assets Crown Jewels With Control Gap % Color = SWITCH ( [Assets Crown Jewels With Control Gap % Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Exceptions Expired Not Remediated Color = SWITCH ( [Exceptions Expired Not Remediated Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```
```DAX
Exceptions Expiring 30 Days Color = SWITCH ( [Exceptions Expiring 30 Days Status], "Green", "#2E7D32", "Amber", "#F9A825", "Red", "#C62828" )
```

### 6.12 Formatting

- **Format as %, 1 decimal:** Controls Coverage %, Controls EDR Coverage % (Servers), Tests Pass Rate %, Tests Failure Rate %, Tests Failure Rate 90 Days %, Reviews Overdue %, Vulns SLA Compliance %, Assets Crown Jewels With Control Gap %
- **1 decimal:** Vulns MTTR Critical (days)
- **Display folder `KRIs`:** all 11 Status and 11 Color measures
- **Cards:** set **Display units** to **None** to see exact values instead of 15K / 11K

## 7. Verification values

After building, these values confirm the model is correct (original dataset, no filters applied).

| Measure | Expected |
|---|---|
| Assets Total | 1,200 |
| Assets Internet-Facing | 156 |
| Controls Applicable | 14,971 |
| Controls Implemented | 11,140 |
| Controls Coverage % | 74.4% (Red) |
| Controls Gaps | 3,831 |
| Controls EDR Coverage % (Servers) | 71.1% (Red) |
| Tests Performed | 47,859 |
| Tests Pass Rate % | 71.4% |
| Tests Failure Rate % | 21.0% |
| Tests Failure Rate 90 Days % | 19.7% (Amber) |
| Reviews Overdue | 2,172 |
| Reviews Overdue % | 14.5% (Red) |
| Reviews Due Next 30 Days | 2,132 |
| Exceptions Active | 639 |
| Exceptions Expired Not Remediated | 353 (Red) |
| Exceptions Expiring 30 Days | 98 (Red) |
| Exceptions High Residual Risk | 100 |
| Vulns Open | 1,081 |
| Vulns Open Crit/High Past SLA | 67 (Red) |
| Vulns SLA Compliance % | 70.2% (Red) |
| Vulns MTTR Critical (days) | 14.7 |
| Assets Unsupported Platform | 48 (Red) |
| Assets Internet-Facing Without Full MFA | 25 (Red) |
| Assets Crown Jewels With Control Gap | 124 |
| Assets Crown Jewels With Control Gap % | 83.8% (Red) |

If a value is off, the cause is usually a query name with a leftover number suffix, a date column typed as Date/Time, or a typo in a referenced measure name.
