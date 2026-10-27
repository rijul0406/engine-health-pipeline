# Prompts to paste into Claude Code

Not read by Claude Code automatically. These are for you to copy and paste.

## 1. First session (Phases 0 and 1)
Read CLAUDE.md, docs/CONTEXT.md and docs/PLAN.md. Then do Phase 0 and Phase 1 from
the plan. Set up the folder structure, docker-compose.yml with PostgreSQL, a Python
virtual environment with a requirements.txt, and a notebook that loads the FD001 files
(they are in data/raw/; if they are missing, tell me exactly what to download and
where to put it). Commit after each working step, update docs/PROGRESS.md, and log
any design choices in docs/DECISIONS.md. Stop at the end of Phase 1 and summarize.
Ask me before any choice that needs my input.

## 2. Next chunk (Phases 2 to 4)
Continue from docs/PROGRESS.md. Do Phases 2, 3 and 4 from docs/PLAN.md. Follow the
rules in CLAUDE.md: idempotent loads, tests for every validation check including bad
input, small commits, and decisions logged. Stop at the end of Phase 4 and summarize
what to check.

## 3. Model (Phase 5)
Continue from docs/PROGRESS.md. Do Phase 5. Build the baseline first, then XGBoost,
with GroupKFold by engine id. Report RMSE and the NASA score. Do not invent numbers;
report only what the code produces, and show me how to reproduce them.

## 4. Orchestration, API, dashboard (Phases 6 to 8)
Continue from docs/PROGRESS.md. Do Phases 6, 7 and 8. If Airflow is too heavy to run
here, tell me and propose the Prefect fallback rather than faking it.

## 5. Polish (Phase 9)
Do Phase 9: README with architecture diagram, setup steps, measured results and a
limitations section stating the data is simulated. Add GitHub Actions to run tests.

## After the build: learning prompts
- Explain this repo as if I had to build it from scratch, phase by phase. Use
  docs/DECISIONS.md and the git history. Assume I know Python, SQL, FastAPI, React and
  Docker basics but am new to Airflow, pandas pipelines and XGBoost.
- Act as a tutor, not a writer: I will rebuild the validation checks myself. Give me
  hints and review my code, but do not write it for me.
- Ask me 15 interview questions about this repo, one at a time, and grade each answer.
- Help me break the project on purpose (corrupt input, split by row, kill a task
  mid-run) and explain what happens and why.
