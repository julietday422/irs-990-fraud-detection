# IRS Form 990: Significant Diversion of Assets Detection

An end-to-end data science project using publicly available IRS Form 990 XML filings to investigate financial patterns associated with nonprofit organizations that self-report significant diversions of assets.

The project combines large-scale XML ingestion, data quality analysis, financial feature engineering, and planned machine learning techniques to identify patterns and potential risk indicators in nonprofit financial reporting.

**Project status:** Data ingestion and quality assessment complete. Financial feature engineering and predictive modeling are upcoming stages.

## 1. Project Overview

IRS Form 990 is an annual information return filed by many tax-exempt organizations in the United States. It contains information about organizational finances, governance, operations, and compliance.

Part VI, Section A, Line 5 asks whether an organization became aware of a significant diversion of its assets during the year.

This project uses the corresponding XML field, `MaterialDiversionOrMisuseInd`, as the primary outcome variable.

**Important distinction:** A positive disclosure represents a self-reported significant diversion of assets, not independently verified fraud. A negative disclosure does not establish that fraud or asset diversion did not occur.

### Project objectives

- Build a reproducible pipeline for processing large volumes of IRS Form 990 XML filings.
- Identify organizations reporting significant diversions of assets.
- Assess data completeness, duplicate filings, and reporting inconsistencies.
- Engineer financial and organizational features associated with diversion disclosures.
- Explore classification, anomaly detection, entity resolution, and network analysis techniques.
- Evaluate whether observable reporting characteristics can help identify patterns associated with diversion risk.

## 2. Data Sources

**Source:** Publicly available IRS Form 990 XML filings.

**Target tax years:** 2023–2025

**Publication archives:** 2023–2026

Tax year and publication year are treated separately because organizations may file returns after the end of the tax year. A filing published in 2025, for example, may describe an organization's 2023 or 2024 tax year.

The dataset is restricted to eligible full Form 990 filings containing the relevant diversion-disclosure field.

The original XML archives are retained locally to support validation and additional feature extraction.

## 3. Key Results

The completed ingestion and data quality pipeline produced the following results:

| Metric | Result |
|---|---:|
| Original XML archives processed | 50 |
| Filing-level records ingested | 841,900 |
| Positive diversion disclosures | 809 |
| Negative diversion disclosures | 841,091 |
| Missing disclosure labels | 0 |
| XML parsing errors | 0 |
| Positive disclosure rate | 0.0961% |
| Provisional organization-year records | 829,043 |
| Organization-year groups with conflicting labels | 16 |
| Conservative modeling candidate records | 829,027 |
| Positive modeling candidate records | 776 |

### Disclosure rates by tax year

| Tax year | Filings | Positive disclosures | Positive rate |
|---|---:|---:|---:|
| 2023 | 359,633 | 333 | 0.0926% |
| 2024 | 362,975 | 349 | 0.0961% |
| 2025 | 119,292 | 127 | 0.1065% |

**Key observations:**

- Positive diversion disclosures are extremely rare, creating a substantial class imbalance for machine learning.
- Some organizations have multiple filings for the same tax year.
- A small number of organization-year groups contain conflicting diversion disclosures.
- Tax year 2025 has incomplete coverage because not all returns have been published.
- Organization-level deduplication and careful model evaluation are essential.

These are descriptive results from the processed filing dataset and should not be interpreted as estimates of actual nonprofit fraud prevalence.

## 4. Pipeline Architecture

The completed pipeline consists of three sequential Jupyter notebooks.

### Stage 1: Data Feasibility and XML Exploration

**Notebook:** `00_data_feasibility_irs990_fraud.ipynb`

Purpose: Determine whether IRS XML filings contain a usable outcome variable for studying significant diversions of assets.

Key activities:

- Inspect IRS Form 990 XML structure and reporting fields.
- Identify `MaterialDiversionOrMisuseInd`.
- Define positive, negative, and missing disclosure labels.
- Process an initial pilot sample of XML filings.
- Evaluate label availability and class imbalance.
- Verify a positive disclosure directly against its source XML.

