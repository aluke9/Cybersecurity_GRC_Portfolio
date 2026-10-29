# Control Coverage Page: Summary

**Report:** Cyber Risk Management Dashboard (Power BI)
**Data snapshot:** October 20, 2026 (fictional data)
**Page built:** October 22, 2026

---

## Purpose

The Executive Summary shows overall control coverage at 74.4%. That is an average, and averages hide where the problem is. This page answers **"where are the gaps?"**: which controls, on which kinds of assets, and in which framework areas.

## Layout (1280 x 720 canvas)

```
| Slicers | Card | Card | Card | Card (Coverage)              |
|         | Heatmap: Control x Criticality | CSF 2.0 Function   |
|         |                                | SP 800-53 Family   |
```

| Visual | Fields | X | Y | W | H |
|---|---|---|---|---|---|
| Page title | "Control Coverage" | 16 | 12 | 900 | 40 |
| 4 slicers (synced) | Criticality, Data Classification, Internet-Facing, Business Unit | 16 | 64 / 136 / 208 / 280 | 170 | 60 |
| Card 1 | Controls Applicable, label "Required Controls" | 202 | 64 | 256 | 105 |
| Card 2 | Controls Implemented, label "Fully Implemented" | 470 | 64 | 256 | 105 |
| Card 3 | Controls Gaps, label "Control Gaps" | 738 | 64 | 256 | 105 |
| Card 4 | Controls Coverage %, label "Coverage", colored by status | 1006 | 64 | 256 | 105 |
| Heatmap (matrix) | Rows: Control_Name; Columns: Criticality; Values: Controls Coverage % | 202 | 181 | 620 | 523 |
| Bar chart | Coverage by CSF_Function | 834 | 181 | 430 | 255 |
| Bar chart | Coverage by SP 800-53 family | 834 | 448 | 430 | 256 |

## Steps completed

### Step 0: Sort criticality by importance
- **Goal:** show criticality as Mission Critical, Business Critical, Administrative, Non-Critical instead of alphabetical order.
- **First attempt (failed):** a DAX calculated column. Using it to sort Criticality caused the error *"A circular dependency was detected"*, because a DAX column that reads Criticality cannot also be used to sort Criticality.
- **Fix:** built `Criticality_Sort` in **Power Query** instead (Add Column > Conditional Column: Mission Critical = 1, Business Critical = 2, Administrative = 3, else 4), typed as Whole Number.
- Set **Criticality > Sort by column > Criticality_Sort**, and hid the sort column.
- Set Summarization to **Don't summarize** on `Criticality_Sort`, `Month Sort` and `Year`, so identifier columns can never be added up in a visual.
- Side benefit: the donut and slicer on the Executive Summary now show criticality in the right order too.

### Step 1: New page and synced slicers
- Added the page `Control Coverage`.
- Copied the four slicers from the Executive Summary and chose **Sync** when pasting, so a filter on one page applies to both.
- Copied the page title text box.

### Step 2: Four cards
- Copied an existing card for each to keep formatting identical.
- **Only the Coverage card is colored**, because it is the only one with a KRI threshold. Color on the dashboard always means KRI status; coloring "Control Gaps" red would imply a failing status that no threshold defines.
- Display units set to **None** on the three count cards.
- Built-in check: Required (14,971) minus Implemented (11,140) equals Gaps (3,831).

### Step 3: Control x Criticality heatmap
- Matrix with Control_Name on rows, Criticality on columns, Controls Coverage % as values (renamed "Coverage").
- Sorted by Coverage ascending, which sorts by the Total column and puts the weakest control on top.
- Cell background colored with **rules** matching the KRI thresholds (75% and 85%), applied to values and totals.
- **Gotcha:** every rule value box had to be switched from **Percent** to **Number**. In rules, "Percent" means a percentile of the range, not the value itself.

### Step 3a: Color and gridline review (accessibility)
The palette was tested for readability rather than taste:

| Issue | Finding | Change |
|---|---|---|
| Heatmap fills | Original light red (#F4C7C3) and light green (#C8E6C9) had nearly the same lightness and looked alike under simulated deuteranopia (red-green color vision deficiency affects about 8% of men) | New fills stepped by lightness: Red `#EE9A92`, Amber `#F5C26B`, Green `#E6F2E7`. Problem cells are darkest, so they draw the eye first |
| Amber text | `#F9A825` on white measured 2.0:1 contrast, below the WCAG AA minimum of 4.5:1 | Changed to `#A15C00` (5.2:1) in all 11 Color measures |
| Red and green text | `#C62828` (5.6:1) and `#2E7D32` (5.1:1) | No change needed |

Gridlines and banding:

| Setting | Heatmap | Plain tables |
|---|---|---|
| Alternate background | `#FFFFFF` (banding off) | `#F7F7F7` |
| Horizontal gridlines | `#FFFFFF`, width 2 | `#E1E1E1`, width 1 |
| Vertical gridlines | `#FFFFFF`, width 2 | Off |

White gridlines turn the heatmap into separated tiles. Banding was turned off on the heatmap because alternating row colors compete with the status colors.

### Step 4: Coverage by NIST CSF 2.0 Function
- Clustered bar chart, CSF_Function on the Y-axis, Controls Coverage % on the X-axis, sorted ascending (smallest bar on top).
- Bars colored by **Controls Coverage % Color** (Field value).
- Data labels on, X-axis off (labels already show the values), Y-axis title off.

### Step 5: Coverage by NIST SP 800-53 family
- Copied the Step 4 chart and swapped the Y-axis field.
- **Problem:** long family names were cut off. Bar chart Y-axis labels do not wrap.
- **Options offered:** raise Y-axis > Values > Maximum width, or add a shorter label column `SP800_53_Family_Short` in Power Query (Duplicate Column + Replace Values) with the full name in the tooltip.

| Full name | Short label |
|---|---|
| System & Communications Protection | Sys & Comms Protection |
| System & Information Integrity | Sys & Info Integrity |
| Identification & Authentication | Identification & Auth |

### Step 6: Test
- Selected Internet-Facing = Yes: every card, the heatmap and both charts changed, and coverage rose. Passed.

## Expected values (no filters)

| Check | Value |
|---|---|
| Cards | 14,971 / 11,140 / 3,831 / 74.4% (Red) |
| Mission Critical selected | 2,118 / 1,844 / 274 / 87.1% (Green) |
| Coverage by criticality | Mission Critical 87.1% (Green), Business Critical 79.5% (Amber), Administrative 74.3% (Red), Non-Critical 64.7% (Red) |
| Heatmap top row | Configuration Baseline (Hardening), 65.7% |
| Heatmap bottom row | Backup & Recovery, 79.8% |
| DLP x Non-Critical | Blank (correct: DLP applies only to PCI-DSS, PII and SOX data, which no Non-Critical asset holds) |
| CSF Functions | Protect 73.2% (Red), Respond 74.9% (Red), Detect 75.0% (Amber), Identify 77.4% (Amber), Recover 79.8% (Amber) |
| SP 800-53 families | Configuration Management 65.7% lowest, Contingency Planning 79.8% highest; six Red, three Amber |

## Corrections made along the way
- Controls Implemented is **11,140** and Controls Coverage % is **74.4%**. Earlier guidance said 11,127 and 74.3%, figures taken from a draft of the data; Power BI's numbers were correct.

## Notes for presenting
- **Crown jewels meet target; the average is pulled down by low-tier assets.** Coverage falls steadily from 87.1% (Mission Critical) to 64.7% (Non-Critical).
- **Hard threshold edges:** Detect (75.02%) shows Amber next to Respond (74.91%) in Red. The colors are correct; focus on the pattern, not one decimal.
- **Govern is missing:** only five of the six CSF 2.0 Functions appear, because all 16 controls are technical. Either explain that governance is tracked in the policy program, or add a governance control to the data.
- **SP 800-53 families:** six of the nine contain a single control, so most family bars repeat a heatmap row. Each control is counted under one primary family, although several map to more than one (for example, DLP maps to AC-4, SC-7(10) and SI-4(4) but is counted under Access Control).
- **Prevention is weakest:** Preventive controls 73.1%, against Detective 75.7% and Corrective 76.4%. The program detects and recovers better than it prevents.
