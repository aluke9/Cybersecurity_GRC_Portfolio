# Testing & Review Schedule Page: Summary

**Report:** Cyber Risk Management Dashboard (Power BI)
**Data snapshot:** October 20, 2026 (fictional data)
**Page built:** October 23, 2026

---

## Purpose

Answers **"are we testing on time, what's coming, and what's most overdue?"**

The obvious question ("which tester is overloaded?") turned out not to be the story: upcoming workload and overdue reviews are spread almost evenly across all eight testers (250 to 291 overdue each). The page therefore focuses on **team-wide capacity** and on **whether the right assets are being prioritized**.

## Layout (1280 x 720 canvas)

```
| Slicers | Card | Card | Card | Card | Card                         |
|         | Tester workload (next 6 months) | Pass / fail mix by quarter |
|         | Overdue work queue (table)        | Overdue % by criticality  |
```

| Visual | Fields | X | Y | W | H |
|---|---|---|---|---|---|
| Page title | "Testing & Review Schedule" | 16 | 12 | 900 | 40 |
| 4 slicers (synced) | Criticality, Data Classification, Internet-Facing, Business Unit | 16 | 64 / 136 / 208 / 280 | 170 | 60 |
| Card 1 | Reviews Overdue %, label "Reviews Overdue", colored | 202 | 64 | 202 | 105 |
| Card 2 | Reviews Overdue, label "Overdue Reviews" | 416 | 64 | 202 | 105 |
| Card 3 | Reviews Overdue Avg Days, label "Avg Days Overdue" | 630 | 64 | 202 | 105 |
| Card 4 | Reviews Due Next 30 Days, label "Due Next 30 Days" | 844 | 64 | 202 | 105 |
| Card 5 | Tests Failure Rate 90 Days %, label "Test Failures (90d)", colored | 1058 | 64 | 202 | 105 |
| Matrix | Rows: Assigned_Tester; Columns: Dim_Date[Month]; Values: Reviews Upcoming | 202 | 181 | 520 | 255 |
| 100% stacked column | X: Year Quarter; Y: Tests Performed; Legend: Result | 734 | 181 | 530 | 255 |
| Table | Overdue work queue | 202 | 448 | 700 | 256 |
| Bar chart | Reviews Overdue % by Criticality | 914 | 448 | 350 | 256 |

## New measures

| Measure | DAX | Purpose |
|---|---|---|
| Reviews Upcoming | `CALCULATE ( COUNTROWS ( Fact_Review_Schedule ), Fact_Review_Schedule[Review_Status] <> "Overdue" )` | Scheduled and in-progress reviews. Unlike Reviews Overdue, it keeps the date filter, so it can be spread across months. |
| Reviews Overdue Avg Days | `CALCULATE ( AVERAGE ( Fact_Review_Schedule[Days_Overdue] ), Fact_Review_Schedule[Review_Status] = "Overdue", REMOVEFILTERS ( Dim_Date ) )` | How late overdue reviews are on average. A count says how many; this says how bad. |

**Formatting:** Reviews Overdue Avg Days set to **0 decimals** (189, not 188.5). Half a day changes no decision, and the extra digit suggests false precision. The same applies to Vulns MTTR Critical (days).
- Measure formatting lives on the **Measure tools** tab, which only appears while the measure itself is selected in the Data pane. Alternative: Model view > Properties > Formatting > Decimal places.

## Steps completed

### Step 1: Page and slicers
Added the page `Testing & Review`, pasted and synced the four slicers, and copied the title text box.

### Step 2: New measures
Created the two measures above.

### Step 3: Five cards
Only the two cards with KRI thresholds are colored (Reviews Overdue % and Tests Failure Rate 90 Days %). The three count cards stay neutral, with Display units set to None.

### Step 4: Tester workload matrix
Shows upcoming reviews per tester per month for the next six months (Oct 2026 to Apr 2027).

**Troubleshooting on this step:**