The pilot processed 5,000 XML files and identified 2,703 eligible Form 990 filings, including one positive disclosure.

**Outcome:** Established the feasibility of using the IRS diversion-disclosure field for further analysis.

### Stage 2: Full XML Ingestion

**Notebook:** `01_full_xml_ingestion.ipynb`

Purpose: Build a scalable, reproducible ingestion process across the available IRS XML archives.

Key activities:

- Process 50 XML archives.
- Extract organization identifiers, tax years, publication metadata, and disclosure labels.
- Handle differences in ZIP compression formats, including archives requiring 7-Zip.
- Write archive-level Parquet partitions.
- Generate processing and validation reports.
- Skip successfully completed archives when rerunning the pipeline.

**Outcome:** Created a filing-level dataset containing 841,900 records with no missing disclosure labels or reported XML parsing errors.

### Stage 3: Data Quality, Deduplication, and Exploratory Analysis

**Notebook:** `02_data_quality_and_eda.ipynb`

Purpose: Assess dataset quality and construct an organization-year dataset suitable for subsequent analysis.

Key activities:

- Examine filing counts and disclosure rates by tax year.
- Compare tax years with publication years.
- Check for repeated filing identifiers and repeated EIN–tax-year combinations.
- Identify organizations with multiple filings.
- Investigate conflicting diversion-disclosure labels.
- Select provisional canonical organization-year records.
- Create a conservative modeling candidate dataset.

A canonical organization-year record is the selected representative filing for an organization and tax year.

The selection uses a deterministic rule based on tax-period end, publication year, and filing ID. This is **provisional**: authoritative amendment status and final filing order have not been independently verified.

The modeling candidate dataset excludes 16 organization-year groups containing conflicting disclosure labels.

**Outcome:** Produced 829,027 organization-year modeling candidates, including 776 positive disclosures.

## 5. Repository Structure

```text
irs-990-fraud-detection/
├── notebooks/
│   ├── 00_data_feasibility_irs990_fraud.ipynb
│   ├── 01_full_xml_ingestion.ipynb
│   └── 02_data_quality_and_eda.ipynb
├── data/
│   ├── raw/
│   ├── staging/
│   ├── cleaned/
│   └── analytics/
├── reports/
│   └── ingestion/
├── src/
├── models/
├── docs/
├── README.md
├── requirements.txt
└── .gitignore
```

### Data directories

- `data/raw/`: Original downloaded IRS XML ZIP archives.
- `data/staging/`: Intermediate archive-level Parquet partitions and pilot outputs.
- `data/cleaned/`: Consolidated filing-level and organization-year datasets.
- `data/analytics/`: Data quality summaries, duplicate analyses, and conflicting-filing records.
- `reports/`: Generated processing and ingestion reports.

Large datasets and generated reports are excluded from version control. They can be regenerated locally using the pipeline, provided the required source archives are available.

## 6. Installation and Setup

### Prerequisites

- Python 3.11
- Conda or another compatible Python environment manager
- JupyterLab
- Git
- Sufficient disk space for IRS XML archives and temporary extraction
- 7-Zip (`7zz`) for archives using unsupported ZIP compression methods

### Clone the repository

Once the project has been published to GitHub:

```bash
git clone https://github.com/YOUR-USERNAME/irs-990-fraud-detection.git
cd irs-990-fraud-detection
```

Replace `YOUR-USERNAME` with the repository owner's GitHub username.

### Create the Python environment

```bash
conda create -n irs990 python=3.11 -y
conda activate irs990
```

### Install dependencies

```bash
python -m pip install -r requirements.txt
```

The dependency list is maintained in `requirements.txt`.

If JupyterLab or the notebook kernel is not included in the installed dependencies, install them with:

```bash
python -m pip install jupyterlab ipykernel
```

Register the environment as a notebook kernel:

