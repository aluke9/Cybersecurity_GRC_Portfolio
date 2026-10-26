# Cyber Risk Management Dashboard: Project Summary

**Status as of:** October 22, 2026 (data snapshot date: October 20, 2026)
**Tools:** Microsoft Excel (data source), Power BI Desktop (model and report)
**Data:** Fictional, randomly generated data for a cybersecurity risk management company

---

## Goal

Build a Power BI dashboard that reports cybersecurity risk the way a GRC (Governance, Risk and Compliance) team would: security control coverage across assets, control testing and review schedules, policy exceptions, vulnerabilities, and Key Risk Indicators (KRIs) with Red / Amber / Green status.

## What was built

### 1. The dataset (`Cyber_Risk_PowerBI_Dataset.xlsx`)

A star-schema dataset rather than one wide spreadsheet, so Power BI can slice, count and trend the data cleanly.

| Table | Rows | Contents |
|---|---|---|
| Dim_Asset | 1,200 | Servers (724) and applications (476) with criticality, data classification, internet-facing flag, business unit, platform, hosting, inherent risk score |
| Dim_Control | 16 | Security controls mapped to control type, NIST CSF 2.0 and NIST SP 800-53 |
| Fact_Asset_Control | 19,200 | One row per asset per control: applicability and implementation status |
| Fact_Control_Test | 47,859 | Control test history, January 2025 to October 2026 |
| Fact_Review_Schedule | 35,566 | Scheduled and overdue control reviews through December 2027 |
| Fact_Exceptions | 1,186 | Policy exceptions / risk acceptances |
| Fact_Vulnerabilities | 6,097 | Vulnerability findings with severity, SLA and remediation dates |

**Asset attributes requested and included**
- Criticality: Mission Critical (148), Business Critical (319), Administrative (282), Non-Critical (451)
- Data classification: PCI-DSS, SOX, PII, Administrative, Non-Critical, Other
- Internet-Facing: Yes (156) / No (1,044)

**Controls**
- The 11 requested controls: Data Encryption, ACLs, Firewalls & Network Segmentation, IDS/IPS, EDR, DLP, SIEM, MFA, IAM/PAM, Antivirus / Anti-Malware, VPN
- 5 added controls to cover NIST SP 800-53 families the original list did not touch (CM, IR, RA, CP) and to supply corrective controls: Vulnerability Scanning, Configuration Baseline (Hardening), Patch Management, Backup & Recovery, Incident Response Playbook Coverage. These are flagged `In_Original_Request = "No - added"`.

**Framework mapping decisions**
- **Control type** (Preventive / Detective / Corrective) and **NIST CSF 2.0 Function** (Govern, Identify, Protect, Detect, Respond, Recover) are kept as separate columns. CSF 2.0 does not use the Preventive / Detective / Corrective terms, so the two are related but not the same thing. Example: Patch Management is a Corrective control that CSF 2.0 places under Protect (PR.PS-02).
- **SP 800-53 families:** MFA maps to IA (Identification & Authentication), not AC. IDS/IPS, EDR, antivirus and patching map to SI (System & Information Integrity).

**Test frequency rule**
- Quarterly: PCI-DSS data or Mission Critical assets
- Semi-Annual: SOX or PII data, or Business Critical assets
- Annual: everything else

### 2. The Power BI model

- Seven tables imported from Excel and cleaned in Power Query (renamed queries, corrected data types, promoted headers on Dim_Control).
- A `Dim_Date` calendar table created in DAX (2025 to 2027) and marked as the date table.
- 13 one-to-many, single-direction relationships from the three dimension tables to the five fact tables. No fact-to-fact relationships.
- A `_Measures` table holding all measures, with KRI Status and Color measures in a `KRIs` display folder.

### 3. Measures and naming convention

- About 45 measures following a **subject-first** naming convention: `Assets ...`, `Controls ...`, `Date ...`, `Exceptions ...`, `Reviews ...`, `Tests ...`, `Vulns ...`, with `%` at the end of percentage measures.
- 11 KRIs, each with three measures: the value, a `Status` measure (Green / Amber / Red) and a `Color` measure (hex code for conditional formatting).

### 4. Executive Summary report page

Built on a 1280 x 720 canvas, Segoe UI font set through the report theme.

| Visual | Content |
|---|---|
| Page title | "Cyber Risk Executive Summary" with the as-of date |
| 6 KRI cards (167 x 105 px each) | Control Coverage, Reviews Overdue, Test Failures (90d), Crit/High Vulns Past SLA, Internet-Facing w/o MFA, Expired Exceptions; each number colored by its KRI status |
| Line chart | Control test pass rate by quarter (69.6% in 2025-Q1 rising to 74.4% in 2026-Q4) |
| Donut chart | Assets by criticality, in neutral colors so it is not confused with risk status |
| Bar chart | Control coverage % by business unit, sorted weakest first |
| Matrix | KRI scorecard: all 11 KRIs as rows, values colored by status |
| 4 slicers | Criticality, Data Classification, Internet-Facing, Business Unit (dropdown style) |

