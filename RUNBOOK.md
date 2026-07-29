# Job Tracker Runbook

Instructions for the scheduled agent run. Read `config.json` in this folder first.

## Steps, once per run

1. For each site in `config.json` → `sites`:
   - Fetch the site's `url` (use WebFetch, or a browser tool if available) and extract the current list of postings (title, department/institution, location, date if shown, and a link or unique identifier for each row). Use pagination/"Show All" controls if present so you capture the full current listing, not just the first page. NYU's page has a JS bot-challenge on first load — if a fetch returns the challenge/interstitial instead of listings, retry once; if it still fails, note that in the digest and skip diffing NYU this run rather than guessing.
   - Compare against `state/<slug>.json` (an array of previously seen postings, matched by title+link). If the state file doesn't exist yet, treat every current posting as new on this first run, then just save state — don't send a notification flood for a first run.
   - Anything in the current listing not present in the state file is "new" for this run.
   - Overwrite `state/<slug>.json` with the full current listing (this becomes the baseline for next run). Keep entries as `{"title": ..., "org_or_dept": ..., "location": ..., "date": ..., "link": ...}`.

2. If there are zero new postings across all sites, write a one-line entry to today's digest (see below) noting nothing new, and stop. Do not send a push notification for a zero-match run.

3. If there are new postings, use the Google Drive connector to find and read the profile documents listed in `config.json` → `profile_documents` (search by filename in the `drive_folder`; these are not stored in this repo). For each new posting, judge fit:
   - Consider field/discipline match, seniority (predoc/RA/postdoc/junior research scientist level — not tenure-track faculty roles, which don't fit this candidate's career stage), and institution.
   - Weight `research_track` documents primarily since these 3 sites are academic/research boards; consider `quant_marketing_track` for marketing- or data-adjacent research roles.
   - Ignore postings clearly outside scope (tenure-track faculty searches, senior staff/administrative roles, unrelated disciplines like clinical medicine, K-12 teaching, etc.) unless there's a clear reason they'd fit.

4. Write a digest file at `digests/YYYY-MM-DD.md` (use the run date) listing every new posting found, grouped by site, with: title, institution/dept, link, and a one-line fit assessment. Mark strong matches clearly (e.g. `**STRONG MATCH**`).

5. If there is at least one strong match, send exactly one `PushNotification` summarizing the count and the best 1-2 matches by name (keep under 200 characters, per that tool's own limit). If there are new postings but none are strong matches, do not push — the digest file is enough.

6. Commit the updated `state/` and `digests/` files to git with a short message like `chore: job tracker run YYYY-MM-DD`, and **push to origin main**. This is essential: each scheduled run starts from a fresh checkout of this repo, so if the updated state isn't pushed, tomorrow's run won't know what was already seen today and will re-notify on the same postings.

## Notes

- Never scrape LinkedIn or Handshake as part of this task — those require login and prohibit automated access. This runbook is scoped to the 3 public sites in `config.json` only.
- If a site's structure has changed enough that extraction fails, note that in the digest instead of guessing, and skip diffing for that site this run.
