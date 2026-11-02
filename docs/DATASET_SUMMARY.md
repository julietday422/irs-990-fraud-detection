# IRS Form 990 Fraud Detection Project
## Final Dataset Summary and Data Quality Report

**Status:** Data ingestion, cleaning, feature engineering, and initial quality validation completed through Notebook 05.

The project has produced three Parquet datasets containing financial and organizational information from IRS Form 990 filings. Together, they provide a foundation for identifying unusual nonprofit financial behavior and studying organizations that self-report significant diversions of assets.

All three datasets are saved in:

`data/analytics/modeling/`

## 1. Overview of the three datasets

| Dataset | Records | Primary purpose |
|---|---:|---|
| `05_form990_modeling_organization_year_v1.parquet` | 829,027 | Main dataset for anomaly detection and predictive modeling |
| `05_other_series_canonical_v1.parquet` | 1,327,127 | Financial and organizational information from Forms 990-EZ, 990-PF, and 990-T |
| `05_other_series_link_summary_v1.parquet` | 829,027 | Links the main Form 990 cohort to additional IRS returns |
| **Total stored rows** | **2,985,181** | Across three datasets, not three independent sets of organizations |

The datasets are related but serve different analytical purposes. The main modeling dataset contains the target labels; the supplementary datasets provide additional context.

---

## 2. Dataset 1 — Main Form 990 Modeling Dataset

**Filename:** `05_form990_modeling_organization_year_v1.parquet`

| Metric | Value |
|---|---:|
| Organization-year records | 829,027 |
| Unique organizations | 388,961 |
| Positive disclosures | 776 |
| Negative disclosures | 828,251 |

### What it contains

This is the primary dataset for modeling. Each record represents one nonprofit organization for one tax year, identified using its Employer Identification Number (EIN) and tax year.

The dataset combines information extracted from full Form 990 XML returns, including:

| Category | Example fields | Analytical value |
|---|---|---|
| Organization and filing | `ein`, `tax_year`, `filing_id` | Identifies organizations and reporting periods |
| Financial activity | `revenue_cy`, `expenses_cy`, `contributions_cy`, `investment_income_cy` | Measures financial size and funding sources |
| Balance sheet | `assets_eoy`, `liabilities_eoy`, `net_assets_eoy` | Evaluates financial position |
| Governance | `voting_board_members`, `independent_board_members` | Describes oversight structure |
| Staffing and compensation | `employees_count`, `part_vii_compensation_total` | Supports compensation and governance analysis |
| Financial ratios | `operating_margin`, `expense_to_revenue`, `contributions_share` | Helps compare differently sized organizations |
| Financial changes | `revenue_change_pct`, `expenses_change_pct`, `assets_change_pct` | Identifies unusual changes |
| Target label | `fraud_label` | Identifies self-reported significant diversion disclosures |

### Label distribution

| Label | Organization-years |
|---|---:|
| No positive disclosure | 828,251 |
| Positive disclosure | 776 |

Only **0.094%** of the organization-year records have a positive disclosure.

This is important because a model could achieve approximately 99.91% accuracy by predicting no positive disclosures for every record, while failing to identify any positives.

### Data-quality issues identified

**1. Conflicting organization-year labels**

During earlier cleaning, 16 EIN/tax-year groups had conflicting disclosure labels. These were excluded from the modeling cohort rather than arbitrarily choosing a label.

**2. Financial reconciliation discrepancies**

Only 14 of 829,027 eligible records failed the balance sheet equation by more than $1.

Mismatch rate: approximately 0.002%.

**3. Functional expense inconsistencies**

During feature engineering, 80,526 full Form 990 records had discrepancies exceeding $1 in the functional-expense checks. A substantial subset reported a positive total functional expense amount while extracted expense components were zero.

The extraction logic was corrected to use the appropriate XML group, and unreliable expense allocation ratios were masked rather than treated as valid zeros.

