# Engine Health Data Pipeline - Project Plan

## Overview
This project builds a data pipeline that turns raw engine sensor readings into a
predicted remaining useful life (RUL) for every engine, plus alerts for engines
close to failure.

End result: a Dockerized system where raw files are ingested, validated, stored
in PostgreSQL, scored by an XGBoost model, scheduled by Airflow, and shown through
a FastAPI service and a React dashboard.

Dataset note: the data is simulated NASA data, not real-world engine data. The CV and
README will say so.

## Scope and non-goals
Everything is built on dataset subset FD001 first. Other subsets only if time remains.

In scope:
- Ingest, validate, transform and load FD001 into PostgreSQL
- Data-quality checks with a rejected-rows table and a run audit log
- RUL baseline and XGBoost model with engine-level validation
- Airflow DAG running the full pipeline with retries
- FastAPI service and a simple React dashboard
- Tests, Docker Compose, README, and CI

Non-goals:
- Real-time streaming (Kafka), cloud deployment, user accounts
- Deep learning models such as LSTMs
- Subsets FD002 to FD004 (optional extension only)

## Architecture and tech stack
Data moves in one direction, from raw files to the dashboard, and Airflow runs
steps 1 to 5 in order.

1. Ingest: read the raw C-MAPSS text files and copy them unchanged into `raw_readings`.
2. Validate: run data-quality checks; send failing rows to `rejected_rows` with a reason.
3. Transform: drop constant sensors, normalize, and build rolling-window features.
4. Load: upsert cleaned rows into `fact_sensor_readings` so re-runs never duplicate data.
5. Model: train or load the RUL model and write predictions to `fact_predictions`.
6. Serve: FastAPI reads predictions and exposes them; the React dashboard calls the API.

| Layer | Tool | Why |
| --- | --- | --- |
| Language | Python | Pipeline, model, API |
| Data handling | pandas, pandera | Cleaning and data-quality checks |
| Warehouse | PostgreSQL, SQLAlchemy | Star schema and SQL analytics |
| Orchestration | Airflow (Prefect as fallback) | Scheduling, retries, run history |
| ML | scikit-learn, XGBoost | Baseline and RUL model |
| API | FastAPI | Health scores and alerts |
| Dashboard | React | Fleet view and charts |
| Packaging | Docker Compose | One-command run |
| Quality | pytest, GitHub Actions | Tests on every push |

## Data: NASA C-MAPSS, subset FD001
C-MAPSS is a simulated turbofan engine degradation dataset from NASA's Prognostics
Data Repository. In FD001, 100 training engines run until failure, and 100 test
engines stop before failure (files: train_FD001.txt, test_FD001.txt, RUL_FD001.txt).

Each row is one engine at one cycle, with 26 space-separated columns and no header
row. Column names are added in code.

| Columns | Meaning |
| --- | --- |
| 1 | Engine (unit) number |
| 2 | Time in cycles |
| 3 to 5 | Three operating settings |
| 6 to 26 | 21 sensor measurements |

- Training RUL label: the engine's last cycle minus the current cycle.
- Test RUL: a separate RUL file gives the true remaining life at each test engine's
  last recorded cycle.
- Constant sensors: some sensors barely change in FD001. Phase 1 identifies them and
  they are dropped.
- RUL cap: many published approaches cap RUL at about 125 cycles, because early-life
  RUL cannot be known. Test with and without the cap and record the effect.

## Warehouse schema
Seven PostgreSQL tables separate raw data, cleaned facts, model output, and audit
records. Design the schema in Phase 2, before any pipeline code.

| Table | Kind | Holds |
| --- | --- | --- |
| raw_readings | Raw | Ingested rows unchanged, plus source file and run id |
| rejected_rows | Audit | Rows that failed checks, with the reason |
| pipeline_runs | Audit | One row per run: start time, status, rows in and out |
| dim_engine | Dimension | One row per engine and dataset subset |
| dim_sensor | Dimension | Sensor names and whether each is constant |
| fact_sensor_readings | Fact | Cleaned readings, one row per engine and cycle, with RUL label |
| fact_predictions | Fact | Predicted RUL per engine and cycle, with model version and timestamp |

First design choice to log in docs/DECISIONS.md: store the 21 sensors as columns of
a wide fact table, or as one row per sensor reading. Wide is simpler to query and to
feed the model.

## Repo structure
```text
engine-health-pipeline/
  CLAUDE.md              rules for Claude Code
  README.md              architecture, setup, results
  docker-compose.yml     Postgres, Airflow, API, dashboard
  data/raw/              C-MAPSS files (not committed)
  sql/                   schema migrations and analytics queries
  pipeline/              ingest, validate, transform, load
  models/                training, evaluation, saved models
  dags/                  Airflow DAG definitions
  api/                   FastAPI service
  dashboard/             React app
  notebooks/             data exploration
  tests/                 pytest tests
  docs/                  PLAN, CONTEXT, DECISIONS, PROGRESS
  .github/workflows/     CI
```

## Phases
About 3 weeks in 10 phases. Phases 0 to 5 are the minimum viable project; phases 6
to 8 add orchestration, API, and dashboard. Each phase ends with something that runs.

### Phase 0: Setup (day 1)
- Create the repo and folders, a Python virtual environment, and keep CLAUDE.md.
- Write `docker-compose.yml` running PostgreSQL.
- Download the FD001 train, test, and RUL files into `data/raw/`.
- Done when: `docker compose up` starts Postgres and you can connect to it.

### Phase 1: Understand the data (days 2 to 3)
- In a notebook, load the files, add column names, and compute the true RUL for
  training rows.
- Plot sensors against cycle for several engines; find which sensors drift and which
  are constant.
