# Job Tracker

Watches public academic/research job boards for new postings and notifies you when something looks like a fit, based on your resume/CV/research statements.

Currently configured for:
- NYU Faculty and Researcher Careers
- NBER Research Assistant Positions (not at NBER)
- Columbia Academic Search and Recruiting

See `config.json` for the exact URLs.

## How it works

A scheduled Claude cloud agent (a "routine" in [claude.ai/code](https://claude.ai/code)) runs on a cron schedule, following the steps in `RUNBOOK.md`:

1. Fetches current listings from each site in `config.json`.
2. Diffs against `state/<slug>.json` to find postings not seen on a previous run.
3. If there are new postings, reads your profile documents (via the Google Drive connector — see below) and scores each new posting for fit.
4. Writes a dated digest to `digests/YYYY-MM-DD.md`.
5. Sends a push notification only for strong matches.
6. Commits and pushes the updated state/digest back to this repo, so tomorrow's run knows what's already been seen.

## Why this repo is safe to be public

This repo only ever contains: site URLs, the matching logic/instructions, and the *history* of public job postings (titles, links, dates) plus the tracker's own generated notes about them. It never contains your resume, CV, or any personal document — those are read live from Google Drive at run time and are never written to disk in this repo or committed.

## Setting this up for yourself

1. **Fork or clone this repo.**
2. **Put your resume/CV/research statements in Google Drive** (any folder — just note the folder name).
3. **Edit `config.json`:**
   - `sites`: the job boards you want tracked. Must be public, no-login pages — do not point this at LinkedIn, Handshake, or any site that requires authentication; both prohibit automated scraping in their terms of service.
   - `profile_documents`: your Drive folder name and the filenames of your resume/CV/statements. Add or rename the "track" categories to match how you organize your own materials.
4. **Edit `RUNBOOK.md`** if your fit criteria differ (e.g. seniority level, disciplines to ignore) — it's plain instructions the agent follows each run, so just edit the prose.
5. **On [claude.ai](https://claude.ai):**
   - Settings → Customize → Connectors: connect **GitHub** (grant it access to your fork) and **Google Drive**.
   - Go to [claude.ai/code/routines](https://claude.ai/code/routines) → New routine → select your fork as the repository → attach both connectors → set a daily schedule → point the instructions at `RUNBOOK.md` in this repo.

## Notes

- Never point this at a site that requires login or prohibits automated access (LinkedIn and Handshake both do) — it's scoped to public, no-login boards only.
- `state/` and `digests/` are committed to git so history survives between runs (each scheduled run is a fresh, isolated checkout with no persistent disk of its own).
