# IRS Form 990: Significant Diversion of Assets and Nonprofit Risk Analysis

An end-to-end data science research project using publicly available IRS Form 990 XML filings and historical National Center for Charitable Statistics (NCCS) data to study financial and governance patterns associated with **self-reported significant diversions of assets** and to develop exploratory nonprofit investigation-prioritization methods.

**Project status (November 2026):** IRS XML ingestion, data quality assessment, financial feature engineering, historical NCCS integration, exploratory modeling, anomaly detection, investigation-queue construction, robustness checks, and a versioned 2024 baseline snapshot have been completed. The baseline remains **pending formal approval and additional validation**. An explanatory Databricks dashboard is a downstream use of the saved scores, not a reason to retrain the models.

> **Interpretation:** A significant-diversion disclosure is self-reported; it is **not independently verified fraud**. A negative disclosure does not establish that diversion or fraud did not occur. Model scores and consensus ranks are research prioritization signals, **not calibrated probabilities of fraud, findings of misconduct, or allegations against any organization**.

## 1. Project overview

IRS Form 990 is an annual information return for many U.S. tax-exempt organizations. Part VI, Section A, Line 5 asks whether an organization became aware of a significant diversion of its assets during the year. The XML field `MaterialDiversionOrMisuseInd` is used to derive the project's IRS diversion-disclosure outcome.

The project aims to:

- Ingest and audit IRS XML filings while distinguishing **publication year** from **tax year**.
- Identify and responsibly interpret self-reported diversion disclosures.
- Build auditable filing- and organization-year financial and governance features.
- Integrate historical NCCS data, measure coverage and missingness, and compare data sources.
- Develop supervised disclosure-prediction and unsupervised peer-anomaly signals.
- Compare transparent investigation rankings, evaluate their sensitivity, and document limitations before operational use.

## 2. Data sources and scope

**IRS XML:** Publication archives spanning 2023–2026, with the initial labeled full-Form-990 analysis focused on tax years 2023–2025. Publication year and tax year differ: a filing released in 2025 may report an earlier tax year. Full Form 990 is modeled separately from 990-EZ, 990-PF, and 990-T, which have different schemas and are not assigned negative diversion labels by default.

**NCCS historical data:** Historical nonprofit tables covering 2010–2024, used for source audits, longitudinal features, modeling, 2024 scoring, and peer comparisons. Source coverage, label availability, and join behavior are explicitly audited; not every organization has an observed diversion-disclosure label.

Sources and supporting documentation:

- [IRS Form 990 series downloads](https://www.irs.gov/charities-non-profits/form-990-series-downloads)
- [Dataset Summary and Data Quality Report](docs/DATASET_SUMMARY.md)

Original ZIP archives, downloaded NCCS Parquet files, generated data, models, investigation packets, and reports are not distributed in this repository.

## 3. Historical IRS ingestion and data-quality results

The completed IRS ingestion and quality assessment reported:

| Metric | Result |
| --- | ---: |
| Original XML archives processed | 50 |
| Filing-level records ingested | 841,900 |
| Positive diversion disclosures | 809 |
| Negative diversion disclosures | 841,091 |
| Missing disclosure labels in the eligible ingested cohort | 0 |
| XML parsing errors reported | 0 |
| Positive disclosure rate | 0.0961% |
| Provisional organization-year records | 829,043 |
| Organization-year groups with conflicting labels | 16 |
| Conservative modeling candidate records | 829,027 |
| Positive modeling candidate records | 776 |

### Disclosure rates by tax year

| Tax year | Filings | Positive disclosures | Positive rate |
| --- | ---: | ---: | ---: |
| 2023 | 359,633 | 333 | 0.0926% |
| 2024 | 362,975 | 349 | 0.0961% |
| 2025 | 119,292 | 127 | 0.1065% |

These are **historical results from the original processed IRS cohort**, not newly recomputed statistics or estimates of fraud prevalence. The 2025 tax-year cohort was incomplete. Canonical organization-year selection was deterministic but provisional: authoritative amendment status and final filing order were not independently verified. Sixteen conflicting-label organization-year groups were excluded from the conservative modeling candidates.

The original Notebook 00 pilot processed 5,000 XML files and identified 2,703 eligible full Form 990 filings, including one positive disclosure.

## 4. Notebook guide

The repository contains **23 notebooks**, numbered 00–16 with intermediate lettered notebooks. The table describes their roles in the historical analytical workflow; it is **not an instruction to run them all**.

| Notebook | Purpose |
| --- | --- |
| `00` | IRS XML feasibility, disclosure-label definition, and pilot audit |
| `01` | Full IRS XML ingestion and archive-level validation |
| `02` | Data quality, filing reconciliation, deduplication, and EDA |
| `03` | Comprehensive ingestion of other 990-series XML into separate datasets |
| `04` | Standard Form 990 financial, compensation, governance, and quality features |
| `04B` | Form-specific features for 990-EZ, 990-PF, and 990-T |
| `05` | Final organization-year modeling dataset assembly and quality checks |
| `06` | Leakage-aware anomaly detection and diversion-disclosure prediction |
| `07` | NCCS-versus-IRS data comparison |
| `08` | NCCS historical integration, 2010–2024, and initial baselines |
| `08A` | NCCS source and join audit |
| `09` | Improved historical NCCS modeling and 2024 scoring |
| `09A` | NCCS source completeness audit |
| `09B` | Model optimization and ranking experiments |
| `09C` | Missingness and model stabilization |
| `09D` | Scoring-population and model-selection audit |
| `10` | Financial and governance peer-anomaly detection |
| `11` | Integration and comparison of risk signals and rankings |
| `12` | Organization-level investigation queue and explanatory context |
| `13` | Queue validation, sensitivity analysis, and review design |
| `14` | Organization-size effects and ranking robustness |
| `15` | Blinded case-study materials and human lead-quality review workflow |
| `16` | Baseline evaluation, leakage audit, protected snapshot, and Databricks handoff |

See each notebook's opening Markdown, configuration section, and output descriptions for its specific inputs, run controls, and artifacts.

## 5. Analytical methodology and interpretation

### Supervised disclosure model

The supervised model estimates association with **observed self-reported significant-diversion disclosures**, not independently confirmed fraud. Rare positive labels, historical coverage, selection effects, and disclosure behavior limit interpretation. Identifier- and outcome-derived fields must not be used as predictive features; organization identity is useful for grouping and leakage-aware evaluation.

### Peer anomalies and consensus rankings

The analysis also develops directional financial and governance peer indicators and explores an Isolation Forest comparison. Notebook 11 compares the supervised signal with a six-indicator directional peer score and other diagnostics. The exploratory consensus ranking uses **equal weights on supervised and peer percentiles (50% / 50%)**. Those weights have **not been validated as optimal**, and the components can share underlying financial information; they are not independent evidence of misconduct.

### Organization-level investigation queues

Notebooks 12–15 deduplicate to the organization level, add historical and peer-relative context, compare ranking stability, assess organization-size effects, and prepare blinded review materials. The review workflow distinguishes generated case packets from **completed human judgments**. No verified lead-quality result should be inferred from an uncompleted worksheet or a model rank.

### Evaluation and frozen baseline

Notebook 16 prepares the `nccs_2024_baseline_v1` technical snapshot and related evaluation and handoff artifacts. The 2023–2024 period had already been examined and is **not a pristine, untouched prospective test set**. A code-level feature-lineage review found no obvious direct outcome or future-tax-year leakage, but **publication-date validation and formal approval remain outstanding**. Feature-name screening alone cannot establish the absence of indirect leakage.

Organization size affects supervised rankings. Size-stratified analyses and simpler benchmarks are important context; they do not by themselves establish a better operational ranking. The frozen baseline is an **exploratory comparison baseline**, not an approved fraud-detection system.

## 6. Current usage: inspect existing work without rerunning it

**The historical analyses have already been executed, and the existing generated outputs and baseline should be treated as frozen. Do not rerun notebooks simply to view the results, prepare a dashboard, or verify that GitHub contains the code.** The notebooks committed to GitHub have their cell outputs cleared; that does **not** mean the underlying analyses were never run or that the models must be retrained.

For current review and dashboard work:

1. Read the notebooks, their Markdown findings, and `docs/DATASET_SUMMARY.md` without executing expensive cells.
2. Use the **existing saved datasets, scores, model artifacts, and manifests** from controlled local or Databricks storage. They are intentionally excluded from GitHub.
3. Validate schema, row counts, unique organization identifiers, version information, and artifact provenance before displaying scores.
4. Keep investigation materials, evaluation outcomes, and restricted case profiles separate from general-purpose dashboard scoring tables.
5. Label rankings and scores as exploratory indicators; never present them as fraud probabilities or confirmed allegations.

GitHub is the **code and documentation repository**, not the system of record for frozen data and model artifacts. Avoid regenerating or overwriting historical exports unless a deliberate, separately documented reproduction or new baseline version is intended.

## 7. Historical reproduction: setup and execution

**This section describes how the original pipeline can be reproduced intentionally. It is separate from current read-only usage.** Reproduction requires original source archives, NCCS files, adequate storage and memory, and careful attention to each notebook's dependencies. Some stages train models or overwrite generated artifacts.

### Prerequisites

- Python 3.11, Git, and JupyterLab.
- Conda or a compatible Python environment manager.
- Disk space for IRS ZIP archives, NCCS historical Parquet files, intermediate extracts, and reports.
- A compatible 7-Zip executable (`7zz`) for IRS archives that use unsupported compression such as Deflate64.

### Clone and install

```bash
git clone https://github.com/julietday422/irs-990-fraud-detection.git
cd irs-990-fraud-detection
conda create -n irs990 python=3.11 -y
conda activate irs990
python -m pip install -r requirements.txt
python -m pip install jupyterlab ipykernel
python -m ipykernel install --user --name irs990 --display-name "Python (irs990)"
jupyter lab
```

The repository also includes `environment.yml`; consult it when setting up a Conda-managed environment. If using that file, avoid creating a second environment unnecessarily. Some notebook versions support configuring the 7-Zip path with the `SEVEN_ZIP` environment variable.

### Source data and execution controls

Download original IRS archives from the [IRS download site](https://www.irs.gov/charities-non-profits/form-990-series-downloads) and obtain the required historical NCCS sources separately. Place them in the paths expected by the relevant notebooks; source and generated data are not included in the clone.

**Inspect configuration before running any notebook.** Relevant safeguards in the reviewed versions include:

| Notebook | Execution safeguard or setting |
| --- | --- |
| `00` | `DOWNLOAD_ONE_ARCHIVE = False` by default; explicitly opt in to the pilot download |
| `01` | `RUN_INGESTION = False` by default; `RUN_MODE` controls the ingestion scope when enabled |
| `03` | `RUN_MODE = "test"` for limited processing; incomplete existing outputs require inspection before rerunning |
| `04` | `RUN_EXTRACTION = False` by default; `MODE` selects repair versus full extraction when enabled |
| `04B` | `RUN_EXTRACTION = False` by default |
| `05` | `WRITE_OUTPUTS = False` by default; enable only for intentional exports |
| `08` | `RUN_DOWNLOADS = False` and `WRITE_OUTPUTS = False` by default; other analysis and modeling cells may still execute |
| `06`, `09`–`16` | Review individual cells carefully: model training, reports, rankings, case packets, or manifests may be regenerated or overwritten |

The notebook numbers indicate a **logical dependency history**, not a guarantee that every notebook can be run independently or safely in numerical order without restoring its inputs. Preserve historical output versions and record changes before reproducing or rebuilding anything.

## 8. Databricks baseline and dashboard handoff

The project includes a 2024 NCCS baseline prepared for controlled Databricks ingestion and comparison with future experiments. The existing Databricks environment includes the `irs_ofe_fwa_demo.nccs_demo` schema and baseline scoring assets, including `baseline_v1_scores`, `baseline_v1_limited_coverage`, and `vw_nonprofit_risk_dashboard`.

The intended dashboard is **explanatory**, not a retraining workflow. It should help users understand organization-level rankings, financial and governance indicators, coverage limitations, methodology, and the differences among supervised, peer, and consensus signals.

For ingestion or validation, follow Notebook 16's handoff instructions: keep the outcome-free primary scoring export separate from restricted evaluation outcomes and case profiles, load limited-coverage organizations separately, verify row counts and EIN uniqueness, and retain the baseline version and file hashes. A frozen technical snapshot is **not the same as formal model approval**.

## 9. Repository contents and data protection

```text
irs-990-fraud-detection/
├── notebooks/                # 23 reviewed analysis notebooks (00–16)
├── docs/
│   └── DATASET_SUMMARY.md
├── README.md
├── requirements.txt
├── environment.yml
└── .gitignore
```

Local analysis also uses directories such as `data/`, `reports/`, and model-output locations, but generated datasets and artifacts are excluded from Git. The `.gitignore` excludes large data formats, model files, local environments, checkpoints, and generated reports. **Do not commit raw filing exports, restricted investigator packets, evaluation outcomes, credentials, or sensitive local paths.** Check staged changes before each push; `.gitignore` does not automatically untrack previously committed files.

## 10. Limitations and responsible use

- The target is a self-reported disclosure, **not a verified fraud event**; nondisclosure is not proof of no misconduct.
- The positive class is rare and reporting practices may vary.
- IRS 2025 tax-year coverage in the original cohort was incomplete; filing histories and canonical selections may be provisional.
- NCCS source coverage, missingness, and label availability affect which organizations can be trained on, evaluated, or ranked.
- Retrospective evaluation on previously examined years is not independent prospective validation.
- Financial and governance indicators can reflect organization size, reporting practices, or accounting context rather than misconduct.
- Correlated signals and unvalidated consensus weights limit interpretation.
- Human investigative review, publication-timing validation, and formal baseline approval remain outstanding where documented in Notebook 16.
- These analyses support exploratory research and contextual prioritization only. Independent verification is required before drawing conclusions about any specific organization.

## 11. Documentation

- [Dataset Summary and Data Quality Report](docs/DATASET_SUMMARY.md) — historical IRS modeling dataset and quality findings.
- [Notebooks](notebooks/) — methodology, configuration, historical analyses, evaluation, and handoff details.
