# Security Control Testing — Workbook Navigation Guide

## Purpose

Use this guide to understand the V3 assessment workbook and the generated monitoring report, how their identifiers relate, and where to find requirements versus evidence versus conclusions.

This is an independent incident-informed simulation. Synthetic records, thresholds, identities, and results are not recovered Okta evidence or findings about Okta's actual controls. Preserve the original evidence. Do not mix V1/V2 records with V3.

## 1. Which Excel workbook am I using?

| Workbook | Purpose | What it does not do |
|---|---|---|
| `Support_Control_Assessment_v3.xlsx` or its fresh replacement copy | Defines the exercise, control criteria, evidence references, and blank assessment workpapers | Does not contain an answer key or completed assessor conclusions |
| `Monitoring_Evidence_Review.xlsx` | Presents calculated DET-01 and DET-02 event/receipt/alert results and supporting reference tables | Does not complete all SUP-03 requirements or all five core controls |

```text
Original V3 assessment workbook
├── What is in scope?
├── What requirements apply?
├── What evidence is provided?
└── Where do I document my assessment?

Generated monitoring report
├── Which source events should trigger monitoring?
├── Did those events reach the collector?
├── Are matching alerts present and timely?
└── Which records need review?
```

## 2. Identifier glossary

These are exercise-specific labels, not official Okta control numbers. The expansions below are practical shorthand based on how the prefixes are used, not a separately defined formal acronym standard.

| Identifier | Meaning in this exercise | Example |
|---|---|---|
| `SUP` | Support-system core control area | `SUP-03`: logging, exports, investigation scope, and monitoring |
| `DET` | Detection rule | `DET-02`: file-download threshold monitoring |
| `EXT` | Optional extension control | `EXT-01`: administrator authentication and helpdesk verification |
| `DTEST` | Approved detection-test reference | `DTEST-002`: case-view detection retest |
| `AUTH` | Authentication event ID | `AUTH-00452` |
| `FILE` | File-access event ID, not an attachment ID | `FILE-00560` |
| `ING` | Collector receipt ID | `ING-01014` |
| `ALERT` | Alert record ID | `ALERT-002` |
| `ATT` | Attachment ID | `ATT-0001` |

The numeric suffix identifies a record or rule; it is not a severity or effectiveness score.

## 3. Control hierarchy

```text
Core assessment
├── SUP-01 — Service credential protection and managed browsers
├── SUP-02 — Exposure assessment: files, sessions, and reports
├── SUP-03 — Logging, exports, investigation scope, and monitoring
│   ├── Source-event and collector-receipt reconciliation
│   ├── Vendor-export coverage and replacement reconciliation
│   ├── Investigation-query coverage
│   └── Detection rules
│       ├── DET-01 — Successful interactive service-account login
│       └── DET-02 — Fifth successful file access per account
│                    within a rolling 60-minute window;
│                    reset the count after a trigger
├── SUP-04 — Response, containment, and revised scope
└── SUP-05 — Staged session binding: provider and customer controls

Optional late-assessment extensions
├── EXT-01 — Admin MFA, phishing resistance, and helpdesk verification
├── EXT-02 — Support access provisioning and retention
├── EXT-03 — Impact notification and independent investigation closure
└── EXT-04 — Late administrative guardrails and timeouts
```

A successful DET-02 test supports one component of SUP-03. It does not establish that SUP-03 as a whole is effective.

## 4. Where to find the exercise requirements

In the original V3 workbook, open `Control Requirements`.

- Column A: control ID.
- Column F: `Fictional testing criterion` — the requirement being evaluated.
- Column G: `Procedure / limitation`.
- Column H: evidence references.
- For SUP-03 specifically: row 4; criterion cell `F4`.

### SUP-03 criterion — original wording

> Cover both file event types and identifier semantics; validate export coverage and reconcile replacements. IOC queries must search both types. Fictional SLAs: SIEM receipt within 15 minutes and alert within 5 minutes of qualifying receipt. DET-01 successful interactive service login; DET-02 fifth successful file download/account/rolling 60 minutes, reset count after trigger. Both routes required.

### Monitoring requirements in plain language

| Requirement | Interpretation |
|---|---|
| Both file routes required | Include `case_attachment.read` and `file_library.download` |
| DET-01 qualifying condition | Successful interactive authentication by a service account |
| DET-02 grouping | Count successful file-access events by account across both routes |
| DET-02 threshold | The fifth qualifying event within a rolling 60-minute window |
| Reset after trigger | The next event begins the next count; the triggering event is not reused |
| Collector receipt | Within 15 minutes / 900 seconds of the source event |
| Alert generation | Within 5 minutes / 300 seconds of the qualifying receipt |

Implementation convention: the current reconstruction includes an event exactly 60 minutes before the current event. This boundary convention is an analysis assumption, not an additional sentence in the original criterion. Distinct source events are counted, not distinct attachments.

