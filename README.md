
# 404-Burnt-Out-Baddies — UMDAC Datathon 2026 (Preliminary Round)

Reverse-engineering the business rules behind Swift's artificial cross-border payment dataset.

- **Preliminary Round:** 13–19 Nov 2026
- **Submission Deadline:** 19 Nov 2026, 12:00 PM
- **Deliverables:** Dashboard PDF + Slide deck PDF

---

## Team

| Member | GitHub | Owns (feature groups) |
|---|---|---|
| *Aleeya Nazirah binti Jamil* | [@leeya068](https://github.com/leeya068) | Currency & amounts; anomalies |
| *Raja Athira Sofea binti Raja Azlan* | @handle | Routing / BIC path / duration / transaction type |
| *Alliyana binti Abdul Rahman* | @handle | Party details & tokens; party type; temporal patterns |

> **Team rule:** No member replacement after registration. Each member owns their feature groups end-to-end (analysis → dashboard → slides).

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

| File | Covers | Feature Group |
|---|---|---|
| `agg/agg_currency_partytype.csv` | Amount by currency × party type | Currency & amounts |
| `agg/agg_routing.csv` | Transaction type × continuity → participants / duration / FX | Routing |
| `agg/agg_anomaly.csv` | Anomaly type × party-type signature | Anomalies |
| `agg/agg_hour_currency.csv` | Hour of day × currency | Temporal |
| `agg/agg_amount_bins.csv` | `log10` amount bins by party type | Amount distribution |
| `agg/agg_conversion.csv` | FX steps × transaction type | Routing / FX |

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
        'C:/Users/leeya/404-Burnt-Out-Baddies/swift_umdac_datathon_2026_fictitious_payment_data.csv'
    )
    GROUP BY
        CURRENCY_CODE,
        ORDERING_TYPE,
        BENEFICIARY_TYPE
) TO 'agg/agg_currency_partytype.csv'
  (HEADER, DELIMITER ',');
```

Update the input path to wherever the raw dataset is stored on your machine.

---

## Repository Layout

```text
.
├── README.md
├── .gitignore
├── queries/             # SQL used to build agg/*.csv
├── agg/                 # Small aggregated CSVs (committed)
├── notebooks/           # Per-person analysis; not run on the full dataset
│   ├── currency_amounts.ipynb
│   ├── routing.ipynb
│   └── party_temporal.ipynb
├── dashboard/           # Chart sources and exported dashboard PDF
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
- [ ] Rules reconstructed for each feature group
- [ ] Dashboards completed (5–15)
- [ ] Slide deck completed (10–20 slides)
- [ ] Self-check against judging rubric
- [ ] Submission completed before **19 Nov 2026, 12:00 PM**