**4. Incomplete recent-year coverage**

The modeling cohort contains:

| Tax year | Records |
|---|---:|
| 2023 | 352,798 |
| 2024 | 357,851 |
| 2025 | 118,378 |

The smaller 2025 cohort reflects incomplete coverage and must not be interpreted automatically as a decline in nonprofit filings.

**5. Label limitations**

The positive label represents a nonprofit's **self-reported significant diversion of assets**. It does not establish criminal fraud or independently verified misconduct.

Likewise, a negative label means no positive disclosure was identified—not that the organization is guaranteed free of fraud.

### Intended use

This dataset will support financial anomaly detection, predictive modeling of diversion disclosures, organization-level risk ranking, and investigation-oriented analytics.

---

## 3. Dataset 2 — Other 990 Series Canonical Dataset

**Filename:** `05_other_series_canonical_v1.parquet`

**Canonical supplementary filings: 1,327,127**

| Form | Canonical records |
|---|---:|
| 990-EZ | 780,349 |
| 990-PF | 463,159 |
| 990-T | 83,619 |

### What it contains

This dataset stores standardized information from three additional IRS return types.

**Form 990-EZ** is a shorter annual information return generally used by smaller eligible tax-exempt organizations. Extracted fields include revenue, expenses, contributions, program service expenses, assets, liabilities, and net assets.

**Form 990-PF** is used by private foundations. Extracted fields include investment income, grants paid, qualifying distributions, expenses, and balance sheet amounts.

**Form 990-T** reports unrelated business income and related tax information. Extracted fields include unrelated business taxable income, total tax, and estimated payments.

A Form 990-T record should not be interpreted as a separate nonprofit organization simply because it appears in this dataset.

### Duplicate filing handling

The initial extraction contained 1,340,182 filings. After canonical selection, 1,327,127 records remained.

| Form | Original filings | Canonical records | Excess records |
|---|---:|---:|---:|
| 990-EZ | 788,553 | 780,349 | 8,204 |
| 990-PF | 467,062 | 463,159 | 3,903 |
| 990-T | 84,567 | 83,619 | 948 |
| **Total** | **1,340,182** | **1,327,127** | **13,055** |

All records had valid organization-year keys.

Canonical selection is deterministic, but **not amendment-verified**. In other words, the process consistently selects one record per form/organization/year, but we have not confirmed that every selected filing is the authoritative amended return.

### Data-quality issues identified

The largest issues involved balance sheet reconciliation.

| Check | Eligible filings | Mismatches | Rate |
|---|---:|---:|---:|
| 990-EZ operating reconciliation | 753,905 | 44 | 0.006% |
| 990-EZ balance sheet | 458,567 | 9,742 | 2.12% |
| 990-PF operating reconciliation | 467,062 | 33 | 0.007% |
| 990-PF balance sheet | 450,296 | 13,048 | 2.90% |

Operating reconciliation performed extremely well. Balance sheet reconciliation was less reliable.

### Investigation of a major discrepancy

We examined original IRS XML for Form 990-PF filing `202533149349102223`.

The filing reported:

| Financial measure | End-of-year value |
|---|---:|
| Assets | $301,752 |
| Liabilities | $0 |
| Net assets | $55,528,839 |
| **Absolute discrepancy** | **$55,227,087** |

The inconsistency was present in the original IRS XML, confirming that this particular discrepancy was not introduced by our extraction.

Among the 13,048 mismatched 990-PF filings:

- 5,479 reported zero end-of-year assets.
- 11,437 reported zero end-of-year liabilities.
- 6,658 reported net assets greater than reported assets.

These categories overlap and should not be added together.

### Other limitations

Some financial fields are missing because of differences in form requirements, reporting practices, or XML availability. Missing liabilities are not automatically equivalent to zero liabilities.

We preserved original values and reconciliation information instead of deleting mismatched filings or changing reported amounts.

### Intended use