## 5. Original V3 workbook — all worksheets

| Worksheet | Purpose |
|---|---|
| `Assessment Brief` | Purpose, scope, evidence periods, assessment approach, and assurance limits. |
| `Boundaries` | Included and excluded systems, populations, incidents, and attribution limits. |
| `Sources` | References for public claims and limits on how each source can be used. |
| `Public Timeline` | Public incident milestones; not a recovered internal activity log. |
| `Control Requirements` | Control IDs, owners, populations, fictional criteria, procedures, limitations, and evidence references. |
| `Populations` | Declared populations, counting units, keys, and cautions about double counting. |
| `Accounts` | Account-inventory snapshots; distinguish snapshot rows from unique accounts. |
| `Permissions` | Permissions by account and snapshot; match the applicable date. |
| `Attachments` | Attachment metadata, case/file identifiers, versions, sizes, and dates. |
| `Changes` | Recorded changes and claimed scope; closed status is not proof of effectiveness. |
| `Incidents` | Reported, acknowledged, escalated, confirmed, and closure timestamps. |
| `Environment Registry` | Customer/tenant/support-environment relationships and system boundaries. |
| `Vendor Exports` | Export request and delivery information and declared coverage; corroborate completeness. |
| `Investigation Queries` | Recorded query scope, filters, export references, and declared result counts. |
| `Tenant Configuration` | Point-in-time feature settings and session limits; not continuous enforcement proof. |
| `Admin Policies` | Point-in-time administrator authentication-policy evidence. |
| `Notifications` | Recorded customer/environment notification and delivery evidence. |
| `Evidence Index` | Evidence references, formats, declared counts, and hashes. |
| `Blank Core Workpaper` | Your assessment entries for each core control and phase. |
| `Blank Extension Workpaper` | Your assessment entries for optional extension controls. |
| `Blank Findings` | Document condition, criterion, evidence, risk, cause confidence, recommendations, and retest requirements. |
| `Executive Summary` | Your high-level assessment, residual risk, closure decisions, and limitations. |
| `Join Guide` | Relationships between files and identifiers; consult before combining tables. |
| `Revision Notes` | V3 changes and supersession of earlier versions. |
| `Data Dictionary` | Field definitions, types, and interpretation cautions. |

## 6. Core control reference

The text below reproduces each core control's criterion and procedure/limitation from the original workbook. Read the original workbook for full context.

### SUP-01 — Service credential protection and managed browsers

- Owner: Support owner.
- Population: Service accounts and managed devices.

Criterion:

> Personal Google profile sign-in must be blocked on managed support devices; service credentials must not be stored in personal profiles. Disable identified compromised account and terminate existing support sessions. Password sync prevention is supplemental fictional policy, not independently stated Okta remediation.

Procedure / limitation:

> Compare snapshots and endpoint examination; corroborate actions and verification; distinguish unverified exposure avenue from confirmed credential storage.

Evidence references:

`account_inventory.csv; permissions.csv; browser_configuration.csv; endpoint_examinations.csv; containment_actions.csv; authentication.jsonl; containment_verification.jsonl`

### SUP-02 — Exposure assessment: files, sessions and reports

- Owner: Support incident team.
- Population: Files, customer sessions, support users, additional contacts.

Criterion:

> Identify customers/files/tokens exposed using both file-event routes and published versions. Reconstruct report filters and reconcile bytes and row counts. Assess all support-user report scope separately from file exposure. Additional contacts do not establish compromise of separate CIC support systems.

Procedure / limitation:

> Reconcile distinct populations and report membership; distinguish accessed files, exposed names/emails, token exposure and observed session reuse. Do not extrapolate actual Okta counts from scaled data.

Evidence references:

`attachment_records.csv; file_record_mapping.csv; attachment_inspections.csv; token_session_mapping.csv; support_user_population.csv; additional_contact_population.csv; report_population_membership.csv; report_reconstructions.csv; attachment_access.jsonl; report_access.jsonl; customer_session_activity.jsonl`

### SUP-03 — Logging, exports, investigation scope and monitoring

- Owner: Support owner / SOC.
- Population: Source auth/file events, exports, queries and rules.

Criterion:

> Cover both file event types and identifier semantics; validate export coverage and reconcile replacements. IOC queries must search both types. Fictional SLAs: SIEM receipt within 15 minutes and alert within 5 minutes of qualifying receipt. DET-01 successful interactive service login; DET-02 fifth successful file download/account/rolling 60 minutes, reset count after trigger. Both routes required.

Procedure / limitation:

> Reproduce queries on specific export membership; reconcile unique source events and receipts. Distinguish recorded activity, available export and investigated records. Do not call every source-IP or older-browser match malicious.

Evidence references:

`vendor_export_manifests.csv; export_event_membership.csv; investigation_queries.csv; investigation_query_results.csv; analyst_notes.csv; customer_iocs.csv; detection_configuration.csv; detection_tests.csv; authentication.jsonl; attachment_access.jsonl; ingestion_events.jsonl; security_alerts.jsonl`

### SUP-04 — Response, containment and revised scope

- Owner: Support IR and customer administrators.
- Population: Customer reports and relevant containment actions.

Criterion:

> Fictional SLAs: acknowledge within 30 minutes; escalate within 60 minutes. Disable/terminate support account sessions within 60 minutes of confirmed compromise. Revoke exposed customer sessions within 60 minutes of documented token identification. Reconcile supplemental exports and verify actions before declaring closure.

Procedure / limitation:

> Evaluate escalation and investigative correctness separately. Token identification times must be evidenced; a mapping delivered later is not proof it was known earlier. Record insufficient evidence where appropriate.

Evidence references:

`incident_tickets.csv; analyst_notes.csv; investigation_queries.csv; attachment_inspections.csv; containment_actions.csv; containment_verification.jsonl; customer_session_activity.jsonl`

### SUP-05 — Session binding: staged provider and customer controls

- Owner: Provider release owner AND tenant administrator.
- Population: 12 main customer tenants; early and late tests.

Criterion:

> Early phase: different-ASN administrator session reuse requires reauthentication for enabled tenants; provider availability separate from adoption. Late phase: enabled IP binding requires reauthentication upon source-IP change, including same-ASN change. Universal adoption is fictional exercise requirement. No retrospective late-feature requirement.

Procedure / limitation:

> Compare configurations, releases and paired tests; assess same-ASN/different-ASN early vs late separately. Pre-availability adoption N/A; absence of configuration is not proof of nonexistent feature.

Evidence references:

`feature_release.csv; tenant_configuration.csv; session_binding_tests.csv; customer_session_activity.jsonl`

## 7. Generated monitoring report — worksheet guide

| Worksheet | What it shows | How to use it |
|---|---|---|
| `Read Me` | Scope, thresholds, assumptions, limitations | Read before interpreting results |
| `Results by Period` | Rule/period summary counts | Separate historical activity from approved retests |
| `DET-01 Event Results` | Qualifying logins, snapshots, receipts, alerts, and timing | Inspect authentication-monitoring operation |
| `DET-02 Trigger Results` | Reconstructed file triggers, receipts, alerts, and timing | Inspect expected versus observed monitoring outcomes |
| `DET-02 Contributing Events` | Five source events behind each trigger | Manually challenge the rolling-window reconstruction |
| `Exceptions` | Receipt or alert checks requiring review | Develop observations; do not assume every exception is an attack |
| `Monitoring Configuration` | Supplied configuration records | Compare documented rule coverage against criteria |
| `Approved Test Records` | Supplied detection-test records | Corroborate test IDs, trigger IDs, approval references, and alert IDs |
| `Raw Export Manifests` | Supplied export metadata | Reference only; export completeness not yet assessed by reporter |
| `Raw Export Membership` | Supplied export/event relationships | Reference only; reconcile overlaps before counting activity |
| `Raw Investigation Queries` | Supplied query definitions | Reference only; query correctness not yet assessed by reporter |
| `Raw Query Results` | Supplied query/event relationships | Reference only; these rows are not new source activity |

### Summary-column glossary

| Column | Meaning | Caution |
|---|---|---|
| `expected_triggers` | Qualifying DET-01 events or independently reconstructed DET-02 triggers | Not proof the actual system recognized a trigger |
| `timely_consistent_receipts` | Matched receipts marked received, with consistent source time and delay from 0 to 900 seconds | Scoped to the tested events, not automatically all source records |
| `matching_alerts` | Alerts matched by source-event reference within the relevant rule | Matching alone does not establish correct timing |
| `alerts_within_300_seconds` | Matched alerts generated from 0 to 300 seconds after receipt | Assess receipt integrity separately |
| `events_with_test_reference` | Qualifying events with populated test references | Reference presence alone is not an independent approval check |
| `missing_alerts` | Expected triggers without matching alerts in supplied evidence | Does not prove an alert never existed elsewhere |

### Reading a result label

```text
Receipt and alert checks met
└── This row met the reporter's receipt and timing checks.
    It is not a conclusion that the whole control is effective.

No matching alert in supplied evidence
└── The expected trigger did not match an alert in the supplied file.
    It is an evidence-backed exception, not proof of an attack or root cause.
```

### File-access fields that are easy to confuse

| Field | Case-view example | File-library example |
|---|---|---|
| `event_type` | `case_attachment.read` | `file_library.download` |
| `route` | `case_view` | `file_library` |
| `record_type` | `support_case` | `support_file` |
| `record_id` | A case ID | A file-record ID |

