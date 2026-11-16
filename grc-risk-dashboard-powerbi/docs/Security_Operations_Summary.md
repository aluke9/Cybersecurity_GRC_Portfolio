# Security Operations Page: Summary

**Report:** GRC Risk Dashboard (Power BI)
**Data snapshot:** October 20, 2026 (synthetic data)
**Page built:** November 16, 2026

---

## Purpose

Shows how the security program performs day to day, answering:

- **How many incidents** do we have, and how severe are they?
- **How fast** do we detect them, and **what** detects them: our controls or people?
- Are employees getting **better at spotting phishing**, and which business units need help?

The page links operations back to control coverage: incidents on assets with working SIEM and EDR controls are found much faster.

## Data used

| Table | What it provides |
|---|---|
| Fact_Security_Incidents | 674 incidents (January 2025 to October 20, 2026): type, severity, detection source, detecting control, hours to detect, contain and resolve, and financial impact |
| Fact_Phishing_Simulation | 176 monthly phishing simulation results (22 months x 8 business units): emails sent, clicks, credentials submitted, reports |
| Dim_Asset, Dim_Business_Unit, Dim_Date | Filters by asset attributes, business unit and time |

How the incident data links to the rest of the model: the detection source of each incident depends on whether that asset's SIEM, EDR or IDS/IPS control is actually implemented in the control coverage data.

## Headline figures (no filters)

| Measure | Value | Status |
|---|---|---|
| Incidents | 674 | |
| High / Critical incidents | 73 (62 High, 11 Critical) | |
| Median hours to detect (last 90 days) | 7.7 | Green (target 8 hours or less) |
| Incidents detected by a control | 62.0% | |
| Phishing click rate (last campaign) | 6.1% | Amber (target 5% or less) |

## What the data shows

**Controls find incidents far faster than people.**

| Detection source | Incidents | Median hours to detect |
|---|---|---|
| EDR Alert | 85 | 2.7 |
| IDS/IPS Alert | 98 | 4.2 |
| SIEM Alert | 235 | 5.4 |
| User Report | 191 | 25.5 |
| Third-Party Notification | 65 | 179.5 |

Incidents caught by EDR were detected about 66 times faster than those reported by a third party. The 38% of incidents that no control caught make the case for raising SIEM and EDR coverage.

**Incident volume is steady, with a slight decline.** 103, 93, 93 and 104 incidents per quarter in 2025; 88, 81 and 94 in 2026-Q1 to Q3. (2026-Q4 covers only October 1 to 20 and is excluded from the trend.)

**Phishing resilience has improved substantially.**
- The click rate fell from **14.1%** (January 2025) to **6.1%** (October 2026).
- The report rate rose from **17.5%** to **43.4%**. People are not just avoiding the bait; they are actively helping detection.
- In the latest campaign, six business units are amber (5.4% to 8.0%) and two are green: Corporate IT (3.7%) and Security Operations (1.1%).

**Payments Processing is the priority in both views.** It has the highest phishing click rate (8.0%) and the largest residual expected loss on the Financial Risk page ($5.3M): two independent data sources pointing to the same targeted training recommendation.

<!-- 
## Notes for presenting

- Lead with the detection-source chart; it is the clearest link between control investment and operational results.
- Median, not average, is used for detection time. A handful of third-party reports took around 180 hours and would pull the average up to 31 hours, while the // typical incident took 7.8.
- Phishing results filter by **business unit and date only**, because phishing simulations are not tied to assets. Criticality, data classification and internet // -facing filters do not affect them.
- All figures are from synthetic data and illustrate the method, not a real environment.
-->