This dataset provides supplementary financial context and allows future research into smaller nonprofits, private foundations, and unrelated business income.

It is **not** part of the primary supervised diversion-label cohort.

---

## 4. Dataset 3 — Supplementary Return Link Summary

**Filename:** `05_other_series_link_summary_v1.parquet`

**Link summary records: 829,027**

One record corresponding to each main Form 990 organization-year.

### What it contains

This dataset connects organizations in the primary Form 990 cohort with additional IRS return types.

For example, an organization might file a full Form 990 and a separate Form 990-T for the same tax year.

The link summary identifies these relationships without adding extra rows to the main modeling dataset.

Fields include supplementary filing counts such as:

- `linked_irs990ez_return_count`
- `linked_irs990pf_return_count`
- `linked_irs990t_return_count`

### Linkage results

| Supplementary form | Main organization-years with matches | Share of cohort |
|---|---:|---:|
| 990-EZ | 762 | 0.09% |
| 990-PF | 46 | 0.006% |
| 990-T | 32,876 | 3.97% |

The 990-T matches are particularly relevant because organizations can file Form 990 and Form 990-T for different reporting purposes.

### Data-quality issues and limitations

**1. Cross-form overlap**

The 990-EZ and 990-PF overlaps are relatively uncommon but merit further review. We haven't established whether they arise from amended filings, form changes, duplicate submissions, or tax-period alignment.

**2. Temporal availability**

A linked supplementary filing might have been submitted or published after the primary Form 990 filing.

That creates a potential data-leakage issue if we use supplementary information to predict an outcome as of an earlier date.

**3. Link counts are not financial risk measures**

The presence of a 990-T return does not imply fraud or misconduct. It indicates that the organization has a corresponding tax filing.

**4. Record preservation**

The link summary has exactly the same number of rows as the primary dataset, supporting the intended one-to-one organization-year linkage. The final QA checks also preserved the primary disclosure-label counts.

### Intended use

This dataset supports cross-form analysis, enrichment, investigation, and potentially future predictive features after temporal availability checks.

---

## 5. Overall data-quality assessment

| Quality dimension | Assessment | Remaining consideration |
|---|---|---|
| Ingestion | Complete | Source archives remain separate |
| Organization-year identification | Validated | Canonical supplementary filings not amendment-verified |
| Main dataset row counts | Passed | 829,027 records preserved |
| Disclosure labels | Passed completeness checks | Labels are self-reported, not verified fraud |
| Main financial reconciliation | Strong | 14 balance sheet mismatches |
| Supplementary operating reconciliation | Strong | Very low mismatch rates |
| Supplementary balance sheets | Requires caution | Approximately 2–3% of eligible filings mismatch |
| Missing financial values | Partially addressed | Model-specific handling remains |
| Supplementary linking | Completed | Timing and overlap need review |
| Model leakage prevention | Not yet completed | Planned for Notebook 06 |

## 6. What remains before modeling

Although the datasets have been cleaned and saved, some modeling-specific preparation remains.

The next notebook should establish a leakage-safe predictor list, remove disclosure-derived information from the feature matrix, define time-aware and organization-aware evaluation, handle missing values using training-only transformations, and evaluate anomaly detection separately from supervised prediction.

We should also be cautious about treating financial discrepancies as predictive signals until we have established that they reflect genuine anomalies rather than reporting or extraction artifacts.

## Final conclusion

The project now has a validated, reproducible data foundation consisting of a main labeled Form 990 dataset, a canonical supplementary return dataset, and an organization-year linkage dataset.

**The central achievement is not that every IRS filing is financially consistent.** It is that we have systematically ingested the data, reconciled key fields, identified and documented exceptions, preserved source values, and separated data-quality issues from actual diversion disclosures.

That provides a defensible starting point for Notebook 06, where we'll build models to identify unusual financial patterns and rank organizations for further review—without equating anomalies or self-reported diversions with confirmed fraud.
