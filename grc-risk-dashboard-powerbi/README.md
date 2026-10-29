# GRC Risk Dashboard (Power BI)

An interactive Power BI dashboard that reports cybersecurity risk the way a Governance, Risk and Compliance (GRC) team would: control coverage mapped to NIST frameworks, control testing and review schedules, Key Risk Indicators (KRIs) with written thresholds, and financial risk quantification.

> **All data in this project is synthetic.** It was generated to model a fictional cybersecurity risk management company. No real organization, system or person is represented.

[Executive Summary](Exported-dashboards/ExecutiveSummary.pdf)

---

## Why this project

Most security dashboards report averages. Averages hide risk.

This dashboard is built to answer the questions a CISO or risk committee actually asks:

- **Where are our control gaps,** and are they on the systems that matter most?
- **Are we testing our controls on time,** and is assurance improving?
- **Which risks are outside our appetite,** measured against thresholds rather than opinion?
- **What is our risk worth in dollars,** and do our controls pay for themselves?

Every view responds to the same slicers (criticality, data classification, internet exposure, business unit), so any metric can be narrowed from all 1,200 assets to the handful of crown-jewel systems.

## Report pages

| Page | Question it answers | Status |
|---|---|---|
| Executive Summary | What is our overall cyber risk posture? | Complete |
| Control Coverage | Where are the gaps: which controls, which assets, which framework areas? | Complete |
| Testing & Review Schedule | Are we testing on time, what is coming, and what is most overdue? | Complete |
| Financial Risk | What is our expected annual loss, and what return do our controls deliver? | In progress |
| Security Operations | How fast do we detect and resolve incidents, and is phishing resilience improving? | In progress |
| Exceptions | Which policy exceptions are active, expiring or past their approved term? | Planned |
| Vulnerabilities | Which vulnerabilities are past SLA, and where? | Planned |

### Control Coverage

[Control Coverage](Exported-dashboards/control-coverage.pdf)

A heatmap of 16 security controls across four asset criticality tiers, with coverage by NIST CSF 2.0 Function and NIST SP 800-53 control family.

### Testing & Review Schedule

[Testing & Review Schedule](Exported-dashboards/testing-review.pdf)

Tester workload for the next six months, control test results by quarter, an overdue review queue sorted by age, and overdue rates by asset criticality.

## Key findings in the data

| Area | Finding |
|---|---|
| Control coverage | Overall coverage is 74.4%, but it falls steadily with criticality: 87.1% on Mission Critical assets to 64.7% on Non-Critical. The 22 Mission Critical, internet-facing systems sit at 88.9%. |
| Weakest areas | Configuration hardening is the weakest control (65.7%). Preventive controls (73.1%) trail Detective (75.7%) and Corrective (76.4%). |
| Testing | Control test failures fell from 22.7% to 18.9% over two years. 14.5% of reviews are overdue; Mission Critical assets have the lowest overdue rate (11.0%) but still miss the 10% target. |
| Financial risk | Residual annualized loss expectancy fell from $20.1M to $14.5M over eight quarters as coverage improved. Controls remove an estimated 78.5% of inherent risk for $9.9M a year. | 
<!-- 
| Detection | Incidents caught by EDR are detected in a median of 2.7 hours; incidents reported by third parties took 179.5 hours. | 
| Phishing | Simulated phishing click rate fell from 14.1% to 6.1%, while the report rate rose from 17.5% to 43.4%. | 
-->
## What is inside

### Data model

A star schema of **12 data tables** (3 dimensions and 9 facts) plus a calculated date table, with about 120,000 rows:

| Dimensions | Facts |
|---|---|
| Assets (1,200 servers and applications) | Asset-control implementation status (19,200) |
| Controls (16, mapped to NIST) | Control tests (47,859) |
| Business units (8) | Review schedule (35,566) |
| Date (calendar, built in DAX) | Policy exceptions (1,186) |
| | Vulnerabilities (6,097) |
| | Asset risk snapshots for ALE (9,600) | 
| | Control cost (32) | 
<!--
| | Security incidents (674) | 
| | Phishing simulations (176) | 
-->
Assets are classified by **criticality** (Mission Critical, Business Critical, Administrative, Non-Critical), **data classification** (PCI-DSS, SOX, PII, Administrative, Other, Non-Critical) and **internet exposure**.

### Framework alignment

- **NIST CSF 2.0:** every control is mapped to a Function, Category and Subcategories.
- **NIST SP 800-53 Rev. 5:** every control is mapped to a control family and specific controls.
- **Control type** (Preventive, Detective, Corrective) is kept separate from the CSF Function, because CSF 2.0 does not use those terms.

### Measures and KRIs

- 78 DAX measures (50 value measures plus 28 KRI status and color measures), using a consistent subject-first naming convention and grouped in display folders.
- **14 KRIs,** each with a value, a Green / Amber / Red status driven by a written threshold, and a color measure for conditional formatting. Thresholds live in one place, so a change in risk appetite means editing one formula.
**Not yet available**
- Financial quantification using **Annualized Loss Expectancy** (SLE x ARO) and **Return on Security Investment (ROSI)**, with control effectiveness linked directly to each asset's implementation data.

### Design choices

- Colors are reserved for KRI status, and the palette was tested for **WCAG contrast** and **red-green color vision deficiency**.
- Snapshot KRIs use a fixed as-of date (October 20, 2026), so every number is reproducible.
- Detection times use the **median**, so a few extreme incidents do not distort the typical case.

## Documentation

| Document | Contents |
|---|---|
| [Build instructions](docs/README_PowerBI_Build_Instructions.md) | Full rebuild guide: Power Query transforms, data types, the date table, all 22 relationships, every measure with its DAX and purpose, KRI thresholds, formatting standards, page layouts and verification values |
| [Quantitative metrics](docs/Quantitative_Metrics_README.md) | Financial risk and security operations tables: what each column means, how the data was generated, the ALE and ROSI method, and its assumptions and limitations |
| [Control Coverage page](docs/Control_Coverage_Summary.md) | How the page was built, issues encountered and fixed, expected values and notes for presenting |
| [Testing & Review page](docs/Testing_Review_Summary.md) | How the page was built, troubleshooting, expected values and notes for presenting |

## Repository structure

```
grc-risk-dashboard-powerbi/
├── README.md
├── docs/
│   ├── ExecutiveSummary_Summary.md
│   ├── Build_Instructions.md
│   ├── Control_Coverage_Summary.md
│   ├── Quantitative_Metrics_Summary.md
│   ├── Testing_Review_Summary.md
│   └── placeholder
└── Exported-dashboards/
    ├── All_Dashboards.pdf
    ├── All_Dashboards_MC_IF_Filter.pdf
    └── placeholder
```

The Power BI report (`.pbix`) and the dataset (`.xlsx`) are not published in this repository. **They are available on request.**

## Assumptions and limitations

- All data is synthetic; values illustrate the method, not a real environment.
- KRI thresholds are illustrative. In a real program they come from a risk appetite statement approved by leadership.
- Financial figures are estimates. Residual ALE assumes controls can remove at most 90% of risk; at assumptions from 60% to 100%, ROSI ranges from 2.6x to 4.9x, so the conclusion that controls pay for themselves holds across that range.
- Per-asset ALE is a simplification of scenario-based methods such as FAIR.
- Control effectiveness reflects implementation status, not tested effectiveness.

## Tools

Power BI Desktop · DAX · Power Query · Excel · Python (data generation)

## License

Copyright (c) 2026. All rights reserved. This work may be viewed for evaluation purposes. It may not be copied, modified or redistributed without permission.
****
