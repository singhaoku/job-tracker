# Job Tracker

Watches 3 public academic/research job boards for new postings and notifies Sing-Hao when something looks like a fit, based on resume/CV/research statements.

## Sites tracked

- NYU Faculty and Researcher Careers
- NBER Research Assistant Positions (not at NBER)
- Columbia Academic Search and Recruiting

See `config.json` for URLs and profile document paths.

## Why not LinkedIn/Handshake

Both require login and their terms of service prohibit automated scraping/bots — this tracker is scoped to public, no-login job boards only.

## How it works

A scheduled Claude agent runs daily, following `RUNBOOK.md`:
1. Fetches current listings from each site.
2. Diffs against `state/<slug>.json` to find postings not seen before.
3. Scores new postings against the resume/CV/research statements referenced in `config.json` (kept outside this repo — see below).
4. Writes a dated digest to `digests/`.
5. Sends a push notification for strong matches only.

## Profile documents

Resume, CV, and research statements are **not stored in this repo**. The scheduled agent reads them via the Google Drive connector (see `config.json` → `profile_documents` for filenames and folder). This repo is public, so no personal documents belong here.

## Repo scope

This repo tracks only public job-posting metadata (titles, links, dates) and the tracker's own logic — no personal documents, no credentials.
