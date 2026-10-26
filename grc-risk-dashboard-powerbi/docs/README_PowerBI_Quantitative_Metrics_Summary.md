# Quantitative Risk and Security Operations Metrics

**Report:** Cyber Risk Management Dashboard (Power BI)
**Source file:** `Cyber_Risk_PowerBI_Dataset.xlsx` (Excel workbook)
**Added:** October 23, 2026
**Data snapshot:** October 20, 2026
**All data is fictional and randomly generated.**

---

## Contents

1. [Overview](#1-overview)
2. [New tables](#2-new-tables)
3. [Where the data comes from](#3-where-the-data-comes-from)
4. [Import and model setup](#4-import-and-model-setup)
5. [Measures](#5-measures)
6. [KRIs](#6-kris)
7. [Display folders](#7-display-folders)
8. [Verification values](#8-verification-values)
9. [What the data shows](#9-what-the-data-shows)
10. [Assumptions and limitations](#10-assumptions-and-limitations)

---

## 1. Overview

The original dashboard measured the security program in counts and percentages (control coverage, overdue reviews, vulnerabilities past SLA). This addition brings in two new kinds of quantitative data:

| Area | Question it answers | Method |
|---|---|---|
| **Financial risk** | How much money could we expect to lose each year, and how much do our controls save relative to what they cost? | Annualized Loss Expectancy (ALE) per asset: SLE x ARO |
| **Security operations** | How many incidents do we have, how fast do we detect and resolve them, and are people getting better at spotting phishing? | Incident records and monthly phishing simulations |

Five tables were added to the existing workbook. **The seven original tables were not changed**, so all existing pages, measures and verified values still hold.

## 2. New tables

### 2.1 Dim_Business_Unit (8 rows)

One row per business unit. Added so that phishing data, which has no assets, can be filtered by business unit.

| Column | Type | Description |
|---|---|---|
| Business_Unit | Text | Business unit name (key) |
| BU_Code | Text | Short code (PAY, FIN, HR, CRP, CIT, SOC, MKT, LGL) |
| BU_Executive | Text | Accountable executive |
| Headcount | Whole Number | Employees; used as the phishing simulation audience |

### 2.2 Fact_Asset_Risk_Quant (9,600 rows)

One row per asset per quarterly snapshot: 1,200 assets x 8 snapshots. This is a **snapshot fact table**: it records the state of each asset at a point in time, not events.

Snapshot dates: 2025-03-31, 2025-06-30, 2025-09-30, 2025-12-31, 2026-03-31, 2026-06-30, 2026-09-30, and **2026-10-20** (the as-of date, used as the current snapshot instead of 2026-12-31).

| Column | Type | Description |
|---|---|---|
| Asset_ID | Text | Links to Dim_Asset |
| Snapshot_Date | Date | Quarter-end snapshot date; links to Dim_Date |
| Asset_Value_USD | Decimal | Value of the asset to the business (replacement plus business value) |
| Exposure_Factor | Decimal (0 to 1) | Share of the asset's value lost in a single successful incident |
| Inherent_ARO | Decimal | Annual Rate of Occurrence: expected successful incidents per year **with no controls** |
| Control_Effectiveness | Decimal (0 to 1) | How well the asset's controls are implemented at that snapshot |

### 2.3 Fact_Control_Cost (32 rows)

One row per control per fiscal year (16 controls x FY2025, FY2026).

| Column | Type | Description |
|---|---|---|
| Control_ID | Text | Links to Dim_Control |
| Fiscal_Year | Whole Number | 2025 or 2026 (Summarization: Don't summarize; not related to Dim_Date) |
| License_Cost_USD | Decimal | Software, subscription and hardware cost |
| Labor_Cost_USD | Decimal | Staff time to operate the control |
| Annual_Cost_USD | Decimal | License plus labor |

### 2.4 Fact_Security_Incidents (674 rows)

One row per security incident, January 1, 2025 to October 20, 2026.

| Column | Type | Description |
|---|---|---|
| Incident_ID | Text | Unique ID |
| Asset_ID | Text | Affected asset; links to Dim_Asset |
| Incident_Type | Text | Phishing / Credential Compromise, Malware, Unauthorized Access, Data Exposure, Misconfiguration, Insider Misuse, Denial of Service, Ransomware Attempt |
| Severity | Text | Low, Medium, High, Critical |
| Detection_Source | Text | SIEM Alert, EDR Alert, IDS/IPS Alert, User Report, Third-Party Notification |
| Detecting_Control_ID | Text | CTL-07 (SIEM), CTL-05 (EDR) or CTL-04 (IDS/IPS); **blank when a person, not a control, detected it**; links to Dim_Control |
| Detected_Date | Date | Links to Dim_Date |
| Time_To_Detect_Hours | Decimal | From occurrence to detection |
| Time_To_Contain_Hours | Decimal | From detection to containment; blank if still open |
| Time_To_Resolve_Hours | Decimal | From detection to resolution; blank if still open |
| Incident_Status | Text | Closed or Open (1 open incident) |
| Financial_Impact_USD | Decimal | Actual loss: response cost, downtime, recovery |

### 2.5 Fact_Phishing_Simulation (176 rows)

One row per monthly phishing campaign per business unit: 22 months (January 2025 to October 2026) x 8 business units.

| Column | Type | Description |
|---|---|---|
| Campaign_ID | Text | One ID per monthly campaign (shared by all 8 business units) |
| Campaign_Date | Date | Sent between the 7th and 13th of each month; links to Dim_Date |
| Campaign_Theme | Text | Lure used (for example, Invoice payment, MFA re-enrollment) |
| Business_Unit | Text | Links to Dim_Business_Unit |
| Emails_Sent | Whole Number | Simulated phishing emails sent |
| Clicked | Whole Number | Recipients who clicked the link |
| Credentials_Submitted | Whole Number | Recipients who entered credentials (a subset of Clicked) |
| Reported | Whole Number | Recipients who reported the email as phishing |

## 3. Where the data comes from

All values are generated, but **they are linked to the original data wherever it makes sense**, so the new metrics stay consistent with the rest of the dashboard.

### 3.1 Asset value
- Base value by criticality: Mission Critical $3,000,000; Business Critical $1,000,000; Administrative $250,000; Non-Critical $60,000.
- Random variation around that base (log-normal, so a few assets are worth much more than typical).
- Applications are valued 15% higher than servers. Rounded to the nearest $1,000.
- Constant across all 8 snapshots.

### 3.2 Exposure factor
- Base by data classification: PCI-DSS 0.60, PII 0.50, SOX 0.40, Other 0.25, Administrative 0.20, Non-Critical 0.10.
- Random variation of plus or minus 20%, kept between 0.05 and 0.95.
- Reasoning: breaches of regulated data carry fines, notification costs and legal exposure on top of the direct loss.

### 3.3 Inherent ARO (annual rate of occurrence without controls)
- Base about 0.10 per year (randomly 0.06 to 0.14).
- x 2.2 if Internet-Facing; x 1.6 if on an unsupported platform.
- x 1.4 for PCI-DSS, x 1.3 for PII, x 1.1 for SOX data (more attractive to attackers).

### 3.4 Control effectiveness (linked to existing data)
- **Current snapshot (2026-10-20):** the asset's average `Implementation_Score` across its applicable controls in **Fact_Asset_Control** (Implemented = 1, Partially Implemented = 0.5, Not Implemented or Risk Accepted = 0).
- **Earlier snapshots:** 0.013 lower for each quarter further back, with small random variation, reflecting gradual program improvement.
- This is the key link: **when control coverage changes, the dollar risk changes with it.**

### 3.5 Control cost
- Annual FY2026 cost per control, from $200,000 (ACLs) to $1,400,000 (SIEM); total $9,900,000.
- FY2025 is 94% of FY2026.
- Split randomly: 55% to 70% license, the rest labor.

### 3.6 Security incidents (linked to existing data)
- About 34 incidents per month at the start, falling 15% over the period.
- **Which asset is affected** is weighted by that asset's current residual ALE, so higher-risk assets have more incidents.
- **Severity mix:** about 58% Low, 30% Medium, 10% High, 2% Critical. Low incidents on Mission or Business Critical assets are sometimes raised to Medium.
- **Detection source depends on the asset's actual controls** in Fact_Asset_Control: if that asset's SIEM, EDR or IDS/IPS is implemented, a control alert is much more likely; if not, detection more often comes from a user or a third party.
- **Time to detect** depends on the source (typical values: EDR 3 hours, IDS/IPS 4, SIEM 6, User Report 30, Third-Party 160) and improves by 25% over the period.
- **Time to contain and resolve** rise with severity.
- **Financial impact** ranges by severity (Low $0 to $3,000, with half at $0; Medium $2,000 to $30,000; High $30,000 to $250,000; Critical $200,000 to $1,500,000), reduced by 40% when a control detected the incident.

### 3.7 Phishing simulations
- Emails sent = about the business unit's headcount.
- **Click rate** starts around 15% and falls about 0.4 points per month, adjusted by a business-unit risk factor: Sales & Marketing 1.45, Human Resources 1.2, Finance & Accounting 1.1, Legal & Compliance 1.05, Client Risk Platform 1.0, Payments Processing 0.95, Corporate IT 0.75, Security Operations 0.45. Minimum 1.5%.
- **Credentials submitted:** about 38% of clicks at the start, falling over time.
- **Report rate** starts around 18% and rises about 1.3 points per month, capped at 75%.

## 4. Import and model setup

### 4.1 Import
1. Close Excel. Save the updated workbook over the existing file (same folder, same name), then **Home > Refresh**. Existing values must not change.
2. **Home > Get data > Excel workbook**, select the same file, tick only the five new tables (table icon), click **Transform Data**.
3. Rename any query with a number suffix.

### 4.2 Data types (Power Query)

| Table | Date | Decimal Number | Whole Number |
|---|---|---|---|
| Fact_Asset_Risk_Quant | Snapshot_Date | Asset_Value_USD, Exposure_Factor, Inherent_ARO, Control_Effectiveness | |
| Fact_Control_Cost | | License_Cost_USD, Labor_Cost_USD, Annual_Cost_USD | Fiscal_Year |
| Fact_Security_Incidents | Detected_Date | Time_To_Detect_Hours, Time_To_Contain_Hours, Time_To_Resolve_Hours, Financial_Impact_USD | |
| Fact_Phishing_Simulation | Campaign_Date | | Emails_Sent, Clicked, Credentials_Submitted, Reported |
| Dim_Business_Unit | | | Headcount |

All ID and name columns are Text. Blank values in Detecting_Control_ID, Time_To_Contain_Hours and Time_To_Resolve_Hours are intentional. Click **Close & Apply**, then set Fiscal_Year to **Don't summarize**.

### 4.3 Relationships (9 new)

Delete any auto-detected Fact-to-Fact relationships first. All relationships: Many to one, Single direction, Active.

| # | From (many side) | To (one side) |
|---|---|---|
| 1 | Fact_Asset_Risk_Quant[Asset_ID] | Dim_Asset[Asset_ID] |
| 2 | Fact_Asset_Risk_Quant[Snapshot_Date] | Dim_Date[Date] |
| 3 | Fact_Control_Cost[Control_ID] | Dim_Control[Control_ID] |
| 4 | Fact_Security_Incidents[Asset_ID] | Dim_Asset[Asset_ID] |
| 5 | Fact_Security_Incidents[Detected_Date] | Dim_Date[Date] |
| 6 | Fact_Security_Incidents[Detecting_Control_ID] | Dim_Control[Control_ID] |
| 7 | Fact_Phishing_Simulation[Campaign_Date] | Dim_Date[Date] |
| 8 | Fact_Phishing_Simulation[Business_Unit] | Dim_Business_Unit[Business_Unit] |
| 9 | Dim_Asset[Business_Unit] | Dim_Business_Unit[Business_Unit] |

Relationship 9 links two dimension tables, so a Business Unit selection flows from Dim_Business_Unit to Dim_Asset and on to every asset-based fact table.

### 4.4 Business Unit slicer
The Business Unit slicer on every page now uses **Dim_Business_Unit[Business_Unit]** instead of Dim_Asset[Business_Unit]. The old field could not filter phishing data (it has no assets). Existing values are unaffected, because the new slicer filters Dim_Asset in the same way.

## 5. Measures

All measures live in the `_Measures` table and follow the subject-first naming convention.

### 5.1 Risk (financial)

Format the dollar measures as **Currency, 0 decimals**.

| Measure | What it measures |
|---|---|
| Risk Max Control Reduction | Assumption: the share of risk that perfectly implemented controls can remove (0.9 = 90%). A parameter, so it can be changed in one place. |
| Risk Snapshot Date | The latest snapshot date in the current filter context. Helper for the measures below. |
| Risk Asset Value | Total value of assets in scope at the snapshot. |
| Risk Inherent ALE | Expected annual loss **if no controls existed**. |
| Risk Residual ALE | Expected annual loss **with current controls**. KRI. |
| Risk Reduction ALE | Expected annual loss avoided because of controls. |
| Risk Reduction % | Share of inherent risk removed by controls. |
| Risk ROSI | Return on Security Investment: net benefit of controls relative to their cost. Program-level only. |

**Core formulas (the ALE method)**
- Single Loss Expectancy: **SLE = Asset Value x Exposure Factor**
- Inherent ALE: **SLE x Inherent ARO**
- Residual ALE: **Inherent ALE x (1 - 0.9 x Control Effectiveness)**
- ROSI: **(Risk Reduction ALE - Control Cost) / Control Cost**

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
Risk ROSI format: Model view > Properties > Formatting > Format: **Custom** > `0.0"x"` (displays 4.3x).

**Why the snapshot pattern?** The table holds 8 snapshots of the same 1,200 assets. A plain SUM would add all 8 together and overstate risk eightfold. These measures find the latest snapshot in the current context (the current one on a card, each quarter's own snapshot on a trend chart) and calculate only that one. This is called a **semi-additive** measure: it adds up across assets, but not across time.

**Why SUMX?** ALE is calculated row by row (value x factor x rate for each asset) and then summed. Multiplying three separate column totals would give a meaningless number.

**Why does ROSI ignore asset slicers?** Control costs belong to the whole program, not to individual assets. Without REMOVEFILTERS, selecting "Mission Critical" would compare a slice of risk against the full budget, and ROSI could turn falsely negative.

### 5.2 Controls (cost)

| Measure | What it measures |
|---|---|
| Controls Cost FY2026 | Total annual cost of controls in FY2026 (license plus labor). |

```DAX
Controls Cost FY2026 = CALCULATE ( SUM ( Fact_Control_Cost[Annual_Cost_USD] ), Fact_Control_Cost[Fiscal_Year] = 2026 )
```

### 5.3 Incidents

| Measure | What it measures |
|---|---|
| Incidents Total | Number of security incidents. |
| Incidents High/Critical | Incidents rated High or Critical. |
| Incidents MTTD Median (hours) | Typical time to **detect** an incident (Mean Time To Detect, calculated as a median). |
| Incidents MTTR Median (hours) | Typical time to **resolve** an incident (closed incidents only; blanks are ignored). |
| Incidents MTTD Median 90 Days (hours) | Typical detection time over the last 90 days. KRI. |
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

**Why median, not average?** A small number of incidents reported by third parties took around 180 hours to detect. They pull the average up to 31 hours, while the typical incident took 7.8. The median describes the typical incident and is not distorted by a few extreme cases. The names keep "MTTD" and "MTTR" because those are the industry terms, with "Median" added to be explicit.

### 5.4 Phishing

| Measure | What it measures |
|---|---|
| Phishing Emails Sent | Simulated phishing emails sent. |
| Phishing Click Rate % | Share of recipients who clicked the link. Lower is better. |
| Phishing Report Rate % | Share who reported the email. Higher is better. |
| Phishing Credential Submit Rate % | Share who entered credentials: the most serious failure. |
| Phishing Click Rate 30 Days % | Click rate for the most recent campaign (last 30 days). KRI. |

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

Rates are calculated as total clicks divided by total emails, not as an average of each business unit's rate. That weights each unit by its size, so a 90-person team does not count as much as a 420-person one.

## 6. KRIs

Three new KRIs, each with a Status measure (Green / Amber / Red) and a Color measure (hex code), stored in the KRIs display folder.

| KRI | Green | Amber | Red | Current |
|---|---|---|---|---|
| Risk Residual ALE | <= $12M | $12M to $18M | > $18M | $14.5M (Amber) |
| Phishing Click Rate 30 Days % | <= 5% | 5% to 10% | > 10% | 6.1% (Amber) |
| Incidents MTTD Median 90 Days (hours) | <= 8 | 8 to 24 | > 24 | 7.7 (Green) |

```DAX
Risk Residual ALE Status = VAR v = [Risk Residual ALE] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 12000000, "Green", v <= 18000000, "Amber", "Red" ) )
```
```DAX
Phishing Click Rate 30 Days % Status = VAR v = [Phishing Click Rate 30 Days %] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 0.05, "Green", v <= 0.10, "Amber", "Red" ) )
```
```DAX
Incidents MTTD Median 90 Days (hours) Status = VAR v = [Incidents MTTD Median 90 Days (hours)] RETURN IF ( ISBLANK ( v ), BLANK (), SWITCH ( TRUE (), v <= 8, "Green", v <= 24, "Amber", "Red" ) )
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

The Risk Residual ALE thresholds represent a **risk appetite**: the amount of expected annual loss leadership is willing to accept. In a real program, these come from a risk appetite statement approved by leadership. The values here are illustrative.

## 7. Display folders

Measures are grouped into collapsible folders in the Data pane (Model view > select measures > Properties > Display folder):

| Folder | Contents |
|---|---|
| Assets | Asset counts and asset-level KRIs |
| Controls | Coverage, gaps, EDR coverage, Controls Cost FY2026 |
| Date | Date As Of |
| Exceptions | Exception counts |
| Incidents | All Incidents measures |
| KRIs | All Status and Color measures |
| Phishing | All Phishing measures |
| Reviews | Review counts, overdue measures |
| Risk | All financial risk measures |
| Tests | Test counts and rates |
| Vulns | Vulnerability measures |

## 8. Verification values

No slicers applied. All values were calculated from the delivered file, and all tests passed in Power BI.

| Measure | Expected |
|---|---|
| Risk Asset Value | $1,002,608,000 |
| Risk Inherent ALE | $67,433,738 |
| Risk Residual ALE | $14,472,800 (Amber) |
| Risk Reduction ALE | $52,960,938 |
| Risk Reduction % | 78.5% |
| Controls Cost FY2026 | $9,900,000 |
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

## 9. What the data shows

**Financial risk**
- **Residual ALE fell from $20.1M to $14.5M over 8 quarters** as control coverage improved:

| Snapshot | Residual ALE |
|---|---|
| 2025-03-31 | $20.09M |
| 2025-06-30 | $19.12M |
| 2025-09-30 | $18.44M |
| 2025-12-31 | $17.60M |
| 2026-03-31 | $16.76M |
| 2026-06-30 | $16.05M |
| 2026-09-30 | $15.23M |
| 2026-10-20 | $14.47M |

- **Risk is concentrated:** Mission Critical assets carry $7.75M and Business Critical $5.69M of residual ALE; Non-Critical carries only $0.20M. So Non-Critical's weak coverage (64.7%) matters much less in dollars than its red color suggests.
- **By business unit:** Payments Processing carries $5.28M (37% of the total), followed by Client Risk Platform ($2.99M). Security Operations ($0.19M) and Corporate IT ($0.23M) carry the least.
- **Highest-risk assets:** mkt-mq-prd-005 (Business Critical, PII, Internet-Facing, $327K residual ALE) and Finance Onboarding API (0501) (Mission Critical, SOX, Internet-Facing, $319K).
- **Program value:** controls remove an estimated $53.0M of $67.4M expected annual loss (78.5%) for $9.9M a year: ROSI 4.3x.

**Security operations**
- **Controls detect incidents faster than people:**

| Detection source | Median hours to detect |
|---|---|
| EDR Alert | 2.7 |
| IDS/IPS Alert | 4.2 |
| SIEM Alert | 5.4 |
| User Report | 25.5 |
| Third-Party Notification | 179.5 |

  This ties security operations back to control coverage: assets with SIEM and EDR implemented are found and fixed sooner.
- 62% of incidents were detected by a control. Raising that is a concrete goal.
- **Actual losses:** $11.6M in 2025 and $7.5M from January 1 to October 20, 2026, below the model's predicted residual ALE. Realized losses below expected loss is normal for a single year, because rare large incidents drive most of the expected value.
- **Phishing:** the click rate fell from 14.1% (January 2025) to 6.1% (October 2026), and the report rate rose from 17.5% to 43.4%. In the latest campaign, Payments Processing (8.0%) and Sales & Marketing (7.5%) had the highest click rates, and Security Operations the lowest (1.1%).

## 10. Assumptions and limitations

Be ready to explain these if the numbers are questioned:

1. **All inputs are estimates.** Asset values, exposure factors and occurrence rates are judgments. Small changes move ALE and ROSI substantially, so present them as directional, not precise.
2. **The 0.9 maximum reduction** is an assumption that controls can never remove all risk. It is a measure (`Risk Max Control Reduction`), so it can be changed to test sensitivity.
3. **Control effectiveness is an implementation score, not a tested effectiveness.** A control that is "Implemented" but failing its tests still counts as effective here. A stronger version would weight effectiveness by recent test results from Fact_Control_Test.
4. **Per-asset ALE is a simplification.** Real quantitative risk programs (for example, FAIR) usually model **risk scenarios** (such as "ransomware on payments systems") with probability ranges and simulation, rather than one value per asset.
5. **ROSI gives full credit to these 16 controls**, ignoring other factors such as training, insurance and architecture, and counts only license and labor costs.
6. **ROSI is program-level only.** It cannot be split by control, because ALE is not attributed to individual controls.
7. **Phishing data is by business unit only**, so the Criticality, Data Classification and Internet-Facing slicers do not affect it.
