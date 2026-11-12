# Hospital Operations: Statistical Investigation & Insight Discovery

**AnalystLab Africa

> **Research question:** How do recorded total claim costs vary across encounter classes, and are the differences associated with encounter duration and payer coverage?

An exploratory and inferential analysis of 27,891 hospital encounters (974 patients, 2011–2022) covering data-quality assessment, three formal hypothesis tests, a confounding check, and independent validation of every headline number.

## Key findings

| Finding | Evidence |
|---|---|
| Cost is extremely skewed | Mean $3,640 vs median $279; top 1% of encounters = 14.5% of all cost |
| Classes differ, but the effect is modest | Kruskal–Wallis H = 2,025, p < 0.001, ε² = 0.072; confirmed at patient level |
| Duration is moderately linked to cost in ambulatory/outpatient care | Spearman ρ = 0.48 / 0.39; only ≈ 0.17 in emergency and inpatient |
| About half of encounters show $0 payer coverage | 48.7% overall; 25% even among insured; differs by class (χ² p < 0.001) |
| Most raw class gaps vanish after adjustment | Cluster-robust OLS: only wellness remains significantly different from ambulatory |
| Data-quality red flags | 18.7% of claims don't reconcile to line items; 68 encounters after a recorded death date; fixed 15-minute durations for two classes |

Full detail, assumptions, limitations and recommendations are in [`statistical_report.md`](statistical_report.md).

## Repository structure

```
.
├── analysis.ipynb          # Executed notebook: code, outputs, figures
├── analysis.py             # Same analysis as a plain "# %%" cell script (source of the notebook)
├── build_nb.py             # Re-runs analysis.py and regenerates analysis.ipynb, figures, results.json
├── statistical_report.md   # Full statistical report and recommendations
├── data_quality_summary.md # Data-quality assessment and cleaning decisions
├── results.json            # Headline numbers produced by the notebook
├── figures/                # Figures 1–8 (aggregated; no patient-level information)
├── data/                   # <- put the source CSVs here (git-ignored)
├── requirements.txt
└── .gitignore
```

## How to reproduce

**Requirements:** Python 3.10+.

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Add the data (not included in this repo – see Privacy)
#    Copy these into ./data/ :
#    encounters.csv, patients.csv, procedures.csv, payers.csv, organizations.csv, data_dictionary.csv

# 3a. Interactive: open analysis.ipynb in Jupyter and choose Run All
jupyter notebook analysis.ipynb

# 3b. Or headless: re-execute everything and rewrite the notebook, figures and results.json
python build_nb.py
```

- The `data/` folder must sit **next to `analysis.ipynb`** (the folder Jupyter was launched from). The notebook also checks `.`, `../data` and `..`.
- To keep the CSVs elsewhere, set `HOSPITAL_DATA_DIR`:
  - macOS/Linux: `export HOSPITAL_DATA_DIR=/path/to/csvs`
  - Windows PowerShell: `$env:HOSPITAL_DATA_DIR="C:\path\to\csvs"`
  - Windows CMD: `set HOSPITAL_DATA_DIR=C:\path\to\csvs`
- Runtime is about 1–2 minutes. Bootstrap randomness is seeded (`SEED = 42`).

## Methods at a glance

| Question | Method | Why |
|---|---|---|
| Cost by class (H1) | Kruskal–Wallis + Holm-adjusted Mann–Whitney; patient-clustered bootstrap | Skewed costs, unequal variances and group sizes rule out ANOVA |
| Duration vs cost (H2) | Spearman rank correlation within class | Robust to extreme durations; avoids mixing class and duration effects |
| Zero coverage by class (H3) | χ² test of independence + Cramér's V | Two categorical variables; repeated among insured only |
| Confounding | OLS on ln(1 + cost) with patient-clustered robust SEs | Adjusts for duration, procedures, age, payer status and year |
| Validation | SQL (SQLite) recomputation, hand-coded Kruskal–Wallis, rank-based Spearman, OLS vs `lstsq` | Independent checks of each headline number |

## Privacy and responsible use

- **No patient-level data is published.** The source CSVs are git-ignored. Patient names, addresses and coordinates are never loaded (`usecols`), and no identifiers appear in outputs or figures.
- **No clinical claims.** This is an operations and finance analysis of *recorded* costs. Results show association, not causation.
- The dataset looks simulated and covers a single hospital, so findings should not be generalised to real-world hospital behaviour.
- The `create_hospital_db.sql` setup script (provided by the programme) drops and recreates `hospital_db`. It was **not** used here; run it only in an isolated practice database.

## Tech stack

Python · pandas · NumPy · SciPy · Matplotlib · Seaborn · Jupyter

## Author

Charity Nduati· Data Science· #AnalystLabAfrica
