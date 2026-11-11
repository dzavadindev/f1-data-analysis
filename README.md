# F1 Data Analysis

An F1 data pipeline and dashboard built with Python, DuckDB, dbt, and Evidence. It combines a bundled driver and team API snapshot with race results and race control files, then presents two historical analyses: driver incident counts and constructor points projections.

## What it does

1. `loader/load_data.py` reads CSV and TXT files and queries the local API in `raw/api/`, loading both into `db/database.db`.
2. dbt models in `models/` clean the raw tables, build a star schema, and produce incident and constructor prediction reports.
3. Evidence reads the report models through `sources/reports/` and renders the pages in `pages/`.

The incident report counts race control entries marked `NOTED` and `INCIDENT` for each driver. The constructor report estimates end-of-season points from average points earned so far and shows a best-case points total. These projections are simple heuristics based on the supplied historical data, not live forecasts.

## Prerequisites and source data

- Python 3.11 and [Poetry](https://python-poetry.org/docs/#installation)
- Docker with a running daemon
- Node.js 18 or newer and npm
- Bash and `curl` for the loader script

The race CSV and TXT files are **not included** in this repository. Put them in `raw/output/` before running the pipeline. The loader expects race control files ending in `Race_racecontrol.txt`, lap files ending in `Race_laps.csv`, weather files ending in `weather.csv`, and these CSV files:

```text
circuits.csv
constructor_results.csv
driver_results.csv
driver_sprint_results.csv
drivers_teams.csv
lap_times.csv
races_carried_out.csv
```

The driver and team API snapshot is included in `raw/api/api.json`. Without the external CSV and TXT files, a fresh clone cannot regenerate the database or dashboards.

## Run locally

From the repository root, after adding the source data:

```sh
bash loader/run.sh init
npm run dev
```

The `init` run creates the Poetry environment and installs Python packages. The script builds and starts the local API container on port `3000`, loads DuckDB, runs `dbt build` (including its schema tests), refreshes Evidence sources, and removes the API container when it exits. Evidence then starts a local dashboard server; use the URL printed by `npm run dev`.

For later data refreshes, run `bash loader/run.sh` and restart the Evidence dev server. The script expects its database profile in the repository's `profiles.yml` and writes the DuckDB file to `db/database.db`.

To run dbt independently after loading data:

```sh
poetry run dbt build --project-dir . --profiles-dir .
```

## Repository map

| Path | Purpose |
| --- | --- |
| `raw/api/` | Bundled JSON snapshot and local API server. |
| `loader/` | Data ingestion and pipeline runner. |
| `models/staging/`, `models/intermediate/`, `models/star/` | dbt transformations and dimensions/facts. |
| `models/reports/` | Incident counts and constructor points projections. |
| `sources/reports/`, `pages/` | Evidence queries and dashboard pages. |

The source data represents a limited historical snapshot. In particular, the original constructor-results input used for the 2023 example covered only part of that season, so projections should be interpreted with that limitation in mind.