### 5. Control Coverage report page

Answers "where are the gaps?" The Executive Summary's 74.4% is an average; this page shows what sits behind it.

| Visual | Content |
|---|---|
| 4 cards | Required Controls (14,971), Fully Implemented (11,140), Control Gaps (3,831), Coverage (74.4%, colored by status). Only the Coverage card is colored, because only it has a threshold. |
| Heatmap matrix | 16 controls x 4 criticality tiers, cells colored by the coverage thresholds, sorted weakest control first |
| Bar chart | Coverage by NIST CSF 2.0 Function, bars colored by status |
| Bar chart | Coverage by NIST SP 800-53 family, bars colored by status |
| 4 slicers | Copied from Executive Summary and synced across both pages |

**What the page shows**
- Coverage falls steadily with criticality: Mission Critical 87.1% (Green), Business Critical 79.5% (Amber), Administrative 74.3% (Red), Non-Critical 64.7% (Red). The crown jewels meet target; the overall average is pulled down by low-tier assets.
- Weakest control: Configuration Baseline (Hardening) at 65.7%. Strongest: Backup & Recovery at 79.8%.
- Preventive controls (73.1%) trail Detective (75.7%) and Corrective (76.4%).
- Only five of the six CSF 2.0 Functions appear: there is no Govern control in the data.

### 6. Formatting and accessibility

- Segoe UI throughout, set once in the report theme.
- Color is reserved for KRI status. Plain counts and the criticality donut stay neutral.
- The palette was tested for readability rather than taste:
  - Amber text changed from `#F9A825` (2.0:1 contrast, below the WCAG minimum of 4.5:1) to `#A15C00` (5.2:1).
  - Heatmap fills changed to `#EE9A92` / `#F5C26B` / `#E6F2E7`, stepped by lightness so they remain distinguishable for viewers with red-green color vision deficiency (about 8% of men). The original red and green fills looked nearly identical under simulated deuteranopia.
- Heatmap uses white tile gridlines and no row banding; plain tables use light gray banding and horizontal gridlines only.

## Key results in the data (original thresholds)

| KRI | Value | Status |
|---|---|---|
| Controls Coverage % | 74.4% | Red |
| Controls EDR Coverage % (Servers) | 71.1% | Red |
| Reviews Overdue % | 14.5% | Red |
| Tests Failure Rate 90 Days % | 19.7% | Amber |
| Vulns Open Crit/High Past SLA | 67 | Red |
| Vulns SLA Compliance % | 70.2% | Red |
| Assets Internet-Facing Without Full MFA | 25 | Red |
| Assets Unsupported Platform | 48 | Red |
| Assets Crown Jewels With Control Gap % | 83.8% | Red |
| Exceptions Expired Not Remediated | 353 | Red |
| Exceptions Expiring 30 Days | 98 | Red |

## Decisions made along the way

1. **Star schema instead of a wide table**, so each control is a row, not a column.
2. **No fact-to-fact relationships**, to avoid ambiguous filter paths.
3. **Fixed snapshot date** (`Date As Of` = October 20, 2026) instead of `TODAY()`, so numbers stay reproducible.
4. **Test failure KRI uses the last 90 days**, because a KRI should reflect current performance, not two years of history.
5. **Coverage % by business unit instead of raw gap counts.** Payments Processing has the most gaps (626) but the best coverage (76.8%), simply because it has the most assets. A raw count would point at the wrong team.
6. **Original data and original thresholds kept.** A regenerated version with a red / amber / green mix and clustered asset compliance was evaluated, and the original data was chosen.
7. **Thresholds live only in the Status measures**, so a change in risk appetite means editing one formula, not every visual.
8. **Criticality sort column built in Power Query, not DAX.** A DAX calculated column that reads Criticality cannot be used to sort Criticality (circular dependency).
9. **Sort and code columns set to "Don't summarize"**, so they can never be accidentally added up in a visual.
10. **Heatmap thresholds match the KRI thresholds** (75% and 85%), so a cell's color means the same thing as a card's color.
11. **Short labels for SP 800-53 families**, because bar chart Y-axis labels do not wrap; the full name stays in the tooltip.

## Next steps

1. Testing & Review Schedule page: tester workload calendar, overdue work queue, pass/fail mix over time
2. Exceptions page
3. Vulnerabilities page
4. Asset drill-through page
5. Consider adding a Govern (GV) control so all six CSF 2.0 Functions are represented
6. Open decision: add a three-way `Compliance_Status` column to Dim_Asset (Compliant / Compliant with Exception / Non-Compliant). With the original data this shows 57 (4.8%) / 35 (2.9%) / 1,108 (92.3%).
