# Financial Risk Page: Summary

**Report:** GRC Risk Dashboard (Power BI)
**Data snapshot:** October 20, 2026 (synthetic data)
**Page built:** November 16, 2026

---

## Purpose

The other pages measure risk in counts and percentages. This page puts it in **dollars**, answering the questions a CFO or risk committee asks:

- What is our **expected annual loss** from cyber incidents, with and without our controls?
- **Where** is that loss concentrated: which business units and which assets?
- Is the risk **trending down**, and is it within our risk appetite?
- Do our controls **pay for themselves**?
- How much do these answers depend on our **assumptions**?

## Data used

| Table | What it provides |
|---|---|
| Fact_Asset_Risk_Quant | Each asset's value, exposure factor, annual rate of occurrence and control effectiveness, at 8 quarterly snapshots (2025-Q1 to the October 20, 2026 as-of date) |
| Fact_Control_Cost | Annual license and labor cost of each control |
| Dim_Asset, Dim_Business_Unit | Criticality, data classification, internet exposure and business unit for filtering |

**Method: Annualized Loss Expectancy (ALE)**
- Single Loss Expectancy (SLE) = Asset Value x Exposure Factor
- Inherent ALE (no controls) = SLE x Annual Rate of Occurrence
- Residual ALE (current controls) = Inherent ALE x (1 - Max Reduction x Control Effectiveness)
- ROSI (Return on Security Investment) = (Risk Reduction - Control Cost) / Control Cost

Control effectiveness at the current snapshot is each asset's average implementation score from the control coverage data, so **the dollar figures move when control coverage moves**.

## Headline figures (no filters, assumption at 0.9)

| Measure | Value |
|---|---|
| Inherent expected loss | $67.4M |
| Residual expected loss | $14.5M (Amber against a $12M / $18M appetite) |
| Risk removed by controls | 78.5% |
| Annual control cost | $9.9M |
| Return on security investment | 4.3x |

## What the data shows

**Risk is falling.** Residual expected loss dropped every quarter as control coverage improved, and crossed below the $18M red threshold in 2025-Q4:

| Quarter | Residual ALE |
|---|---|
| 2025-Q1 | $20.1M |
| 2025-Q2 | $19.1M |
| 2025-Q3 | $18.4M |
| 2025-Q4 | $17.6M |
| 2026-Q1 | $16.8M |
| 2026-Q2 | $16.1M |
| 2026-Q3 | $15.2M |
| 2026-Q4 (Oct 20) | $14.5M |

**Risk is concentrated.**
- **Payments Processing** carries $5.3M (37% of the total), followed by Client Risk Platform ($3.0M) and Finance & Accounting ($2.0M). Corporate IT and Security Operations carry about $0.2M each.
- The **22 Mission Critical, internet-facing assets (1.8% of all assets) carry $2.7M (19%)** of residual expected loss.
- Seven of the top 10 assets by residual loss belong to Payments Processing.

**Good controls do not make valuable assets low-risk.** Payments Gateway Core is 96% effective and still ranks fourth by residual loss, because its value is so high. Risk that controls cannot remove is a candidate for **risk transfer**, such as cyber insurance.

**The conclusion survives the assumption.** The model assumes controls can remove at most 90% of risk. Testing that assumption with the slider:

| Assumed max reduction | Residual ALE | KRI status | ROSI |
|---|---|---|---|
| 0.6 | $32.1M | Red | 2.6x |
| 0.8 | $20.4M | Red | 3.8x |
| 0.9 | $14.5M | Amber | 4.3x |
| 1.0 | $8.6M | Green | 4.9x |

The KRI color depends on the assumption; the conclusion that controls are worth far more than they cost does not.

## Notes for presenting

- Lead with the trend: residual expected loss fell 28% in seven quarters.
- Use the 22-asset finding to justify the crown-jewel focus.
- Say the 0.9 assumption out loud, and use the slider to show that ROSI stays at about 2x or higher across the whole range (2.0x at 0.5, 4.9x at 1.0).
- **Residual ALE status is shown only for the whole program.** The $12M / $18M appetite applies to total risk, so with any asset filter selected, the card shows its value without a status color. A filtered $2.7M is not "within appetite"; it is simply a smaller number.
- All figures are estimates from synthetic data and illustrate the method, not a real environment.
