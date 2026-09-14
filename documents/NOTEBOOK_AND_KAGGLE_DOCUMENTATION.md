# Rufunsa Town Council: Notebook and Kaggle Documentation

## 1. Project purpose

This project builds reusable datasets from Rufunsa Town Council publications.  The work centres on Constituency Development Fund (CDF) community-project submissions and tables extracted from the Rufunsa Integrated Development Plan (IDP).  It is intended to make source material easier to inspect, analyse, and publish as an open dataset.

The main workflow is implemented in [`notebooks/Untitled.ipynb`](../notebooks/Untitled.ipynb).  The notebook creates the project directories, records source links, downloads source files, extracts tables, cleans records, adds provenance fields, and writes pipe-delimited CSV files.

## 2. Repository layout

| Location | Contents | Status |
| --- | --- | --- |
| `notebooks/Untitled.ipynb` | 96-cell Python notebook (80 code, 16 Markdown) | Primary workflow; contains exploratory and production cells |
| `data/raw/rufunsa_sources.csv` | Inventory of 8 official council sources | Pipe-delimited |
| `data/raw/rufunsa_cdf_links.csv` | 9 filtered CDF links discovered from the council CDF page | Pipe-delimited |
| `data/raw/2025_cdf_projects_received_from_wdcs.xlsx` | Original 2025 WDC community-project submission workbook | Source file |
| `data/raw/rufunsa_idp.pdf` | Rufunsa Integrated Development Plan PDF | Source file |
| `data/raw/2025_approved_secondary_school_bursaries.pdf` | Downloaded bursary-related PDF | Source file; see known limitations |
| `data/cleaned/db-unza26-csc4792-rufunsa_cdf_community_projects.csv` | Curated 2025 community-project dataset | Analysis-ready, subject to data-quality notes |
| `data/cleaned/db-unza26-csc4792-rufunsa_idp_projects.csv` | Tables extracted from the IDP PDF | Broad raw extraction; not a project-only table |

All CSV files use a pipe (`|`) delimiter. Read them with `pd.read_csv(path, sep="|")`.

## 3. Data sources and provenance

The notebook records the following official sources in `rufunsa_sources.csv`:

1. Rufunsa Town Council website.
2. Rufunsa Town Council CDF page.
3. WDC Community Project Submissions 2024.
4. Rufunsa Integrated Development Plan, 2024–2034.
5. Rufunsa Constituency and Wards page.
6. Council publications page.
7. Civic leaders page.
8. Council departments page.

The community-project dataset is derived from the council workbook **“2025 CDF Projects Received from WDCs.”** Each exported record retains the source URL, source document title, and retrieval date so that results can be traced back to the publication.

## 4. Notebook workflow

### A. Set up folders and source inventory

The first cells create `data/raw`, `data/cleaned`, and `documents`. They then construct a small source catalogue and save it to `data/raw/rufunsa_sources.csv`.

### B. Discover CDF publications

The notebook requests the Council CDF page, parses its links with BeautifulSoup, filters links matching CDF-project, bursary, empowerment, skills-development, road, and approval titles, and saves the result in `data/raw/rufunsa_cdf_links.csv`.

### C. Build the 2025 CDF community-project dataset

The notebook selects the **2025 CDF Projects Received from WDCs** link, downloads its Excel workbook, and identifies the actual header row (row 4 when loaded with `header=None`). It then:

- removes wholly empty rows and columns;
- removes duplicate records;
- normalises column names;
- renames key fields such as `no.` to `project_id` and `number_of_beneficiaries` to `beneficiaries`;
- adds council, province, district, constituency, financial year, source, and retrieval metadata; and
- exports the result to `data/cleaned/db-unza26-csc4792-rufunsa_cdf_community_projects.csv`.

### D. Explore bursary, skills, and empowerment sources

The next section identifies relevant CDF links and demonstrates requests and PDF-table extraction with `pdfplumber`. It defines target fields for bursaries and empowerment, but it is not yet a completed, reliable export workflow.

### E. Extract IDP tables

The IDP section downloads the PDF, extracts text and every detected table with `pdfplumber`, concatenates the table fragments, adds geographic metadata, validates basic shape/duplicates, and writes `db-unza26-csc4792-rufunsa_idp_projects.csv`.

## 5. Published CDF community-project dataset

`db-unza26-csc4792-rufunsa_cdf_community_projects.csv` contains **96 records** and **20 columns**.

| Field | Meaning |
| --- | --- |
| `project_id` | Identifier/sequence from the source workbook |
| `ward` | Ward where the project is proposed |
| `zone` | Zone/locality reported in the source |
| `project_name` | Name of the proposed project |
| `project_description` | Narrative project description |
| `sector` | Reported sector |
| `project_type` | Reported project type |
| `beneficiaries` | Beneficiary count or source text; not consistently numeric |
| `cdf_application_form_yes_no` | Whether a CDF application form was reported |
| `wdec_recommendation_yes_no` | Whether WDEC recommendation was reported |
| `wdc_minutes_yes_no` | Whether WDC minutes were reported |
| `zonal_minutes_yes_no` | Whether zonal minutes were reported |
| `council`, `province`, `district`, `constituency` | Geographic metadata added by the notebook |
| `financial_year` | Financial year (2025) |
| `source_url` | Direct source-workbook URL |
| `source_document` | Source document title |
| `retrieval_date` | Date the notebook wrote the dataset |

