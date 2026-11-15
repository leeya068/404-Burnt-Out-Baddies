
# 404-Burnt-Out-Baddies — UMDAC Datathon 2026 (Preliminary Round)

Reverse-engineering the business rules behind Swift's artificial cross-border payment dataset.

- **Preliminary Round:** 13–19 Nov 2026
- **Submission Deadline:** 19 Nov 2026, 12:00 PM
- **Deliverables:** Dashboard PDF + Slide deck PDF

---

## Team & Responsibilities

| Member | GitHub | Primary Role | Key Deliverables |
|---|---|---|---|
| *Aleeya Nazirah binti Jamil* | [@leeya068](https://github.com/leeya068) | **Data Cleaning & Dashboard Engineering** | Data pipeline (DuckDB aggregates), data preparation log, chart generation, dashboard layout, PDF assembly |
| *Raja Athira Sofea binti Raja Azlan* | @handle | **Presentation & Insight Synthesis** | Slide deck narrative structure, rule statements, economic interpretation, slide visual design, APA citations |
| *Alliyana binti Abdul Rahman* | @handle | **Presentation & Quality Assurance** | Cross-referencing dashboard evidence to slides, rubric self-check, compliance documentation, hypothesis log, final proofreading |

> **Team rule:** No member replacement after registration. Each member owns their deliverables end-to-end, but all three review the final PDFs together before submission.

---

### Workstream Breakdown

**Aleeya — Data & Dashboard**
- Build and maintain the DuckDB streaming pipeline (1.5 GB CSV → aggregated CSVs).
- Document all data cleaning decisions (null handling, placeholder parsing, dropped columns).
- Generate all charts using the agreed colour palette and layout template.
- Export the final dashboard PDF (12 pages) and ensure every chart shows a baseline comparison.
- Ensure all outputs are saved to the `output/` folder.

**Athira — Presentation & Narrative**
- Write the slide deck narrative: headline finding first, methodology, one slide per feature group, conclusion.
- Restate each reconstructed rule in the required format: *When [condition], [effect] by [magnitude], covering [scope].*
- Draft the economic interpretation for each rule (why it exists in the real world).
- Source and format all APA citations with embedded URLs.
- Design slide visuals to match the dashboard colour scheme.

**Alliyana — QA, Compliance & Final Check**
- Cross-check every slide claim against the corresponding dashboard chart (claim-evidence linkage).
- Maintain the hypothesis log: what was tested, what was rejected, what remains uncertain.
- Verify compliance: dataset-only usage, no external primary data, tool disclosures, AI usage explainability.
- Run the rubric self-check before submission (Insight Quality 45%, Evidence 23%, Communication 20%, Interpretation 12%).
- Proofread both PDFs for typos, formatting consistency, and page numbering.

---

## Competition Summary

The task is to **reverse-engineer the rules used to generate the dataset**. The data is artificial, built by distilling real payments traffic into business logic. We must reconstruct as much as possible of the causal logic, relationships, and business knowledge behind the data.

**Key principle:** A well-argued partial or negative result scores higher than a lucky guess. State rules in this format:

> *When [condition], [effect] by [magnitude], covering [scope].*

Separate observation from interpretation, and document rejected hypotheses.

### Judging Weights

| Criterion | Weight |
|---|---:|
| Insight Quality | 45% |
| Interpretation & Economic Reasoning | 12% |
| Evidence, Method & Data Integrity | 23% |
| Communication | 20% |

### Deliverables

1. **Dashboard PDF** — 5–15 dashboards, with one dashboard per feature-type group. Each dashboard must state the reconstructed rule(s) clearly and include at least one chart showing the rule in the data.
2. **Slide deck PDF** — 10–20 slides. Include an introduction and methodology, with a maximum of one slide per feature group. Each feature-group slide must restate the rule, reference the supporting chart, and explain the real-world economic behaviour.

---

## Dataset

- **File:** `swift_umdac_datathon_2026_fictitious_payment_data.csv`
- **Size:** 1.5 GB
- **Storage:** Local only; not committed to this repository (see `.gitignore`).
- **Description:** Artificial cross-border payments; a tabular, timestamped representation of a directed temporal pseudograph.

### Columns

- `EPOCH_SECONDS`
- `CURRENCY_CODE`
- `USD_AMOUNT`
- `ORDERING_BIC8`
- `BENEFICIARY_BIC8`
- `BIC8_PATH`
- `TOTAL_DURATION`
- `CONTINUITY_QFLAG`
- `CURRENCY_CONVERSION_COUNT`
- `PARTICIPANT_COUNT`
- `TRANSACTION_TYPE_CODE`
- `ORDERING_ACCOUNT`
- `BENEFICIARY_ACCOUNT`
- `ORDERING_DETAILS`
- `ORDERING_TOKENS`
- `BENEFICIARY_DETAILS`
- `BENEFICIARY_TOKENS`
- `ORDERING_TYPE`
- `BENEFICIARY_TYPE`
- `ANOMALY_TYPE`
- `DISCLAIMER`

---

## Data Pipeline (8 GB RAM Workaround)

The full CSV is too large to load into memory on an 8 GB machine. We stream-scan it with **DuckDB**, export small aggregated CSV files, and use those aggregates to generate charts.

```text
Raw CSV (1.5 GB, local only)
        |
        v
DuckDB read_csv (streaming, low memory)
        |
        v
GROUP BY aggregations
        |
        v
agg/*.csv (small, committed)
        |
        v
Charts and dashboards
        |
        v
Dashboard PDF
```

### Reproduce the Aggregates

From the repository root, with the DuckDB CLI available:

```bash
duckdb
```

Run the SQL scripts in `queries/` against the full path to the raw CSV. Generated aggregate files are saved in `agg/`.

### Aggregate Files

| **File**                              | **Covers**                                                   | **Feature Group**    |
|:--------------------------------------|:-------------------------------------------------------------|:--------------------|
| `data_agg/agg_currency_partytype.csv`  | Amount by currency × party type                              | Currency & amounts  |
| `data_agg/agg_routing.csv`             | Transaction type × continuity → participants / duration / FX | Routing             |
| `data_agg/agg_anomaly.csv`             | Anomaly type × party-type signature                          | Anomalies           |
| `data_agg/agg_hour_currency.csv`       | Hour of day × currency                                       | Temporal            |
| `data_agg/agg_amount_bins.csv`         | `log10` amount bins by party type                             | Amount distribution |
| `data_agg/agg_conversion.csv`          | FX steps × transaction type                                  | Routing / FX        |
| `data_agg/agg_fat_finger.csv`          | Amount distribution for fat-finger anomalies                 | Anomalies           |
| `data_agg/agg_duration_hops.csv`       | Duration by participant count                                | Routing             |
| `data_agg/agg_hour_routing.csv`        | Hour of day × routing type                                   | Temporal / Routing  |

### Regenerate an Aggregate (Example)

The following DuckDB query regenerates the currency-by-party-type aggregate:

```sql
COPY (
    SELECT
        CURRENCY_CODE,
        ORDERING_TYPE,
        BENEFICIARY_TYPE,
        COUNT(*) AS payment_count,
        AVG(USD_AMOUNT) AS avg_amount,
        MEDIAN(USD_AMOUNT) AS median_amount
    FROM read_csv(
        'C:/Users/leeya/404-Burnt-Out-Baddies/data_raw/swift_umdac_datathon_2026_fictitious_payment_data.csv'
    )
    GROUP BY
        CURRENCY_CODE,
        ORDERING_TYPE,
        BENEFICIARY_TYPE
) TO 'data_agg/agg_currency_partytype.csv'
  (HEADER, DELIMITER ',');
```

Update the input path to wherever the raw dataset is stored on your machine.

---

## Repository Layout

```text
.
├── README.md
├── .gitignore
├── queries/             # SQL used to build data_agg/*.csv
├── data_raw/            # Full 1.5 GB CSV (local only, ignored by git)
├── data_agg/            # Small aggregated CSVs (committed)
├── notebooks/           # Analysis and chart generation notebooks
│   └── dashboard_generator.ipynb
├── output/              # Final PDFs and rendered pages
│   ├── dashboard_final.pdf
│   └── page*.png        # Intermediate renders (ignored by git)
└── slides/              # Slide deck source and exported PDF
```

---

## Environment Setup (Python / Notebooks)

Create and activate a virtual environment:

```bash
python -m venv .venv
```

**Windows:**

```powershell
.venv\Scripts\activate
```

Install the required packages:

```bash
pip install duckdb pandas numpy matplotlib seaborn plotly jupyter ipykernel
```

Register the notebook kernel:

```bash
python -m ipykernel install --user --name=datathon --display-name "Python (Datathon)"
```

> **Important:** Notebooks should read only the small files in `agg/*.csv`. Do not load the full 1.5 GB dataset into notebook memory.

---

## Working Agreement

- Branch off `main`; open a pull request or announce in Discord before merging.
- Run `git pull` before starting work each day.
- Maintain one notebook per feature group to avoid merge conflicts.
- Every dashboard chart must show the finding against a baseline or comparison group, and include a takeaway title and clear labels.
- Every headline claim must be traceable to a chart or calculation.
- All outputs (PDFs, PNGs) must be saved to the `output/` folder, not the repository root.
- Intermediate files (`page*.png`) are ignored by git and should not be committed.

---

## Data Use & Compliance

- Use only the organiser-provided dataset for primary analysis. Do not use external primary datasets.
- External reference statistics from papers, government reports, or industry reports may be used only to calibrate, justify, or scale findings. Cite these sources in APA style with embedded URLs.
- All submissions must be original.
- Disclose any pre-existing code or tools used.
- AI tools are permitted and do not require a separate declaration, but every method used must be explainable.
- The competition dataset must not be redistributed. It is therefore excluded from this repository.

---

## Project Status

- [x] Environment and DuckDB pipeline set up
- [x] Aggregates built (`agg/`)
- [x] Dashboard PDF completed (12 pages, saved to output/)
- [ ] Slide deck completed (10–20 slides)
- [ ] Hypothesis log and rejected-hypothesis slide drafted
- [ ] Self-check against judging rubric
- [ ] Submission completed before 19 Nov 2026, 12:00 PM
