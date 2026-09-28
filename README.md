# Fitness Tracker Analytics — Fitbit Case Study

A single-file Streamlit app plus a companion EDA notebook, built on a Fitbit
fitness-tracker export (33 participants, 12 Apr – 12 May 2016, public CC0
dataset). Modeled on a Bellabeat/Strava-style wellness case study: merge the
raw exports, analyze usage patterns, and turn that into a customer-facing
dashboard and recommendations.

## What's in this project

| File | What it is |
|---|---|
| `streamlit_app.py` | The Streamlit app — dashboard, participant heatmaps, SQL lab, recommendations, methodology. Reads `daily_merged.csv` and `hourly_merged.csv` from the same folder. |
| `EDA_Analysis.ipynb` | Jupyter notebook reproducing the project's core exploratory charts (weekday patterns, participant heatmaps, activity-category breakdown, hourly calorie burn) with pandas/matplotlib/seaborn, plus a written summary and recommendations. |
| `requirements.txt` | Python dependencies for running the app locally or deploying it. |
| `daily_merged.csv`, `hourly_merged.csv`, `minute_narrow_merged.csv`, `minute_wide_merged.csv`, `heartrate_seconds_merged.csv` | The 19 raw Fitbit export files, merged down to 5 by time grain (outer joins on `Id` + timestamp, no values corrected). Only `daily_merged` and `hourly_merged` are used by the app; the finer-grained files are for further SQL or notebook work (store them gzipped if they are too big for GitHub). |

## Data & methodology (short version)

- **Source:** Fitbit tracker exports distributed via Amazon Mechanical Turk, CC0.
- **Merge:** 19 raw files → 5 files, grouped by grain (daily / hourly / minute-narrow /
  minute-wide / heartrate-seconds), joined with `outer` merges so no source row is
  ever dropped.
- **No correction:** values were not cleaned, recalculated, or filtered — only combined.
  Known caveats (sparse weight logs, partial sleep logs, some zero/low days consistent
  with the tracker not being worn) are left in the data and documented in the app's
  "Data & Methodology" tab, not silently removed.
- **Full detail:** see that tab in the running app.

## Running locally

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

Then open the URL Streamlit prints (usually `http://localhost:8501`).

To open the notebook:

```bash
pip install jupyter matplotlib seaborn pandas
jupyter notebook EDA_Analysis.ipynb
```

(The notebook's charts are already rendered/saved in the file, so you can also
just open it in VS Code, JupyterLab, or GitHub's preview without re-running anything.)

## Deploying the Streamlit app

### Option A — Streamlit Community Cloud (free, easiest)

1. Create a GitHub repo and push these **four files to the repo root** (same folder, not inside a sub-folder):
   `streamlit_app.py`, `requirements.txt`, `daily_merged.csv`, `hourly_merged.csv`.
   (`requirements.txt` must be named exactly that and must list `plotly`, otherwise the app crashes with
   `ModuleNotFoundError: plotly`.)
2. Go to [share.streamlit.io](https://share.streamlit.io) and sign in with GitHub.
3. Click **"New app"**, pick the repo/branch, and set **Main file path** to
   `streamlit_app.py`.
4. Click **Deploy**. Streamlit Cloud installs `requirements.txt` automatically and
   gives you a public `*.streamlit.app` URL — that's the link to share with your
   team/stakeholders.
5. Any future push to that branch redeploys the app automatically.

### Option B — Any other host that runs Python (Render, Railway, an EC2/VM, etc.)

1. Push the same four files to a repo or upload them to the server.
2. Install dependencies: `pip install -r requirements.txt`.
3. Start the app bound to the host's port, e.g.:
   ```bash
   streamlit run streamlit_app.py --server.port $PORT --server.address 0.0.0.0
   ```
4. Most platforms (Render, Railway) auto-detect this as a web service if you give
   them that start command in their dashboard/`Procfile`.

### Option C — Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY streamlit_app.py daily_merged.csv hourly_merged.csv ./
EXPOSE 8501
CMD ["streamlit", "run", "streamlit_app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

```bash
docker build -t fitness-analytics .
docker run -p 8501:8501 fitness-analytics
```

## App structure (tabs)

1. **BI Dashboard** — KPI cards, average steps by weekday, activity-category
   breakdown, calories-vs-active-minutes, calories by hour.
2. **Participant Heatmaps** — steps / sedentary minutes / calories, per
   participant × weekday.
3. **Sleep & Weight** — sleep-duration distribution, logged weight over time.
4. **SQL Analysis** — 12 pre-written SQL queries (window functions, CTEs,
   rankings) against an in-memory SQLite copy of the merged data, plus a free-text
   editor for custom queries.
5. **Recommendations** — customer-facing, tied directly to what the data shows.
6. **Data & Methodology** — the merge approach, caveats, and full column lists.

All charts respect the sidebar filters (date range, participant, weekday) except
the SQL Lab, which queries the full unfiltered tables by design.