An event ID identifies an action; an attachment ID identifies the file. Multiple legitimate event records may refer to the same attachment.

## 8. Results observed at the October 27, 2026 checkpoint

These counts come from the project run shared during the exercise. They are observations, not a full control opinion.

| Rule | Period | Expected triggers | Timely consistent receipts | Matching alerts | Timely alerts | Missing alerts |
|---|---|---:|---:|---:|---:|---:|
| DET-01 | incident_discovery | 20 | 20 | 0 | 0 | 20 |
| DET-01 | early_retest | 1 | 1 | 1 | 1 | 0 |
| DET-02 | incident_discovery | 21 | 21 | 0 | 0 | 21 |
| DET-02 | early_retest | 2 | 2 | 2 | 2 | 0 |

Interpretation:

- Trigger events in these populations reached the collector on time.
- Incident-period expected triggers had no matching alerts in the supplied evidence.
- Three early-retest cases generated timely alerts.
- September DET-02 configuration lists only the case-view event type.
- November and February snapshots list both event types.
- The later snapshots and successful tests do not demonstrate continuous operation between snapshots or sustained remediation effectiveness.
- A rule/period combination absent from the summary means no qualifying cases were reconstructed, not a pass.
- Forty-one incident-period trigger opportunities do not mean forty-one attacks, customers, or independent incidents.

## 9. Turn the evidence into an assessment

```text
Risk
└── What could go wrong?
    └── Control objective
        └── What should the control accomplish?
            └── Criterion
                └── What exact requirement applies?
                    ├── Design evaluation
                    │   └── Would the documented design address the objective?
                    └── Operating test
                        └── Did the control perform as required in the tested cases?
                            └── Conclusion, limitations, and remediation
```

| Assessment element | Example for DET-02 |
|---|---|
| Risk | High-volume attachment access through an unmonitored route may not generate an alert |
| Objective | Detect qualifying file-access activity across both routes |
| Criterion | SUP-03 criterion in the original workbook, cell F4 |
| Design evidence | Dated detection configuration records |
| Operating evidence | Source events, reconstructed triggers, collector receipts, alerts, and approved tests |
| Observation | State the precise condition with dates, counts, and record IDs |
| Design conclusion | State whether the specific design requirement is met, not whether all of SUP-03 is effective |
| Operating conclusion | State what the tested cases demonstrate, separately by period |
| Limitations | Point-in-time snapshots, selected tests, unverified implementation details, and incomplete evidence |

Suggested order for the workpaper: criterion -> population -> procedure -> evidence -> observation -> conclusion -> limitation -> recommendation and retest requirement.

## 10. Evidence and period boundaries

- Baseline source period: September 14-27, 2023.
- Incident source period: September 28-October 17, 2023; discovery/response records extend into October 18-19.
- Early retest: November 3-9, 2023.
- Late retest: February 8-9, 2024; only two days, not a long-term operating period.
- Transition days do not have complete daily operating evidence.
- November 29 documentary reassessment is not a complete November log population.
- Preserve distinctions among source-event time, collector receipt time, vendor availability, and bulk export delivery.
- Public historical incident counts are not the scaled synthetic exercise populations.
- Approved tests are retest evidence, not adversary activity.

## 11. Running the existing reporting workflow

Keep `build_monitoring_report.py` beside `analysis.py`. The reporter runs the existing script, so both are required.

```bash
python -m pip install pandas openpyxl
python build_monitoring_report.py
```

Generated outputs in the project:

```text
assessment_output/
├── Monitoring_Evidence_Review.xlsx
├── monitoring_summary.csv
└── analysis_run_log.txt
```

Review the validation log even when workbook generation succeeds. Some existing validation checks report counts without stopping execution. The reporter overwrites generated outputs on a later run; preserve dated copies when retaining workpaper evidence.

Do not treat the generated `Exceptions` table as the sole evidence-quality check: it covers receipt/alert checks, while other checks are recorded in the log.

## 12. What remains outside the current monitoring calculations

```text
Current calculation coverage
├── DET-01 expected events, receipts, and alert matching/timing
├── DET-02 reconstructed triggers, receipts, and alert matching/timing
└── Traceable contributing events and period summaries

Not yet completed by the reporter
├── Full vendor-export completeness and replacement reconciliation
├── Investigation-query correctness and scope reconciliation
├── Full design assurance, routing, investigation, and response evaluation
├── Sustained remediation-effectiveness assessment
└── The remaining core controls and optional extensions
```

## Source basis

This guide was prepared from the original V3 workbook, its README, the uploaded analysis script, the reporting-script structure, and the run results shared in the project conversation. Exact criteria above are reproduced from the workbook. Navigation explanations and workflow recommendations are supporting guidance, not additional fictional policy requirements.
