# CW_100925 – Trend Analysis

Sensor CSV → Postgres (Neon) → sanity checks → live plots → exports.

## Contents
- **notebook/trendanalysis.ipynb** – main workflow (cells 1–11)
- **data/** – input CSV(s)
- **notebook/exports/** – generated CSV/XLSX (git-ignored)

## Prerequisites
- Python 3.10+ (3.12 works great)
- Neon Postgres database (connection string)
- VS Code + Jupyter (recommended)

## Setup

```bash
# 1) Create & activate a virtualenv
python -m venv .venv
# Windows PowerShell
. .venv/Scripts/Activate.ps1
# macOS/Linux
# source .venv/bin/activate

# 2) Install deps
pip install -r requirements.txt
```
## Configure secrets
Create .env at the repo root (this file is git-ignored):
DATABASE_URL=postgresql+psycopg://neondb_owner:<YOUR_PASSWORD>@ep-cold-haze-adxeexvl.c-2.us-east-1.aws.neon.tech/neondb?sslmode=require

## Execution
Open notebook/trendanalysis.ipynb and run top → bottom:
  * Cell 1 – CSV settings & imports
  * Cell 2 – DB connection (prints server version)
  * Cell 2.1 – SQL helpers
  * Cell 3–4 – schema + seed (axes_dim, readings_fact)
  * Cell 5 – CSV → DB ingest (vectorized; UTC-safe)
  * Cell 6 – live plot from CSV (optional DB ingest off by default)
  * Cell 7–10 – sanity checks and summaries
  * Cell 11 – exports to notebook/exports (chunked CSV + Excel sample)

## Typical outputs
  * notebook/exports/neon_export.csv – full dataset (chunked)
  * notebook/exports/neon_export_summary.csv
  * notebook/exports/neon_export_latest20.csv
  * notebook/exports/neon_export.xlsx – small sample + summaries

## Project layout  
CW_100925/
├─ .venv/                   # virtual environment (git-ignored)
├─ data/
│  └─ RMBR4-2_export_test.csv
├─ notebook/
│  ├─ trendanalysis.ipynb
│  └─ exports/              # generated CSV/XLSX (git-ignored)
├─ .env                     # DB URL (git-ignored)
├─ .gitignore
├─ README.md
└─ requirements.txt
