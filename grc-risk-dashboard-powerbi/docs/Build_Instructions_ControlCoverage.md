# Cyber Risk Dashboard: Power BI Build Instructions

Step-by-step instructions to rebuild the Power BI model from `Cyber_Risk_PowerBI_Dataset.xlsx`.

The workbook holds **12 data tables**: the 7 original control, testing, exception and vulnerability tables, plus 5 quantitative tables: financial risk, control cost, security incidents, phishing simulations and a business unit dimension. All data is fictional.

**Snapshot date for all status fields:** October 20, 2026

---

## Contents

1. [Import the data](#1-import-the-data)
2. [Transform the data (Power Query)](#2-transform-the-data-power-query)
3. [Create the Dim_Date table](#3-create-the-dim_date-table)
4. [Model view: table relationships](#4-model-view-table-relationships)
5. [Measures](#5-measures)
6. [Verification values](#6-verification-values)
7. [Report formatting standards](#7-report-formatting-standards)
8. [Report pages](#8-report-pages)

---

## 1. Import the data

1. **Home > Get data > Excel workbook**, select the file.
2. In the Navigator, each data sheet appears twice: once as a **sheet** and once as an **Excel table** with the same name. Power BI adds a number to one of them (for example `Fact_Control_Test4`). They contain the same data.
3. Select **one version of each** of these 12 items (the table-icon versions are preferred):

| Original tables | 
|---|
| Dim_Asset | 
| Dim_Business_Unit |
| Dim_Control |
| Fact_Asset_Control | 
| Fact_Control_Cost |
| Fact_Control_Test | 
| Fact_Exceptions |
| Fact_Phishing_Simulation |
| Fact_Review_Schedule | 
| Fact_Security_Incidents | 
| Fact_Asset_Risk_Quant |
| Fact_Vulnerabilities | 


4. Do **not** select README, KRI_Definitions, DAX_Measures or Dashboard_Plan. These are guide sheets.
5. Click **Transform Data** (not Load).

## 2. Transform the data (Power Query)

### 2.1 Rename queries

In the **Queries** pane, right-click each query > **Rename** and remove any number suffix. 

`Dim_Asset`, `Dim_Control`, `Fact_Asset_Control`, `Fact_Control_Test`, `Fact_Review_Schedule`, `Fact_Exceptions`, `Fact_Vulnerabilities`, `Dim_Business_Unit`, `Fact_Asset_Risk_Quant`, `Fact_Control_Cost`, `Fact_Security_Incidents`, `Fact_Phishing_Simulation`

### 2.2 Check headers

If any table shows `Column1`, `Column2` instead of real column names, select it and click **Home > Use First Row as Headers**. 

### 2.3 Set data types

Click the type icon at the left of each column header. When asked, choose **Replace current** so the existing "Changed Type" step is edited instead of a second conversion being stacked on top. Tip: Ctrl+click several columns of the same type and set them at once.

**Use **Date**, not Date/Time, for every date column. Convention: MM/DD/YYYY.** 

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

**Dim_Business_Unit**
- Text: `Business_Unit`, `BU_Code`, `BU_Executive`
- Whole Number: `Headcount`

**Fact_Asset_Risk_Quant**
- Text: `Asset_ID`
- Date: `Snapshot_Date`
- Decimal Number: `Asset_Value_USD`, `Exposure_Factor`, `Inherent_ARO`, `Control_Effectiveness`

**Fact_Control_Cost**
- Text: `Control_ID`
- Whole Number: `Fiscal_Year`
- Decimal Number: `License_Cost_USD`, `Labor_Cost_USD`, `Annual_Cost_USD`

**Fact_Security_Incidents**
- Text: `Incident_ID`, `Asset_ID`, `Incident_Type`, `Severity`, `Detection_Source`, `Detecting_Control_ID`, `Incident_Status`
- Date: `Detected_Date`
- Decimal Number: `Time_To_Detect_Hours`, `Time_To_Contain_Hours`, `Time_To_Resolve_Hours`, `Financial_Impact_USD`

**Fact_Phishing_Simulation**
- Text: `Campaign_ID`, `Campaign_Theme`, `Business_Unit`
- Date: `Campaign_Date`
- Whole Number: `Emails_Sent`, `Clicked`, `Credentials_Submitted`, `Reported`


### 2.4 Check data quality

1. **View** tab > tick **Column quality**.
2. **Error** must be 0% in every column. **Empty** values are expected and intentional:

| Table | Column(s) | Expected Empty | Why |
|---|---|---|---|
| Fact_Asset_Control | Implementation_Score, Test_Frequency, Last_Test_Date, Last_Test_Result, Next_Review_Due | about 22% | "Not Applicable" asset-control pairs (for example EDR on applications) |
| Fact_Control_Test | Finding_Severity | about 71% | Tests that passed cleanly have no finding |
| Fact_Vulnerabilities | Remediated_Date, Days_To_Remediate | about 16% | Vulnerabilities still open |
| Fact_Security_Incidents | Detecting_Control_ID | about 38% | Incidents detected by a user or a third party, not by a control |
| Fact_Security_Incidents | Time_To_Contain_Hours, Time_To_Resolve_Hours | 1 row | The one incident still open |

3. Do not replace these blanks with 0. Measures such as coverage and MTTR rely on them being blank.
4. Column quality profiles only the first 1,000 rows by default (note at bottom left), so small differences from the percentages above are normal.

### 2.5 Add sort and label columns

These columns are created in Power Query, not DAX. A DAX calculated column that reads `Criticality` cannot also be used to sort `Criticality`: Power BI reports a circular dependency, because each column would depend on the other. Power Query runs before the model, so the loop never forms.

**Dim_Asset: `Criticality_Sort`** (puts criticality in order of importance instead of alphabetical)

1. Select **Dim_Asset** > **Add Column > Conditional Column**.
2. New column name: `Criticality_Sort`

| | Column Name | Operator | Value | Output |
|---|---|---|---|---|
| If | Criticality | equals | Mission Critical | 1 |
| Else If | Criticality | equals | Business Critical | 2 |
| Else If | Criticality | equals | Administrative | 3 |
| Else | | | | 4 |

3. Set the column type to **Whole Number**.

**Dim_Control: `SP800_53_Family_Short`** (optional; shorter chart labels, because bar chart Y-axis labels do not wrap)

1. Right-click **SP800_53_Family_Name** > **Duplicate Column**, rename the copy `SP800_53_Family_Short`.
2. **Transform > Replace Values**, once per row:

| Value to find | Replace with |
|---|---|
| System & Communications Protection | Sys & Comms Protection |
| System & Information Integrity | Sys & Info Integrity |
| Identification & Authentication | Identification & Auth |

### 2.6 Load

Click **Close & Apply**.

### 2.7 Column settings (after loading)

In **Table view**, select each column and set:

| Table | Column | Setting |
|---|---|---|
| Dim_Asset | Criticality | **Column tools > Sort by column > Criticality_Sort** |
| Dim_Asset | Criticality_Sort | Summarization: **Don't summarize**; **Hide in report view** |
| Dim_Date | Month Sort | Summarization: **Don't summarize**; **Hide in report view** |
| Dim_Date | Year | Summarization: **Don't summarize** |
| Fact_Control_Cost | Fiscal_Year | Summarization: **Don't summarize** |

Sort and code columns are identifiers, not quantities. With the default **Sum**, dragging one into a visual would add the numbers together and display a meaningless total.

## 3. Create the Dim_Date table

### Purpose

Dim_Date is a calendar table with one row per day from January 1, 2025 to December 31, 2027 (1,095 rows). It lets every date-based visual (test pass rate by quarter, reviews due by month, residual ALE by quarterly snapshot, incidents and phishing results by month) share one consistent set of years, quarters and months. It also enables DAX time-intelligence functions such as `DATESINPERIOD`, used by the 90-day test failure, 90-day detection time and 30-day phishing KRIs.

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

## 4. Model view: table relationships

### Step 1: Remove auto-detected fact-to-fact relationships

1. **Model view > Manage relationships**.
2. Delete any relationship where **both** tables start with `Fact_` (for example Fact_Control_Test to Fact_Asset_Control on `Asset_Control_Key`), whichever direction it points.
3. `Asset_Control_Key` is for drill-through and lookups only. Linking fact tables to each other creates ambiguous filter paths.

### Step 2: Create the 22 relationships

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
| 6 | Fact_Asset_Control[Control_ID] | Dim_Control[Control_ID] | Manual |
| 7 | Fact_Control_Test[Control_ID] | Dim_Control[Control_ID] | Manual |
| 8 | Fact_Review_Schedule[Control_ID] | Dim_Control[Control_ID] | Manual |
| 9 | Fact_Exceptions[Control_ID] | Dim_Control[Control_ID] | Manual |
| 10 | Fact_Control_Test[Test_Date] | Dim_Date[Date] | Manual |
| 11 | Fact_Review_Schedule[Due_Date] | Dim_Date[Date] | Manual |
| 12 | Fact_Exceptions[Expiration_Date] | Dim_Date[Date] | Manual |
| 13 | Fact_Vulnerabilities[Discovered_Date] | Dim_Date[Date] | Manual |
| 14 | Fact_Asset_Risk_Quant[Asset_ID] | Dim_Asset[Asset_ID] | Quantitative |
| 15 | Fact_Asset_Risk_Quant[Snapshot_Date] | Dim_Date[Date] | Quantitative |
| 16 | Fact_Control_Cost[Control_ID] | Dim_Control[Control_ID] | Quantitative |
| 17 | Fact_Security_Incidents[Asset_ID] | Dim_Asset[Asset_ID] | Quantitative |
| 18 | Fact_Security_Incidents[Detected_Date] | Dim_Date[Date] | Quantitative |
| 19 | Fact_Security_Incidents[Detecting_Control_ID] | Dim_Control[Control_ID] | Quantitative |
| 20 | Fact_Phishing_Simulation[Campaign_Date] | Dim_Date[Date] | Quantitative |
| 21 | Fact_Phishing_Simulation[Business_Unit] | Dim_Business_Unit[Business_Unit] | Quantitative |
| 22 | Dim_Asset[Business_Unit] | Dim_Business_Unit[Business_Unit] | Quantitative |

Notes:
- Fact_Vulnerabilities has no Control_ID, so it relates only to Dim_Asset and Dim_Date. Fact_Asset_Control is a current-state table with no main date, so it relates only to Dim_Asset and Dim_Control.
- Fact_Control_Cost uses a Fiscal_Year number, not a date, so it is not related to Dim_Date.
- Fact_Phishing_Simulation has no assets; it is filtered by business unit and date only.
- Relationship 22 links two dimension tables, so a Business Unit selection flows from Dim_Business_Unit to Dim_Asset and on to every asset-based fact table.
- **Check the column on every relationship you create.** During the build, the Due_Date relationship was accidentally created on Fact_Review_Schedule[Asset_ID]; every review then fell into a "(Blank)" column. In Model view, clicking a relationship line highlights the two columns it joins.

### Step 3: Check

In Model view, every line runs from a `Dim_` table (marked **1**) to a `Fact_` table (marked **\***), except relationship 22, which runs from Dim_Business_Unit to Dim_Asset. If "Make this relationship active" is greyed out, a fact-to-fact relationship still exists somewhere.

## 5. Measures

### 5.1 Create the measures table

1. **Home > Enter data**, leave the grid empty, name it `_Measures`, click **Load**.
2. For each measure: right-click **_Measures** > **New measure**, paste the formula, press Enter.
3. Create them **in the order listed**, because later measures reference earlier ones.

### 5.2 Naming convention

- **Subject first** (plural), then the descriptor: `Assets ...`, `Controls ...`, `Date ...`, `Exceptions ...`, `Incidents ...`, `Phishing ...`, `Reviews ...`, `Risk ...`, `Tests ...`, `Vulns ...`
- **`%` at the end** of percentage measures
- KRI helper measures add a suffix: `<value measure> Status` and `<value measure> Color`
- Measures are grouped in **display folders** (Model view > select measures > Properties > Display folder): one folder per subject prefix (Assets, Controls, Date, Exceptions, Incidents, Phishing, Reviews, Risk, Tests, Vulns), plus a **KRIs** folder for all Status and Color measures

To export the current list of measures at any time, open **DAX query view** and run:

```DAX
EVALUATE
SELECTCOLUMNS ( INFO.VIEW.MEASURES (), "Name", [Name], "Expression", [Expression] )
```

### 5.3 Date

| Measure | Purpose |
|---|---|
| Date As Of | Fixed snapshot date. Used by date-relative measures so results are reproducible. Swap to `TODAY()` only if the data is regenerated on a schedule. |

```DAX
Date As Of = DATE ( 2026, 10, 20 )
```

### 5.4 Assets

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

### 5.5 Controls

| Measure | Purpose |
|---|---|
| Controls Applicable | Asset-control pairs where the control is required. Denominator for coverage. |
| Controls Implemented | Asset-control pairs where the control is fully implemented. |
| Controls Coverage % | **KRI.** Share of required controls that are fully implemented. Core posture measure. |
| Controls Gaps | Required controls that are not implemented, partially implemented, or covered only by an exception. |
| Controls EDR Coverage % (Servers) | **KRI.** Coverage % for EDR (CTL-05) only. EDR applies to servers only. The same pattern works for any control ID. |
| Controls Cost FY2026 | Total annual cost of controls in FY2026 (license plus labor). Used by Risk ROSI. |

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
```DAX
Controls Cost FY2026 = CALCULATE ( SUM ( Fact_Control_Cost[Annual_Cost_USD] ), Fact_Control_Cost[Fiscal_Year] = 2026 )
```

### 5.6 Tests

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

### 5.7 Reviews

| Measure | Purpose |
|---|---|
| Reviews Overdue | Control reviews past their due date and not completed. Ignores date slicers on purpose (snapshot). |
| Reviews Overdue % | **KRI.** Overdue reviews as a share of applicable controls. Untested controls mean unknown assurance. |
| Reviews Due Next 30 Days | Upcoming workload for the GRC testing team. |
| Reviews Upcoming | Scheduled and in-progress reviews. Unlike Reviews Overdue, it keeps the date filter, so it can be spread across months (tester workload matrix). |
| Reviews Overdue Avg Days | How late overdue reviews are on average. A count says how many; this says how bad. |

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
```DAX
Reviews Upcoming = CALCULATE ( COUNTROWS ( Fact_Review_Schedule ), Fact_Review_Schedule[Review_Status] <> "Overdue" )
```
```DAX
Reviews Overdue Avg Days =
CALCULATE (
    AVERAGE ( Fact_Review_Schedule[Days_Overdue] ),
    Fact_Review_Schedule[Review_Status] = "Overdue",
    REMOVEFILTERS ( Dim_Date )
)
```

### 5.8 Exceptions

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

### 5.9 Vulnerabilities

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

### 5.10 Risk (financial quantification)
**0.9 is an assumption built into the model where a control can remove at most 90% of the risk**

Expected annual loss per asset using the Annualized Loss Expectancy (ALE) method:
- Single Loss Expectancy: **SLE = Asset Value x Exposure Factor**
- Inherent ALE (no controls): **SLE x Inherent ARO**
- Residual ALE (current controls): **Inherent ALE x (1 - 0.9 x Control Effectiveness)**
- Return on Security Investment: **ROSI = (Risk Reduction ALE - Control Cost) / Control Cost**

Control_Effectiveness at the latest snapshot is each asset's average Implementation_Score in Fact_Asset_Control, so the dollar figures move when control coverage moves. See `Quantitative_Metrics_README.md` for how every input was generated.

| Measure | Purpose |
|---|---|
| Risk Max Control Reduction | Assumption: share of risk that perfectly implemented controls can remove (0.9). A parameter, so it can be changed in one place. |
| Risk Snapshot Date | Latest snapshot date in the current filter context. Helper for the measures below. |
| Risk Asset Value | Total value of assets in scope at the snapshot. |
| Risk Inherent ALE | Expected annual loss if no controls existed. |
| Risk Residual ALE | **KRI.** Expected annual loss with current controls. |
| Risk Reduction ALE | Expected annual loss avoided because of controls. |
| Risk Reduction % | Share of inherent risk removed by controls. |
| Risk ROSI | Net benefit of controls relative to their cost. Program-level only. |

```DAX
Risk Max Control Reduction = 0.9
```
```DAX
Risk Snapshot Date = MAX ( Fact_Asset_Risk_Quant[Snapshot_Date] )
```
```DAX
Risk Asset Value =
VAR s = [Risk Snapshot Date]
RETURN
    CALCULATE ( SUM ( Fact_Asset_Risk_Quant[Asset_Value_USD] ), Fact_Asset_Risk_Quant[Snapshot_Date] = s )
```
```DAX
Risk Inherent ALE =
VAR s = [Risk Snapshot Date]
RETURN
    CALCULATE (
        SUMX (
            Fact_Asset_Risk_Quant,
            Fact_Asset_Risk_Quant[Asset_Value_USD] * Fact_Asset_Risk_Quant[Exposure_Factor] * Fact_Asset_Risk_Quant[Inherent_ARO]
        ),
        Fact_Asset_Risk_Quant[Snapshot_Date] = s
    )
```
```DAX
Risk Residual ALE =
VAR s = [Risk Snapshot Date]
VAR r = [Risk Max Control Reduction]
RETURN
    CALCULATE (
        SUMX (
            Fact_Asset_Risk_Quant,
            Fact_Asset_Risk_Quant[Asset_Value_USD] * Fact_Asset_Risk_Quant[Exposure_Factor] * Fact_Asset_Risk_Quant[Inherent_ARO]
                * ( 1 - r * Fact_Asset_Risk_Quant[Control_Effectiveness] )
        ),
        Fact_Asset_Risk_Quant[Snapshot_Date] = s
    )
```
```DAX
Risk Reduction ALE = [Risk Inherent ALE] - [Risk Residual ALE]
```
```DAX
Risk Reduction % = DIVIDE ( [Risk Reduction ALE], [Risk Inherent ALE] )
```
```DAX
Risk ROSI =
VAR red = CALCULATE ( [Risk Reduction ALE], REMOVEFILTERS ( Dim_Asset ), REMOVEFILTERS ( Dim_Business_Unit ) )
VAR cost = CALCULATE ( [Controls Cost FY2026], REMOVEFILTERS ( Dim_Control ) )
RETURN
    DIVIDE ( red - cost, cost )
```

- **Snapshot pattern:** Fact_Asset_Risk_Quant holds 8 snapshots of the same 1,200 assets. A plain SUM would add all 8 and overstate risk eightfold. These measures calculate only the latest snapshot in the current context: the current one on a card, each quarter's own snapshot on a trend chart. This is a **semi-additive** measure: it adds up across assets, but not across time.
- **SUMX:** ALE is calculated row by row (value x factor x rate for each asset) and then summed. Multiplying separate column totals would give a meaningless number.
- **ROSI ignores asset and business unit slicers:** control costs belong to the whole program. Without REMOVEFILTERS, selecting "Mission Critical" would compare a slice of risk against the full budget, and ROSI could turn falsely negative.

### 5.11 Incidents

| Measure | Purpose |
|---|---|
| Incidents Total | Number of security incidents. |
| Incidents High/Critical | Incidents rated High or Critical. |
| Incidents MTTD Median (hours) | Typical time to detect an incident. |
| Incidents MTTR Median (hours) | Typical time to resolve an incident (closed incidents only; blanks ignored). |
| Incidents MTTD Median 90 Days (hours) | **KRI.** Typical detection time over the last 90 days. |
| Incidents Detected by Controls % | Share of incidents detected by SIEM, EDR or IDS/IPS rather than by a person. |
| Incidents Financial Impact | Total actual loss from incidents. |

```DAX
Incidents Total = COUNTROWS ( Fact_Security_Incidents )
```
```DAX
Incidents High/Critical = CALCULATE ( [Incidents Total], Fact_Security_Incidents[Severity] IN { "High", "Critical" } )
```
```DAX
Incidents MTTD Median (hours) = MEDIAN ( Fact_Security_Incidents[Time_To_Detect_Hours] )
```
```DAX
Incidents MTTR Median (hours) = MEDIAN ( Fact_Security_Incidents[Time_To_Resolve_Hours] )
```
```DAX
Incidents MTTD Median 90 Days (hours) =
VAR d = [Date As Of]
RETURN
    CALCULATE ( [Incidents MTTD Median (hours)], DATESINPERIOD ( Dim_Date[Date], d, -90, DAY ) )
```
```DAX
Incidents Detected by Controls % =
DIVIDE (
    CALCULATE ( [Incidents Total], NOT ISBLANK ( Fact_Security_Incidents[Detecting_Control_ID] ) ),
    [Incidents Total]
)
```
```DAX
Incidents Financial Impact = SUM ( Fact_Security_Incidents[Financial_Impact_USD] )
```

**Median, not average:** a few incidents reported by third parties took around 180 hours to detect, pulling the average up to 31 hours while the typical incident took 7.8. The median describes the typical incident without distortion from a few extreme cases.

### 5.12 Phishing

| Measure | Purpose |
|---|---|
| Phishing Emails Sent | Simulated phishing emails sent. |
| Phishing Click Rate % | Share of recipients who clicked. Lower is better. |
| Phishing Report Rate % | Share who reported the email. Higher is better. |
| Phishing Credential Submit Rate % | Share who entered credentials: the most serious failure. |
| Phishing Click Rate 30 Days % | **KRI.** Click rate for the most recent campaign (last 30 days). |

```DAX
Phishing Emails Sent = SUM ( Fact_Phishing_Simulation[Emails_Sent] )
```
```DAX
Phishing Click Rate % = DIVIDE ( SUM ( Fact_Phishing_Simulation[Clicked] ), [Phishing Emails Sent] )
```
```DAX
Phishing Report Rate % = DIVIDE ( SUM ( Fact_Phishing_Simulation[Reported] ), [Phishing Emails Sent] )
```
```DAX
Phishing Credential Submit Rate % = DIVIDE ( SUM ( Fact_Phishing_Simulation[Credentials_Submitted] ), [Phishing Emails Sent] )
```
```DAX
Phishing Click Rate 30 Days % =
VAR d = [Date As Of]
RETURN
    CALCULATE ( [Phishing Click Rate %], DATESINPERIOD ( Dim_Date[Date], d, -30, DAY ) )
```

Rates are total clicks divided by total emails, not an average of each business unit's rate, so each unit is weighted by its size.

### 5.13 KRI Status measures (KRIs folder)

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
| Risk Residual ALE | <= $12M | $12M to $18M | > $18M |
| Phishing Click Rate 30 Days % | <= 5% | 5% to 10% | > 10% |
| Incidents MTTD Median 90 Days (hours) | <= 8 | 8 to 24 | > 24 |

The Risk Residual ALE thresholds represent a **risk appetite**: the expected annual loss leadership is willing to accept. In a real program they come from an approved risk appetite statement; these values are illustrative.

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
```DAX
Risk Residual ALE Status = VAR v = [Risk Residual ALE] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 12000000, "Green", v <= 18000000, "Amber", "Red" ) )
```
```DAX
Phishing Click Rate 30 Days % Status = VAR v = [Phishing Click Rate 30 Days %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 0.05, "Green", v <= 0.10, "Amber", "Red" ) )
```
```DAX
Incidents MTTD Median 90 Days (hours) Status = VAR v = [Incidents MTTD Median 90 Days (hours)] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 8, "Green", v <= 24, "Amber", "Red" ) )
```

### 5.14 KRI Color measures (KRIs folder)

**Purpose:** each converts its Status into a hex color, used by conditional formatting (**fx > Format style: Field value**). Colors: Green `#2E7D32`, Amber `#A15C00`, Red `#C62828`. The Color measures never need editing when thresholds change.

All three colors meet the WCAG AA contrast minimum of 4.5:1 for text on a white background (Green 5.1:1, Amber 5.2:1, Red 5.6:1). The amber was originally `#F9A825`, which measures only 2.0:1 and was hard to read as card text.

```DAX
Controls Coverage % Color = SWITCH ( [Controls Coverage % Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Controls EDR Coverage % (Servers) Color = SWITCH ( [Controls EDR Coverage % (Servers) Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Reviews Overdue % Color = SWITCH ( [Reviews Overdue % Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Tests Failure Rate 90 Days % Color = SWITCH ( [Tests Failure Rate 90 Days % Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Vulns Open Crit/High Past SLA Color = SWITCH ( [Vulns Open Crit/High Past SLA Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Vulns SLA Compliance % Color = SWITCH ( [Vulns SLA Compliance % Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Assets Internet-Facing Without Full MFA Color = SWITCH ( [Assets Internet-Facing Without Full MFA Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Assets Unsupported Platform Color = SWITCH ( [Assets Unsupported Platform Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Assets Crown Jewels With Control Gap % Color = SWITCH ( [Assets Crown Jewels With Control Gap % Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Exceptions Expired Not Remediated Color = SWITCH ( [Exceptions Expired Not Remediated Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Exceptions Expiring 30 Days Color = SWITCH ( [Exceptions Expiring 30 Days Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Risk Residual ALE Color = SWITCH ( [Risk Residual ALE Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Phishing Click Rate 30 Days % Color = SWITCH ( [Phishing Click Rate 30 Days % Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```
```DAX
Incidents MTTD Median 90 Days (hours) Color = SWITCH ( [Incidents MTTD Median 90 Days (hours) Status], "Green", "#2E7D32", "Amber", "#A15C00", "Red", "#C62828" )
```

### 5.15 Formatting

- **Format as %, 1 decimal:** Controls Coverage %, Controls EDR Coverage % (Servers), Tests Pass Rate %, Tests Failure Rate %, Tests Failure Rate 90 Days %, Reviews Overdue %, Vulns SLA Compliance %, Assets Crown Jewels With Control Gap %, Risk Reduction %, Incidents Detected by Controls %, Phishing Click Rate %, Phishing Report Rate %, Phishing Credential Submit Rate %, Phishing Click Rate 30 Days %
- **Currency, 0 decimals:** Risk Asset Value, Risk Inherent ALE, Risk Residual ALE, Risk Reduction ALE, Controls Cost FY2026, Incidents Financial Impact
- **Custom format `0.0"x"`:** Risk ROSI (Model view > Properties > Formatting > Format: Custom; displays 4.3x)
- **1 decimal:** Vulns MTTR Critical (days), Incidents MTTD Median (hours), Incidents MTTR Median (hours), Incidents MTTD Median 90 Days (hours)
- **0 decimals:** Reviews Overdue Avg Days (half a day changes no decision, and the extra digit suggests false precision)
- **Display folder `KRIs`:** all 14 Status and 14 Color measures; every other measure in the folder matching its subject prefix
- Measure formatting is on the **Measure tools** tab, which appears only while the measure is selected in the Data pane (or Model view > Properties > Formatting)
- **Cards:** set **Display units** to **None** to see exact values instead of 15K / 11K

## 6. Verification values

After building, these values confirm the model is correct (no filters applied). They were calculated directly from the workbook, and every value checked in Power BI during the build matched.

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
| Reviews Upcoming | 33,394 |
| Reviews Overdue Avg Days | 189 |
| Controls Cost FY2026 | $9,900,000 |
| Risk Asset Value | $1,002,608,000 |
| Risk Inherent ALE | $67,433,738 |
| Risk Residual ALE | $14,472,800 (Amber) |
| Risk Reduction ALE | $52,960,938 |
| Risk Reduction % | 78.5% |
| Risk ROSI | 4.3x |
| Incidents Total | 674 |
| Incidents High/Critical | 73 |
| Incidents MTTD Median (hours) | 7.8 |
| Incidents MTTD Median 90 Days (hours) | 7.7 (Green) |
| Incidents MTTR Median (hours) | 32.4 |
| Incidents Detected by Controls % | 62.0% |
| Incidents Financial Impact | $19,070,400 |
| Phishing Emails Sent | 41,321 |
| Phishing Click Rate % | 10.8% |
| Phishing Click Rate 30 Days % | 6.1% (Amber) |
| Phishing Report Rate % | 32.2% |

If a value is off, the cause is usually a query name with a leftover number suffix, a date column typed as Date/Time, or a typo in a referenced measure name.

## 7. Report formatting standards

### 7.1 Canvas and fonts

- Canvas: 16:9, 1280 x 720 pixels. Margins of 16 px on every edge, 12 px between visuals.
- Set fonts once in the theme: **View > Themes > Customize current theme > Text**.

| Theme setting | Font | Size |
|---|---|---|
| General | Segoe UI | 10 |
| Title | Segoe UI Semibold | 12 |
| Cards and KPIs | Segoe UI Semibold | 28 |
| Tab headers | Segoe UI Semibold | 12 |

- Page title text box: Segoe UI Semibold, 20 pt.
- Shorten long labels with **Rename for this visual** (right-click the field in the visual's field well). This changes only the label on that visual; the measure keeps its full name.

### 7.2 Color rules

- **Color always means KRI status.** Only measures with a threshold are colored; plain counts stay in the default text color. Non-status visuals (for example, the criticality donut) use shades of one neutral color so they are not mistaken for status.
- **Text and bars** use the Color measures: Green `#2E7D32`, Amber `#A15C00`, Red `#C62828`.
- **Heatmap cell fills** use lighter tints, stepped by lightness so they stay distinguishable for viewers with red-green color vision deficiency (about 8% of men). Problem cells are darkest, so they draw the eye first.

| Status | Fill | Text contrast with dark text |
|---|---|---|
| Red (< 75%) | `#EE9A92` | 7.1:1 |
| Amber (75% to 85%) | `#F5C26B` | 9.5:1 |
| Green (>= 85%) | `#E6F2E7` | 13.4:1 |

The original fills (`#F4C7C3`, `#FCE8B2`, `#C8E6C9`) were replaced because the red and green had almost the same lightness and looked alike under simulated deuteranopia.

### 7.3 Tables and matrices

| Setting | Heatmap matrix | Plain tables (KRI scorecard, work queues) |
|---|---|---|
| Values > Background color | `#FFFFFF` | `#FFFFFF` |
| Values > Alternate background color | `#FFFFFF` (banding off) | `#F7F7F7` |
| Grid > Horizontal gridlines | On, `#FFFFFF`, width 2 | On, `#E1E1E1`, width 1 |
| Grid > Vertical gridlines | On, `#FFFFFF`, width 2 | Off |
| Grid > Border | Off | Default |

White gridlines on the heatmap separate the colored cells into tiles. Banding is turned off there because alternating row colors compete with the status colors.

### 7.4 Slicers

Four dropdown slicers in a strip on the left of each page (X 16, width 170, height 60, at Y 64, 136, 208, 280):
Dim_Asset[Criticality], Dim_Asset[Data_Classification], Dim_Asset[Internet_Facing], **Dim_Business_Unit[Business_Unit]**.

The Business Unit slicer uses Dim_Business_Unit (not Dim_Asset[Business_Unit]) so that it also filters phishing data, which has no assets. It filters Dim_Asset the same way, so existing values are unchanged.

To reuse them on a new page: copy them from an existing page, paste on the new page, and choose **Sync** when prompted (or **View > Sync slicers**). A selection on one page then applies to all synced pages.

Do not add a date slicer to these pages. Several KRIs deliberately ignore date filters (snapshot as of October 20, 2026), so a date slicer would change some visuals but not others.

## 8. Report pages

### 8.1 Executive Summary

**Question it answers:** what is our overall cyber risk posture?

| Visual | Fields | X | Y | W | H |
|---|---|---|---|---|---|
| Page title | "Cyber Risk Executive Summary" plus "As of October 20, 2026" | 16 | 12 | 900 | 40 |
| Card 1 | Controls Coverage % (color: Controls Coverage % Color) | 202 | 64 | 167 | 105 |
| Card 2 | Reviews Overdue % (color: its Color measure) | 381 | 64 | 167 | 105 |
| Card 3 | Tests Failure Rate 90 Days % (color) | 560 | 64 | 167 | 105 |
| Card 4 | Vulns Open Crit/High Past SLA (color) | 739 | 64 | 167 | 105 |
| Card 5 | Assets Internet-Facing Without Full MFA (color) | 918 | 64 | 167 | 105 |
| Card 6 | Exceptions Expired Not Remediated (color) | 1097 | 64 | 167 | 105 |
| Line chart | X: Dim_Date[Year Quarter]; Y: Tests Pass Rate % | 202 | 181 | 400 | 255 |
| Donut | Legend: Dim_Asset[Criticality]; Values: Assets Total | 614 | 181 | 290 | 255 |
| KRI scorecard (matrix) | 11 KRI value measures, Switch values to rows: On; font color per measure from its Color measure | 916 | 181 | 348 | 523 |
| Bar chart | Y: Dim_Asset[Business_Unit]; X: Controls Coverage %, sorted ascending | 202 | 448 | 702 | 256 |

**How to color a card:** Format visual > Callout value > Color > **fx** > Format style: **Field value** > choose the matching **Color** measure (in _Measures > KRIs). The field must be the Color measure, not the value measure, or OK stays greyed out.

**Why coverage by business unit, not gap count:** Payments Processing has the most gaps (626) but the best coverage (76.8%), because it has the most assets. A raw count would point at the wrong team.

**Expected:** pass rate rises from 69.6% (2025-Q1) to 74.4% (2026-Q4); 2026-Q4 covers only October 1 to 20. Donut: 451 Non-Critical, 319 Business Critical, 282 Administrative, 148 Mission Critical. Business units range from Corporate IT (70.3%) to Payments Processing (76.8%).

### 8.2 Control Coverage

**Question it answers:** where are the control gaps: which controls, on which assets, in which framework areas?

| Visual | Fields | X | Y | W | H |
|---|---|---|---|---|---|
| Page title | "Control Coverage" | 16 | 12 | 900 | 40 |
| Card 1 | Controls Applicable (label "Required Controls"), no color | 202 | 64 | 256 | 105 |
| Card 2 | Controls Implemented ("Fully Implemented"), no color | 470 | 64 | 256 | 105 |
| Card 3 | Controls Gaps ("Control Gaps"), no color | 738 | 64 | 256 | 105 |
| Card 4 | Controls Coverage % ("Coverage"), color: Controls Coverage % Color | 1006 | 64 | 256 | 105 |
| Heatmap (matrix) | Rows: Dim_Control[Control_Name]; Columns: Dim_Asset[Criticality]; Values: Controls Coverage % | 202 | 181 | 620 | 523 |
| Bar chart | Y: Dim_Control[CSF_Function]; X: Controls Coverage % | 834 | 181 | 430 | 255 |
| Bar chart | Y: Dim_Control[SP800_53_Family_Short] (or _Name); X: Controls Coverage %; Tooltip: SP800_53_Family_Name | 834 | 448 | 430 | 256 |

Cards 1 to 3: Display units **None**.

**Heatmap settings**
- Sort: ... > Sort by > Coverage > Sort ascending (sorts by the Total column, weakest control on top).
- Cell elements > Background color > **fx** > Format style: **Rules**, Apply to: **Values and totals**:

| If value >= | and < | Color |
|---|---|---|
| 0 (Number) | 0.75 (Number) | `#EE9A92` |
| 0.75 (Number) | 0.85 (Number) | `#F5C26B` |
| 0.85 (Number) | <= 1 (Number) | `#E6F2E7` |

  Set every value box to **Number**, not Percent. In rules, "Percent" means a percentile of the range, not the value, and colors the wrong cells.
- Gridlines and banding as in section 7.3.

**Bar chart settings (both)**
- Sort axis > Coverage > Sort ascending (smallest bar on top).
- Bars > Color > **fx** > Field value > Controls Coverage % Color.
- Data labels On (Display units None, 1 decimal); X-axis Off; Y-axis title Off.
- Bar chart Y-axis labels do not wrap. Raise Y-axis > Values > Maximum width, or use the short label column from section 2.5.

**Expected values (no filters)**

| Check | Value |
|---|---|
| Cards | 14,971 / 11,140 / 3,831 / 74.4% (Red) |
| Coverage by criticality | Mission Critical 87.1% (Green), Business Critical 79.5% (Amber), Administrative 74.3% (Red), Non-Critical 64.7% (Red) |
| Heatmap top row | Configuration Baseline (Hardening), 65.7% |
| Heatmap bottom row | Backup & Recovery, 79.8% |
| DLP x Non-Critical | Blank (correct: DLP applies only to PCI-DSS, PII and SOX data, which no Non-Critical asset holds) |
| CSF Functions | Protect 73.2% (Red), Respond 74.9% (Red), Detect 75.0% (Amber), Identify 77.4% (Amber), Recover 79.8% (Amber) |
| SP 800-53 families | Configuration Management 65.7% lowest, Contingency Planning 79.8% highest; six Red, three Amber |
| Mission Critical selected | 2,118 / 1,844 / 274 / 87.1% (Green) |

**Notes for presenting**
- Detect (75.02%) shows Amber next to Respond (74.91%) in Red. The colors are correct; thresholds create hard edges, so focus on the pattern rather than one decimal.
- Only five of the six CSF 2.0 Functions appear. There is no Govern (GV), because all 16 controls are technical. Either explain that governance is tracked in the policy program, or add a governance control to the data.
- Six of the nine SP 800-53 families contain a single control, so most family bars repeat a heatmap row. Each control is counted under one primary family, although several map to more than one (for example DLP maps to AC-4, SC-7(10) and SI-4(4) but is counted under Access Control).
- By control type, Preventive controls are weakest (73.1%), against Detective (75.7%) and Corrective (76.4%): the program detects and recovers better than it prevents.

**Test:** select Internet-Facing = Yes. Every card, the heatmap and both charts should change, and coverage should rise.

### 8.3 Testing & Review Schedule

**Question it answers:** are we testing on time, what's coming, and what's most overdue?

| Visual | Fields | X | Y | W | H |
|---|---|---|---|---|---|
| Page title | "Testing & Review Schedule" | 16 | 12 | 900 | 40 |
| Card 1 | Reviews Overdue % ("Reviews Overdue"), colored | 202 | 64 | 202 | 105 |
| Card 2 | Reviews Overdue ("Overdue Reviews") | 416 | 64 | 202 | 105 |
| Card 3 | Reviews Overdue Avg Days ("Avg Days Overdue") | 630 | 64 | 202 | 105 |
| Card 4 | Reviews Due Next 30 Days ("Due Next 30 Days") | 844 | 64 | 202 | 105 |
| Card 5 | Tests Failure Rate 90 Days % ("Test Failures (90d)"), colored | 1058 | 64 | 202 | 105 |
| Matrix | Rows: Fact_Review_Schedule[Assigned_Tester]; Columns: Dim_Date[Month]; Values: Reviews Upcoming; visual filter Dim_Date[Month Sort] 202610 to 202704 | 202 | 181 | 520 | 255 |
| 100% stacked column | X: Dim_Date[Year Quarter]; Y: Tests Performed; Legend: Fact_Control_Test[Result] | 734 | 181 | 530 | 255 |
| Table | Asset_Name, Criticality, Control_Name, Assigned_Tester, Due_Date, Days_Overdue; visual filter Review_Status = Overdue; sorted by Days_Overdue descending | 202 | 448 | 700 | 256 |
| Bar chart | Y: Dim_Asset[Criticality]; X: Reviews Overdue %; bars colored by Reviews Overdue % Color | 914 | 448 | 350 | 256 |

- **Workload filter:** use Dim_Date[Month Sort] (>= 202610 and <= 202704) on the visual, not typed dates. Typed dates are read in the computer's regional format and can return no data. Month Sort is hidden: Data pane > ... > View hidden to reach it. Add it under **Filters on this visual**, not Filters on this page.
- **Result colors:** Fail `#C62828`, Pass with Exceptions `#F5C26B`, Pass `#1B5E20` (dark forest green) with white data labels. Standard forest green `#228B22` looks almost identical to the Fail red under simulated deuteranopia; `#1B5E20` is clearly darker and stays distinguishable.
- **Work queue:** if Due_Date splits into a hierarchy, choose **Due_Date** instead of **Date Hierarchy** in the field's dropdown (or turn off Auto date/time: File > Options and settings > Options > Current File > Data Load).

**Expected:** cards 14.5% (Red) / 2,172 / 189 / 2,132 / 19.7% (Amber). Workload Oct 2026 total 695 (Oct 20 to 31 only), grand total 14,595. Fail rate falls from 22.7% (2025-Q1) to 18.9% (2026-Q4). Work queue: 2,172 rows, top row fin-app-prd-003 (Patch Management), 479 days overdue. Overdue rate: Mission Critical 11.0%, Non-Critical 14.6%, Administrative 14.9%, Business Critical 15.8%.
