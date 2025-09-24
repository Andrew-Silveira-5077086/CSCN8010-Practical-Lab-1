# DataStreamVisualization_Workshop

**Use case:** Manufacturing Robot Predictive Maintenance  
Live current readings are streamed from a CSV “controller feed” into a Neon (Postgres) database and visualized as a rolling window chart.  
A lightweight anomaly placeholder (z-score) flags potential maintenance alerts.

## Team
- **Team #**: Group 2
- **Members**: Andrew Silveira - Student #: 5077086

## Dataset
- **Source**: CSV provided with the workshop (see `data/`).
- **License**: Class use (replace with link if public).
- **Description**: Columns like `Axis #1 … Axis #N` plus an optional timestamp.  
  Missing loads are stored as NULL (= “OK null”, not an error).

## Problem & Approach
- **Problem**: We need visibility into robot energy use to spot anomalies and decide when maintenance is required.  
- **Approach**:
  - Stream each CSV row every *TICK_SECONDS* into a rolling window chart.
  - Persist readings to Neon for downstream analytics and auditing.
  - Use a **schema-locked** axis list from `axes_dim` so only expected axes are plotted/ingested.
  - “**OK null**” = no load at that instant (not a data failure).
  - Fixed integer x-axis ticks avoid tick-spacing flip-flop during live updates.

## What’s in the notebook
- **Cell 1** – Notebook config: CSV path (`CSV_PATH`), plotting knobs (`TICK_SECONDS`, `WINDOW_ROWS`), imports.  
- **Cell 2** – DB connection: build SQLAlchemy `engine`, quick server/version sanity print.  
- **Cell 2.1** – SQL helpers: small utilities like `read_sql_df(...)`, `query_value(...)`.  
- **Cell 3** – Schema DDL: create `axes_dim` and `readings_fact`.  
- **Cell 4** – Seed `axes_dim`: insert Axis IDs; maps schema-driven labels.  
- **Cell 5** – Bulk ingest CSV → `readings_fact` (UTC-safe, schema-locked, chunked).  
- **Cell 6a** – `StreamingSimulator` class (exposes `nextDataPoint()`).  
- **Cell 6b** – Streaming playback → live plot + optional Neon ingest.  
- **Cell 7** – Fast sanity queries (latest rows, latest-per-axis, window peaks).  
- **Cell 8–9** – CAT* views + sanity checks.  
- **Cell 10** – Data health & coverage summary.  
- **Cell A** – Setup & Database Connection (reuse engine, notebook options).  
- **Cell B** – Load Streamed Data from DB (verify persistence).  
- **Cell C** – Optional smoothing (1-min resample).  
- **Cell D** – Summary chart (time series per axis).  
- **Cell E** – Per-axis summary (table + bar chart).  
- **Cell 12** – Predictive analytics placeholder (per-axis z-score alerts).  
- **Cell 13A / 13B** – Phrase tests (“latest per axis”, “1-minute peak”).  
- **Cells 11.1–11.2** – Exports (paged CSV + Excel sample).  

## How to run

### 1) Environment
- Python **3.10+** (3.12 OK)
- VS Code + Jupyter (recommended)

```bash
# Create & activate a virtualenv
python -m venv .venv
# Windows PowerShell
. .venv/Scripts/Activate.ps1

# macOS/Linux
# source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

# Configure Neon
Create a .env at repo root (git-ignored):
DATABASE_URL=postgresql+psycopg2://USER:PASSWORD@HOST/neondb?sslmode=require

## Run the notebook
Open `notebook/DataStreamVisualization_workshop.ipynb`, then run in order:

1. **Cell 1** – Configure paths & knobs  
   Set `CSV_PATH`, `TICK_SECONDS`, `WINDOW_ROWS`, plotting options.
2. **Cell 2** – Connect to Neon  
   Builds the SQLAlchemy `engine` (uses `.env` `DATABASE_URL` if present).
3. **Cell 2.1** – Helper functions  
   Convenience helpers like `read_sql_df(...)`, `query_value(...)`.
4. **Cell 3** – Create schema  
   DDL for `axes_dim` and `readings_fact`.
5. **Cell 4** – Seed `axes_dim`  
   Inserts Axis IDs/names; drives schema-locked plotting/ingest.
6. **Cell 5** – (Optional) Bulk ingest CSV → DB  
   UTC-safe, chunked ingest. *Skip if you only want live playback in Cell 6b.*
7. **Cell 6a** – `StreamingSimulator` class  
   Provides `nextDataPoint()` for the driver cell.
8. **Cell 6b** – Live playback + (optional) DB ingest  
   Rolling window chart; batched inserts if `INGEST_TO_DB=True`.  
   *Knobs:* `TICK_SECONDS`, `WINDOW_ROWS`, `INGEST_TO_DB`, `X_AXIS_TICKS=("fixed"|"auto")`, `STACKED`.
9. **Cells 7–10** – Sanity checks, CAT* views, data-health summaries
10. **Cell A** – Setup & Database Connection (reuse engine, notebook options)  
11. **Cell B** – Load Streamed Data from DB (verify persistence)  
12. **Cell C** – Optional smoothing (1-min resample)  
13. **Cell D** – Summary chart (time series per axis)  
14. **Cell E** – Per-axis summary (table + bar chart)
15. **Cell 12** – Predictive analytics placeholder  
    Simple z-score alerts over the recent window.
16. **Cell 13A / 13B** – Phrase tests  
    “Show latest per axis” and “Show 1-minute peak”.
17. **Cells 11.1–11.2** – Exports  
    Paged full CSV + Excel sample (summaries + latest).


# Phrase tests (expected)
“Show latest per axis” (Cell 13A): one row per axis with the newest timestamp.
“Show 1-minute peak” (Cell 13B): top axis by max amps in the last 60s.

# Outputs
  notebook/exports/neon_export_*.csv – full dataset (paged by id)
  notebook/exports/neon_export_summary_*.csv – per-axis summary
  notebook/exports/neon_export_latest20_*.csv – last 20 rows
  notebook/exports/neon_export_*.xlsx – sample readings + summaries

# Folder Structure
CW_100925/
├─ data/
│  └─ RMBR4-2_export_test.csv
├─ notebook/
│  ├─ DataStreamVisualization_Workshop.ipynb
│  └─ exports/                 # generated CSV/XLSX (git-ignored)
├─ .env                        # DATABASE_URL (git-ignored)
├─ requirements.txt
├─ .gitignore
└─ README.md

# Pratical Lab 1

This lab os to simulate a predictive maintenance scenario, where alerts and errors can falg the user with potential failures

![Regression model](./images/axis1.png)
![Redifual model](./images/residual-axis1.png)