- Done when: you can explain the data and the useful sensors in two minutes.

### Phase 2: Design the warehouse (day 4)
- Write the schema as SQL migration files in `sql/`, covering the seven tables above.
- Log the wide-versus-long fact table choice in docs/DECISIONS.md.
- Done when: all tables exist in Postgres and you can draw the schema from memory.

### Phase 3: Ingest and validate (days 5 to 6)
- Load raw files into `raw_readings`, stamped with a run id and source file.
- Add pandera checks: columns and types, no nulls, plausible ranges, no duplicate
  engine-cycle pairs, cycles increasing without gaps.
- Send failing rows to `rejected_rows` with a reason, never drop them silently.
- Test with deliberately corrupted input.
- Done when: a clean file loads fully, a corrupted file is caught, and tests pass.

### Phase 4: Transform and load (days 7 to 8)
- Drop constant sensors, normalize, and add rolling mean and standard deviation
  features per engine.
- Load into `fact_sensor_readings` with upserts so the load is idempotent.
- Write SQL window-function queries for per-engine sensor trends in `sql/`.
- Done when: running the pipeline twice leaves identical row counts.

### Phase 5: Model (days 9 to 12)
- Build the baseline, then XGBoost, using engine-level splits (see ML approach).
- Write predictions to `fact_predictions` with a model version.
- Done when: XGBoost beats the baseline on held-out engines and the numbers are recorded.

### Phase 6: Orchestration (days 13 to 15)
- Build an Airflow DAG: ingest, validate, transform, load, model, with retries and
  logging to `pipeline_runs`.
- Run Airflow in Docker next to Postgres. If the laptop struggles, switch to Prefect.
- Done when: one trigger runs the whole pipeline and a failed run is visibly retried.

### Phase 7: API (days 16 to 17)
- Endpoints: list engines, one engine's health and RUL trend, an alerts list below a
  RUL threshold, and recent pipeline runs.
- Add tests against a test database.
- Done when: the auto-generated API docs page can query every endpoint.

### Phase 8: Dashboard (days 18 to 19)
- React page with a fleet table sorted by risk, a chart of one engine's sensors and
  predicted RUL, and an alerts panel.
- Keep it simple; the data engineering is the point.
- Done when: one page shows which engines need attention.

### Phase 9: Polish and CV (days 20 to 21)
- README with architecture diagram, setup steps, baseline-versus-XGBoost results, and
  a screenshot or demo clip.
- GitHub Actions running tests on every push; a limitations section stating the data
  is simulated.
- Update the CV with real numbers (see the CV checklist below).
- Done when: a stranger can clone the repo and run it from the README.

## ML approach
The model predicts RUL, a number, so this is a regression problem; the key rule is to
validate by engine, never by row.

1. Baseline first: predict the average RUL, or fit a simple linear model, so there is
   a number to beat.
2. XGBoost: a gradient-boosted tree model that works well on tabular data and trains
   fast without a GPU. Inputs are the kept sensors plus the rolling-window features.
3. Grouped validation: use GroupKFold grouped by engine id. Cycles from one engine must
   never appear in both training and validation sets, or the score is inflated by
   leakage.
4. Metrics: report RMSE and the NASA scoring function, which penalizes late predictions
   more than early ones.
5. Explain it: record feature importance and write 3 to 4 sentences on what the model
   relies on.
6. Version it: save the trained model with a version tag and store that tag in
   `fact_predictions`.

Record the baseline and XGBoost numbers in the README exactly as measured.

## Learning plan after the build
Claude Code builds first; then Rijul learns the project well enough to defend it in an
interview. Each step makes him produce something.

1. Walkthrough: explain the repo as if he had to build it from scratch, phase by phase,
   using docs/DECISIONS.md and the git history.
2. Run it himself: start the stack from the README, run the pipeline twice, and read the
   key files: validation checks, the load step, the grouped split, and the DAG.
3. Rebuild three pieces without looking: the validation checks, the idempotent load, and
   the engine-level train/validation split. Claude Code acts as a tutor here, not a writer.
4. Break it on purpose: feed a corrupted file, split by row instead of engine, and kill a
   task mid-run. Note what happens and why.
5. Mock interview: quiz him with 15 questions about the repo, one at a time, and grade
   each answer.
6. One-page explainer: he writes his own two-minute walkthrough from raw data to dashboard.

## Risks and fallbacks
| Risk | Fallback |
| --- | --- |
| Time runs out before the project is done | Stop after Phase 5; pipeline, warehouse, and model are already a complete CV project |
| Airflow is too heavy for the laptop or slow to set up | Switch to Prefect, or a plain scheduled script with run logging, and say so honestly |
| XGBoost does not clearly beat the baseline | Report the real numbers; tune features and the RUL cap, and document what was tried |
| Score looks too good | Check for leakage: confirm every split is by engine id |
| Claude Code produces code Rijul cannot explain | Run the learning plan before putting any claim on the CV |
| Job description lists a different data stack | Keep the design, swap tools where cheap, and mention the concepts that transfer |

## CV update checklist
The CV lists the project as ongoing with no numbers, and it stays that way until these
are done.

- [ ] Pipeline runs end to end from a clean clone using the README
- [ ] Rijul has run the learning plan and can explain every phase
- [ ] Baseline and XGBoost results come from his own run, with exact RMSE figures
- [ ] Tools named on the CV (Airflow, XGBoost, pandas, FastAPI, React, Docker) are all used in the repo
- [ ] README states that the data is simulated NASA data
- [ ] CV bullets use the real numbers; "Ongoing" is replaced with the final description
- [ ] Skills section adds pandas, XGBoost, and Airflow only after they were used
- [ ] GitHub repo link added to the project entry
