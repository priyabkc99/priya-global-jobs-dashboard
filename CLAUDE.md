# CLAUDE.md

Instructions for any Claude Code session working in this repo.

## Read and maintain `memory.md`

`memory.md` (this directory) holds durable, cross-session project knowledge. Read it at the
start of any nontrivial work here. Update it whenever you learn something a future session would
need — a new gotcha, a fixed bug with a non-obvious cause, a changed architecture/config.

## What this project is

Priya's **global** job board: jobs in English-speaking countries outside Finland (UK, IE, CA, AU,
NZ, SG, IN, ZA, MT, HK), any work model. It is a sibling of `..\priya_jobs` (her Finland + remote
board), cloned from it on 2026-09-27, but fully independent: own GitHub repo
(`vinchess1989/priya-global-jobs-dashboard`), own Firebase project (`priya-global-jobs`), own venv,
own Scheduled Task (`PriyaGlobalJobsLocalLLMOrchestrator`).

Differences from `priya_jobs` that must be kept:
- **Local LLM only** — `_call_llm_with_fallback` never calls Groq/Gemini. Don't add cloud keys.
- **Lowest local-LLM priority** — defers to the manju, vineeth and priya priority locks; claims none.
- **No duplicates with the Finland board** — `scrape_all_jobs` skips URLs (LinkedIn: job IDs)
  already in `..\priya_jobs\jobs.json`.
- **Strict per-board link patterns** — `SITE_JOB_URL_PATTERNS` in `scraper.py`.

## Parity

Structural fixes in `priya_jobs/scraper.py` or its dashboard (git handling, review loop, dashboard
UI) usually apply here too, and vice versa. Site lists, `job_requirements.md` location rules and
the LLM provider setup are deliberately different — don't sync those.

## Out of scope (for now)

The resume/apply skills (`tailor-resume`, `fill-form`, `find-apply-link`, `mark-job-deleted`) and
their scripts were not ported. `priya_jobs` has them; port separately if wanted.