### Snapshot of the CDF data

- The largest reported sector is Education (46 records), followed by Water (20) and Health (18).
- The dataset includes 13 distinct ward spellings/values. `Chitimbwi` and `Chitimbwi ` differ only by trailing whitespace and should be standardised before ward-level totals are calculated.
- The `beneficiaries` column is mixed-format: most values are numeric, while some are blank, `nill`/`Nill`, or text such as `12 households`. Treat it as text until a clearly documented numeric-cleaning rule is applied.
- Sector and project-type labels also have inconsistent whitespace and wording (for example, `Education` and `Education `). Normalise these values before aggregation.

The document values reflect the files present in this repository on 14 September 2026 and are not claims of current project approval, funding, or completion.

## 6. IDP extract: important interpretation note

The IDP output has **2,037 rows** and **16 columns**. It contains all tables detected in the PDF, including demographic and other plan tables, rather than a curated list of IDP projects. Several columns are positional names (`0`, `1`, …) because the PDF tables do not share a single consistent schema.

For Kaggle, publish this file as **“Rufunsa IDP PDF Table Extract”**, not as “IDP Projects,” unless it is further filtered and transformed into the target project schema:

`record_id, council, province, district, constituency, ward, project_name, sector, project_description, implementation_period, project_status, estimated_cost, source_url, source_document, retrieval_date`.

## 7. Kaggle publication guide

### Recommended dataset title

**Rufunsa 2025 CDF Community Project Submissions**

### Suggested Kaggle description

> This dataset contains 96 community-project submissions received from Ward Development Committees for the 2025 Constituency Development Fund cycle in Rufunsa, Lusaka Province, Zambia. It was transformed from an official Rufunsa Town Council workbook. The dataset preserves source URL, document name, and retrieval date for provenance. It represents submissions received and should not be interpreted as a list of approved, funded, or completed projects.

### Files to upload

1. `db-unza26-csc4792-rufunsa_cdf_community_projects.csv` — recommended primary dataset.
2. `rufunsa_sources.csv` — recommended supporting provenance file.
3. `rufunsa_cdf_links.csv` — optional link catalogue.
4. `db-unza26-csc4792-rufunsa_idp_projects.csv` — optional, only when labelled as a raw PDF table extract and accompanied by the limitation note above.

Do not upload the source PDFs/XLSX unless their reuse terms and personal-data implications have been reviewed.

### Tags and usability notes

Suggested tags: `zambia`, `public-sector`, `local-government`, `community-development`, `open-data`, and `infrastructure`.

Set the dataset type to tabular data. State that values are sourced from a public local-government publication, that the data is not independently verified, and that the original source should be consulted for official decisions.

### Kaggle notebook starter

```python
import pandas as pd

cdf = pd.read_csv(
    "/kaggle/input/rufunsa-2025-cdf-community-project-submissions/"
    "db-unza26-csc4792-rufunsa_cdf_community_projects.csv",
    sep="|"
)

# Standardise category fields before analysis
for col in ["ward", "sector", "project_type"]:
    cdf[col] = cdf[col].astype("string").str.strip()

# Preserve the raw beneficiary field; derive a numeric field safely.
cdf["beneficiaries_numeric"] = pd.to_numeric(
    cdf["beneficiaries"], errors="coerce"
)

projects_by_sector = cdf["sector"].value_counts(dropna=False)
projects_by_ward = cdf["ward"].value_counts(dropna=False)
```

## 8. Requirements and reproduction

The notebook uses Python plus `pandas`, `requests`, `beautifulsoup4`, `pdfplumber`, and an Excel engine supported by pandas (normally `openpyxl`).

Run the notebook from the `notebooks` directory so paths such as `../data/raw` resolve to the repository’s `data/raw` folder. Internet access is needed for the download cells; the existing raw files allow later cleaning and analysis to be repeated without downloading them again.

For reproducibility, avoid using `verify=False` in production requests unless there is a documented certificate-validation issue. Capture package versions in a `requirements.txt` or Kaggle environment specification before publishing a fully reproducible notebook.

## 9. Known limitations and recommended next work

1. A generic early download block refers to `file_type` before it is set; it needs parameters or removal.
2. The bursary/skills/empowerment section mixes source variables and filenames (for example, it downloads from `skills_url` while saving a bursary filename) and later uses variables that are not defined in the notebook (`bursary_url`). It should be rebuilt as separate, explicit pipelines for each source.
3. The PDF-table code has an indentation issue in one conditional block and reuses `all_tables` across sections. Use scoped variables such as `bursary_tables` and `idp_tables`.
4. The IDP output needs a table-selection, header-detection, and field-mapping stage before it can support project-level analysis.
5. Add validation rules for IDs, ward names, categorical labels, beneficiary values, source URLs, and duplicate keys. Keep the original raw source unchanged and publish a separate data-cleaning log.

## 10. Citation

When using the primary dataset, cite:

> Rufunsa Town Council. (2025). *2025 CDF Projects Received from WDCs* [Excel workbook]. Retrieved through the Rufunsa Town Council CDF page. Processed in the CSC4792 Rufunsa project.

Also cite the direct `source_url` included in the dataset and record the version/retrieval date used in the analysis.
