# Project Memory — priya_global_jobs

Priya's job board for **English-speaking countries outside Finland**, with a local-LLM-only scraper.
Created 2026-09-27 as a sibling of [../priya_jobs](../priya_jobs/memory.md) (her Finland + remote
board). The site list comes from a read-only probe of ~45 boards that day (details in
priya_jobs/memory.md, "Trial: English-speaking countries").

## Major Features
1. **Global job scraping:** LinkedIn in 10 English-speaking countries plus 9 national job boards (UK, IE, CA, AU, ZA, MT), 12 role keywords.
2. **Local-LLM screening only:** every job is reviewed by the shared LM Studio server against `job_requirements.md` — no cloud LLMs.
3. **Dashboard:** Firebase Hosting at https://priya-global-jobs.web.app, cross-linked with the Finland board, data served from GitHub Pages.

## Scope decisions (user, 2026-09-27)
- Countries: UK, Ireland, Canada, Australia, New Zealand, Singapore, India, South Africa, Malta,
  Hong Kong. Any work model; she's open to relocating. Never the US.
- A posting that explicitly says no visa sponsorship / must already have the right to work there
  is a "no" (she only holds a Finnish permit). Silent postings are judged normally.
- The LLM reviews every scraped job (no title pre-filter), even though local review is slow.
- Lowest priority on the local LLM: OpenClaw > manju > vineeth > priya (Finland) > priya-global.

## Infrastructure
- Folder `C:\Users\vinee\priya_global_jobs`, GitHub `vinchess1989/priya-global-jobs-dashboard`
  (public, Pages from `main`), Firebase project `priya-global-jobs` (Firestore `eur3`, Hosting).
- Firestore placeholder docs `shared_state/job_status` and `shared_state/re_review_request` were
  created on setup (same create-vs-update rules gotcha as priya_jobs).
- venv uses LM Studio's bundled CPython 3.11 like the siblings, but has Playwright 1.63 (newer
  than the siblings) — it needed its own `playwright install chromium`.
- Scheduled Task `PriyaGlobalJobsLocalLLMOrchestrator` → `orchestrator.py` → `scraper.py`.
  Restarting: stop the task AND kill the child `scraper.py` (single-instance lock).
- `LOCAL_LLM_ENDPOINT`/`LOCAL_LLM_MODEL` come from the Windows user env vars (shared).

## Scraper specifics (vs priya_jobs)
- `SITE_JOB_URL_PATTERNS` + `parse_known_board`: on the listed boards only links matching the
  board's job-detail regex are kept, with query/`;jsessionid` stripped. Without it, town filters
  (`/jobs/<kw>/in-<town>`), menus and categories got through, and PNet (`-inline.html`) and
  CareerJunction (`-job-NNN.aspx`) jobs were dropped. Job Bank wraps the whole card in one link,
  so the title comes from `.noctitle` (`_anchor_title`).
- LinkedIn: 1 page per country/keyword and `LINKEDIN_DELAY_SECONDS` (8s) before each request —
  the probe got HTTP 429 after ~60 quick requests. Malta needs `geoId=100961908`:
  `location=Malta` resolves to Malta, Ohio.
- Cross-board dedupe: `_dedupe_key` compares LinkedIn jobs by numeric ID (the same job appears
  as fi./uk./mt.linkedin.com) and skips anything already in `..\priya_jobs\jobs.json`.
- Review queue: never-evaluated first, newest `added_at` first within that (the Finland board
  uses file order).
- Job Bank's keyword search is loose (a "Release Manager" search returns admin jobs), and
  JobsInMalta ignores keywords entirely — both cost local-LLM time on irrelevant jobs.

## Setup status (2026-09-27)
- Verified: all 9 boards + LinkedIn Malta return clean job links (live parser test); a test
  scrape added 15 LinkedIn UK DevOps jobs (still `pending`); both dashboards are deployed and
  cross-linked; GitHub Pages serves `jobs.json`.
- **Not yet verified: the local-LLM review step.** The first test review was stopped because
  another session was benchmarking a model on LM Studio. First real run: the scheduled task at
  08:00 on 2026-09-28. Check for `LLM: local/...` verdicts in `logs/scraper_*.log`, and that the
  "SC Cleared" test job (sponsorship/clearance rule) comes back "no".
- The Firebase Console "Google sign-in" provider is not enabled yet (manual step). Viewing works
  without it; the dashboard's write actions (applied/feedback) need it.
- The first Firestore database was accidentally created in `nam5`; it was deleted and recreated in
  `eur3`. A deleted `(default)` ID can be reused only after ~5 min.

## Known blocked boards (don't re-add without a new approach)
Indeed (all countries, Cloudflare), Seek AU/NZ, JobStreet SG, JobsDB HK (Cloudflare), Naukri,
Foundit (Access Denied), Adzuna (403/429), CTgoodjobs, KeepMePosted (captcha), Eluta, Careers24,
MyCareersFuture (JS-only), JobsPlus MT (404), Guardian Jobs / Trade Me (ignore keywords).

---
Last updated: 2026-09-27