```bash
python -m ipykernel install --user --name irs990 --display-name "Python (irs990)"
```

### Configure 7-Zip

Some IRS ZIP archives use Deflate64 compression, which Python's standard `zipfile` library does not support.

Install a compatible 7-Zip executable and make `7zz` available on your system PATH, or configure the ingestion notebook to reference its location.

For a notebook version that supports the `SEVEN_ZIP` environment variable:

```bash
export SEVEN_ZIP="/path/to/7zz"
```

### Download the source data

Download the relevant IRS Form 990 XML archives from the IRS website and place them in:

```text
data/raw/
```

Source: https://www.irs.gov/charities-non-profits/form-990-series-downloads

The original XML archives are not included in this Git repository.

### Run the notebooks

Launch JupyterLab:

```bash
jupyter lab
```

Select the `Python (irs990)` kernel and run the notebooks in order:

1. `00_data_feasibility_irs990_fraud.ipynb`
2. `01_full_xml_ingestion.ipynb`
3. `02_data_quality_and_eda.ipynb`

Notebook 01 performs the full ingestion and can require substantial processing time and temporary disk space. Notebook 02 uses the generated Parquet partitions and consolidated data.

## 7. Modeling Considerations

### Class imbalance

Only approximately 0.096% of filing-level records contain positive disclosures.

Accuracy alone will therefore be a misleading model evaluation metric.

Future modeling will emphasize precision, recall, precision-recall AUC, and appropriate baseline comparisons.

### Target leakage

The outcome variable is derived from `MaterialDiversionOrMisuseInd`.

Columns derived directly from the target, including `ever_positive`, `positive_filing_count`, and `distinct_labels`, must not be used as predictive features.

Identifiers and source metadata should also be excluded from the default feature set. EIN can be retained for organization-level grouping and train/test splitting.

Features that directly reveal diversion disclosures or related follow-up information should not be included in predictive models.

### Organization-level leakage

The same organization may appear in multiple tax years.

Train/test splitting should account for EIN so that repeated observations of an organization do not create misleading model performance.

### Interpretation

Predicting a contemporaneous disclosure from the same year's financial information is different from prospectively predicting future diversion events.

Model objectives and evaluation designs must distinguish between these use cases.

## 8. Planned Next Steps

### Stage 4: Financial Feature Engineering

Extract additional fields from the original IRS XML filings, including:

- Revenue, expenses, assets, and liabilities
- Net assets and financial ratios
- Program service and fundraising expenses
- Contributions and grants
- Compensation-related measures
- Year-over-year financial changes

### Stage 5: Machine Learning

Explore supervised classification and unsupervised anomaly detection, with evaluation strategies designed for extreme class imbalance.

### Stage 6: Entity Resolution and Network Analysis

Investigate relationships among organizations, officers, addresses, and other shared attributes where the available data supports reliable linkage.

### Stage 7: Model Evaluation and Interpretation

Evaluate predictive performance, feature importance, data limitations, and potential use cases for nonprofit financial risk analysis.

## 9. Limitations

- Diversion disclosures are self-reported and do not independently establish fraud.
- Organizations reporting no diversion may still have experienced undisclosed incidents.
- Positive labels are extremely rare.
- The 2025 tax-year dataset is incomplete.
- Multiple filings may reflect amendments or other filing-history complexities.
- Canonical filing selection is provisional.
- Financial and governance variables may be correlated with organizational size, reporting behavior, and disclosure practices rather than misconduct itself.
- Any resulting model should be treated as a research tool, not as evidence of wrongdoing by an individual organization.

## 10. Project Scope and Responsible Use

This project is intended for exploratory data science research, reproducible analysis, and investigation of patterns in publicly available nonprofit filings.

Its purpose is not to accuse or classify individual organizations as fraudulent. Any analytical findings should be interpreted with appropriate context, uncertainty, and independent verification.

## 11. Documentation 
[Dataset Summary and Data Quality Report](docs/DATASET_SUMMARY.md)