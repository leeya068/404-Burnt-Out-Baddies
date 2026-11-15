# 404-Burnt-Out-Baddies — UMDAC Datathon 2026 (Preliminary Round)

Reverse-engineering the business rules behind Swift's artificial cross-border payment dataset.

- **Preliminary Round:** 13–19 Nov 2026
- **Submission Deadline:** 19 Nov 2026, 12:00 PM
- **Deliverables:** Dashboard PDF + Slide deck PDF

---

## Team

| Member | GitHub | Owns (feature groups) |
|---|---|---|
| *Aleeya Nazirah binti Jamil* | @leeya068 | Currency & amounts; anomalies |
| *Raja Athira Sofea binti Raja Azlan* | @handle | Routing / BIC path / duration / transaction type |
| *Alliyana binti Abdul Rahman* | @handle | Party details & tokens; party type; temporal patterns |

> Rule: no member replacement after registration. Each member owns their feature groups end-to-end (analysis → dashboard → slide).

---

## Competition Summary

The task is to **reverse-engineer the rules used to generate the dataset**. The data is artificial, built by distilling real payments traffic into business logic. We must reconstruct as much as possible of the causal logic, relationships, and business knowledge behind the data.

**Key principle:** A well-argued partial or negative result scores higher than a lucky guess. State rules as:

> *When [condition], [effect] by [magnitude], covering [scope].*

Separate observation from interpretation. Document rejected hypotheses.

### Judging Weights

| Criterion | Weight |
|---|---|
| Insight Quality | 45% |
| Interpretation & Economic Reasoning | 12% |
| Evidence, Method & Data Integrity | 23% |
| Communication | 20% |

### Deliverables

1. **Dashboard PDF** — 5–15 dashboards, one per feature-type group. Each dashboard states the reconstructed rule(s) clearly + at least one chart showing the rule in the data.
2. **Slide deck PDF** — 10–20 slides. Must include introduction, methodology, and max one slide per feature group restating the rule, referencing the supporting chart, and explaining the real-world economic behaviour.

---

## Dataset

- **File:** `swift_umdac_datathon_2026_fictitious_payment_data.csv` (1.5GB, **not committed** — see `.gitignore`)
- Artificial cross-border payments; tabular, timestamped representation of a directed temporal pseudograph.
- **Columns:** `EPOCH_SECONDS`, `CURRENCY_CODE`, `USD_AMOUNT`, `ORDERING_BIC8`, `BENEFICIARY_BIC8`, `BIC8_PATH`, `TOTAL_DURATION`, `CONTINUITY_QFLAG`, `CURRENCY_CONVERSION_COUNT`, `PARTICIPANT_COUNT`, `TRANSACTION_TYPE_CODE`, `ORDERING_ACCOUNT`, `BENEFICIARY_ACCOUNT`, `ORDERING_DETAILS`, `ORDERING_TOKENS`, `BENEFICIARY_DETAILS`, `BENEFICIARY_TOKENS`, `ORDERING_TYPE`, `BENEFICIARY_TYPE`, `ANOMALY_TYPE`, (`DISCLAIMER`).

---

## Pipeline (8GB RAM Workaround)

The full CSV is too large to load into memory on an 8GB machine. We stream-scan it with **DuckDB** and export small aggregated CSVs, then chart those.

raw CSV (1.5GB, local only)
└─ DuckDB read_csv (streaming, low memory)
└─ GROUP BY aggregations → agg/*.csv (small, committed)
└─ charts / dashboards → PDF


### Reproduce the Aggregates

```bash
# from repo root, DuckDB CLI available
duckdb

Run the SQL in queries/ against the full path of the raw CSV. Outputs land in agg/.

File	Covers	Feature Group
agg/agg_currency_partytype.csv	amount by currency × party type	Currency & amounts
agg/agg_routing.csv	transaction type × continuity → participants/duration/FX	Routing
agg/agg_anomaly.csv	anomaly type × party-type signature	Anomalies
agg/agg_hour_currency.csv	hour-of-day × currency	Temporal
agg/agg_amount_bins.csv	log10 amount bins by party type	Amount distribution
agg/agg_conversion.csv	FX steps × transaction type	Routing / FX
Regenerate an Aggregate (Example)

COPY (
    SELECT CURRENCY_CODE, ORDERING_TYPE, BENEFICIARY_TYPE,
           COUNT(*) AS payment_count,
           AVG(USD_AMOUNT) AS avg_amount,
           MEDIAN(USD_AMOUNT) AS median_amount
    FROM read_csv('C:/Users/leeya/404-Burnt-Out-Baddies/swift_umdac_datathon_2026_fictitious_payment_data.csv')
    GROUP BY CURRENCY_CODE, ORDERING_TYPE, BENEFICIARY_TYPE
) TO 'agg/agg_currency_partytype.csv' (HEADER, DELIMITER ',');

Repo Layout
.
├── README.md
├── .gitignore
├── queries/            # SQL used to build agg/*.csv
├── agg/                # small aggregated CSVs (committed)
├── notebooks/          # per-person analysis (not executed on full data)
│   ├── currency_amounts.ipynb
│   ├── routing.ipynb
│   └── party_temporal.ipynb
├── dashboard/          # chart sources + exported dashboard PDF
└── slides/             # slide deck source + exported PDF

Environment (Python / Notebooks)
python -m venv .venv
# Windows
.venv\Scripts\activate
pip install duckdb pandas numpy matplotlib seaborn plotly jupyter ipykernel
python -m ipykernel install --user --name=datathon --display-name "Python (Datathon)"

Notebooks read only agg/*.csv (small), never the full 1.5GB file.

Working Agreement
Branch off main; open a PR or announce in Discord before merging.

git pull before starting work each day.

One notebook per feature group to avoid merge conflicts.

Every chart in a dashboard must show the finding against a baseline or comparison group, with a takeaway title and labels.

Every headline claim must be traceable to a chart or calculation.

Data Use & Compliance
Only the organiser-provided dataset is used for primary analysis. No external primary datasets.

External reference statistics (papers, government/industry reports) are allowed only to calibrate/justify/scale, cited in APA with embedded URLs.

All submissions original; pre-existing code/tools disclosed; AI tools permitted (no separate declaration) but every method must be explainable.

Competition dataset must not be redistributed — hence excluded from this repo.

Status
☑ Environment + DuckDB pipeline
☑ Aggregates built (agg/)
□ Rules reconstructed per feature group
□ Dashboards (5–15)
□ Slides (10–20)
□ Self-check against rubric + submit (before 19 Nov, 12:00 PM)