| Symptom | Cause | Fix |
|---|---|---|
| Date filter returned no data | Typed dates are read in the computer's regional format; also the filter had been added under "Filters on this page" instead of "Filters on this visual" | Removed the page-level filter. Used a locale-proof visual filter on **Dim_Date[Month Sort]**: >= 202610 and <= 202704 (Data pane > ... > View hidden to reach the hidden column) |
| Matrix showed one blank column with each tester's full total (4,052, 4,346 ...) | The blank column is Power BI's "(Blank)" member: no review matched a Dim_Date row. The relationship had been created on **Fact_Review_Schedule[Asset_ID]** to Dim_Date[Date] | Edited the relationship to **Fact_Review_Schedule[Due_Date]** to Dim_Date[Date] |
| Columns well held **Date** (one column per day) | Wrong field | Replaced with **Dim_Date[Month]** |
| Date columns showed as hierarchies | Power BI's Auto date/time feature | Recommended turning it off: File > Options and settings > Options > Current File > Data Load > untick Auto date/time |
| "Manage relationships" not on the Home tab | In Report view it sits on the **Modeling** tab; on Home only in Model view | Used Modeling tab, or Model view (click the relationship line to see which columns it joins) |

Lesson: the Testing page was the first visual to use the Fact_Review_Schedule date relationship, which is why the mistake surfaced here. The Exceptions and Vulnerabilities date relationships should be checked the same way before building those pages.

No conditional color on the workload matrix: workload is not a status.

### Step 5: Pass / fail mix by quarter
100% stacked column chart of Tests Performed by Year Quarter, split by Result.

| Result | Color | Reason |
|---|---|---|
| Fail | `#C62828` | Status red |
| Pass with Exceptions | `#F5C26B` | Status amber |
| Pass | `#1B5E20` (dark forest green), white data labels | Requested forest green |

- Standard forest green (`#228B22`) was rejected: under simulated deuteranopia it and the Fail red become almost the same olive brown, so color-blind viewers could not tell passes from failures.
- `#1B5E20` stays clearly darker than the red (lightness 34 vs 49) and keeps them distinguishable. White labels on it measure 7.9:1 contrast.
- Lighter alternative if the dark green feels heavy: sage `#7BA77A`.
- Data labels act as a second safety net, since the percentages identify each segment regardless of color.

### Step 6: Overdue work queue
- Table: Asset_Name, Criticality, Control_Name, Assigned_Tester, Due_Date, Days Overdue.
- Visual filter: Review_Status = Overdue. Sorted by Days Overdue, descending.
- Neutral data bars on Days Overdue; plain-table formatting.
- If Due_Date splits into a hierarchy, choose **Due_Date** instead of **Date Hierarchy** in the field's dropdown.

### Step 7: Overdue % by criticality
Clustered bar chart of Reviews Overdue % by Criticality, bars colored by Reviews Overdue % Color.

## Expected values (no filters)

| Check | Value |
|---|---|
| Card 1: Reviews Overdue % | 14.5% (Red) |
| Card 2: Overdue Reviews | 2,172 |
| Card 3: Avg Days Overdue | 189 |
| Card 4: Due Next 30 Days | 2,132 |
| Card 5: Test Failures (90d) | 19.7% (Amber) |
| Workload matrix | About 230 to 330 reviews per tester per month; Oct 2026 total 695 (covers Oct 20 to 31 only); grand total 14,595 |
| Pass / fail mix | Fail falls from 22.7% (2025-Q1) to 18.9% (2026-Q4); Pass rises from 69.6% to 74.4%; Pass with Exceptions about 7 to 8% |
| Work queue | 2,172 rows; top row fin-app-prd-003, Administrative, Patch Management, Internal Audit - F. Laurent, due 2025-06-28, 479 days overdue |
| Overdue % by criticality | Mission Critical 11.0%, Non-Critical 14.6%, Administrative 14.9%, Business Critical 15.8% (all Red, above the 10% threshold) |

## Notes for presenting
- **Prioritization is partly working:** Mission Critical has the lowest overdue rate (11.0% against about 15% elsewhere), but it is still above the 10% threshold, and Business Critical is the worst.
- **Capacity, not individuals:** workload and overdue counts are nearly identical across the eight testers, so the fix is team capacity or testing scope, not one person's performance.
- **The oldest item is 479 days late:** a review more than a year overdue means that control is effectively untested.
- **Be ready for a realism question:** this data has eight testers handling about 2,300 reviews a month, roughly 14 each per working day. Real programs make that workable through **sampling** (testing a sample of assets per control) or **automated control testing**.
