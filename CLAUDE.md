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

## Resume/apply skills (ported 2026-09-29)

`tailor-resume`, `fill-form`, `find-apply-link` and `mark-job-deleted` are in `.claude/commands/`,
copied from `priya_jobs` with their helpers (`make_resume.py`, `html_to_pdf.py`,
`job_status_store.py`, `sync_resume_links.py`, `upload_resume_links.py`, `scrape_application.py`,
`find_repos.py`, `site_patterns.json`), repointed at the `priya-global-jobs` Firebase project /
`priya-global-jobs-dashboard` repo. They run from this repo root on Priya's PC and share her private
resume repo (`vinchess1989/priya-jobs-private`, `Resumes/<job_id>/`) with the Finland board — job
IDs never collide because the scraper skips URLs already on the Finland board.

Global-specific rules that must NOT be synced back from `priya_jobs`: cover letters never claim
right to work or "no sponsorship needed" (her permit is Finland-only) and drop the Finnish-PR /
learning-Finnish lines; forms answer right-to-work "No — would require visa sponsorship".
Structural fixes to these skills/scripts in `priya_jobs` usually DO apply here.
