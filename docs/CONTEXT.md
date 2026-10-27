# Project context

Background from the planning conversation. Read this so you understand why the
project exists and what matters about it.

## Who and why
- The owner is Rijul, a third-year B.Tech Mathematics and Computing student at
  MIT Manipal (batch of 2028).
- Career target: SDE and Forward-Deployed Engineer roles.
- This project is a data-engineering portfolio project built around data
  pipelines, SQL, ML on tabular data, and engine/maintenance data.
- Technical interviews go through the CV and projects, so Rijul must be able to
  explain every part of this project.

## Why this project
- For engine and equipment makers, uptime and predictive maintenance are core
  to the business, which makes this a realistic industrial data problem.
- Rijul's CV already shows web development (FastAPI, React, PostgreSQL, Docker),
  SQL, and ML research (glaucoma detection). It lacked data-engineering work:
  pipelines, data-quality checks, orchestration, and ML on tabular data.
- This project replaces an unstarted project (CareDesk AI) on the CV. On the CV
  it is listed as "Engine Health Data Pipeline - Predictive Maintenance",
  currently "Ongoing", with no numbers.

## How we agreed to work
- Claude Code builds the project phase by phase; Rijul then learns it
  thoroughly afterwards.
- Because he will learn from the repo, DECISIONS.md and PROGRESS.md must be kept
  up to date as you go, and code should be readable with comments on the why.
- After the build, Rijul will ask for an explanation as if he had to build it
  from scratch, then rebuild a few pieces himself, break things on purpose, and
  run a mock interview. See "Learning plan after the build" in docs/PLAN.md.

## Honesty rules (important)
- The dataset is simulated NASA data, not real-world engine data. Always say so.
- Never invent metrics. Every number on the CV or in the README must come from
  running the code.
- Do not claim a tool on the CV (Airflow, XGBoost, pandas) that is not used in
  the repo.
- If a tool becomes impractical (for example Airflow is too heavy), say so
  plainly and switch to the fallback in docs/PLAN.md instead of faking it.

## Constraints
- Time: roughly 3 weeks of part-time work alongside classes; the deadline may
  come early, so Phases 0 to 5 must form a complete, presentable project on their
  own.
- Runs on a personal computer with Docker Desktop. Keep resource use modest.
- Work in a public GitHub repo: rijul0406/engine-health-pipeline.
