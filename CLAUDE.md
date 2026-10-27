# Engine Health Data Pipeline

Goal: ingest NASA C-MAPSS FD001 engine sensor data, validate it and store it in
PostgreSQL (star schema), predict remaining useful life (RUL) with XGBoost,
orchestrate everything with Airflow, and serve results via FastAPI + React.
The data is simulated NASA data, not real-world engine data. Say so in the README.

## Read first, every session
- docs/CONTEXT.md  - why this project exists, who it is for, constraints
- docs/PLAN.md     - the phases, schema, ML approach, done-when criteria
- docs/DECISIONS.md and docs/PROGRESS.md - what has been decided and built so far

## Stack
Python, pandas, pandera, PostgreSQL, SQLAlchemy, XGBoost, scikit-learn, Airflow
(Prefect as fallback), FastAPI, React, Docker Compose, pytest, GitHub Actions.

## Rules
- Work through the phases in docs/PLAN.md in order. After each phase, commit and
  add a short summary of what was built and why to docs/PROGRESS.md. Keep going
  unless blocked or a choice needs my input.
- Log every non-obvious design choice in docs/DECISIONS.md: what was chosen,
  what was rejected, and why.
- Make small commits, one per working step, with clear messages.
- Write tests as you go, including a bad-input test for each validation check.
- Loads must be idempotent: running the pipeline twice must not duplicate rows.
- Validate model performance by engine (GroupKFold on engine id), never by row.
- Never invent metrics. Report only numbers produced by running the code.
- Prefer simple, readable code over clever code; comment the why, not the what.
- Keep secrets out of git. Use .env (ignored) and commit only .env.example.

## About me (for explanations)
I am a third-year student. I know Python, SQL, FastAPI, React and Docker basics,
and I have trained deep-learning models in research. I am NEW to Airflow,
pandas-based data pipelines, and XGBoost. When I ask you to explain something,
explain it from scratch as if I had to rebuild it myself, and say why each
choice was made.
