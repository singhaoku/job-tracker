# Job Tracker Runbook

Instructions for the scheduled agent run. Read `config.json` in this folder first.

## Steps, once per run

1. For each site in `config.json` → `sites`:
   - Fetch the site's `url` (use WebFetch, or a browser tool if available) and extract the current list of postings (title, department/institution, location, date if shown, and a link or unique identifier for each row). Use pagination/"Show All" controls if present so you capture the full current listing, not just the first page. NYU's page has a JS bot-challenge on first load — if a fetch returns the challenge/interstitial instead of listings, retry once; if it still fails, note that in the digest and skip diffing NYU this run rather than guessing.
   - Compare against `state/<slug>.json` (an array of previously seen postings, matched by title+link). If the state file doesn't exist yet, treat every current posting as new on this first run, then just save state — don't send a notification flood for a first run.
   - Anything in the current listing not present in the state file is "new" for this run.
   - Overwrite `state/<slug>.json` with the full current listing, unfiltered (this becomes the baseline for next run — keep every posting here regardless of the recency window below). Keep entries as `{"title": ..., "org_or_dept": ..., "location": ..., "date": ..., "link": ...}`.

2. Apply the recency filter (`config.json` → `recency_window_days`, currently 7) to the "new" postings only, not to the state file: if a posting shows a discoverable post/open date, drop it from the digest and notification if that date is older than the window — it likely means the site backfilled or resurfaced an old listing rather than posting something genuinely fresh. If a site doesn't expose a post date (e.g. NBER's plain link list, or predoc.org/Columbia entries without a visible "posted" date), keep relying on the new-vs-seen diff itself as the recency signal — don't drop those just for lacking a date field.

3. If there are zero postings left after the recency filter, write a one-line entry to today's digest (see below) noting nothing new, and stop. Do not send a push notification for a zero-match run.

4. Otherwise, use the Google Drive connector to find and read the profile documents listed in `config.json` → `profile_documents` (search by filename in the `drive_folder`; these are not stored in this repo). For each posting that passed the recency filter, judge fit against `config.json` → `audience` and the profile documents:
   - Consider field/discipline match, seniority (predoc/RA/postdoc/junior research scientist level — not tenure-track faculty roles, which don't fit this candidate's career stage), and institution.
   - Weight `research_track` documents primarily since these sites are academic/research boards; consider `quant_marketing_track` for marketing- or data-adjacent research roles.
   - Location: treat NY-metro postings (NYU, Columbia, or any NYC/NY-metro institution surfaced via NBER/predoc.org) as a plus, not a hard filter — still surface strong non-NY matches, but call out location explicitly in the digest so it's easy to scan for local opportunities.
   - Ignore postings clearly outside scope (tenure-track faculty searches, senior staff/administrative roles, unrelated disciplines like clinical medicine, K-12 teaching, etc.) unless there's a clear reason they'd fit.

5. Write a digest file at `digests/YYYY-MM-DD.md` (use the run date) listing every posting that passed the recency filter, grouped by site, with: title, institution/dept, link, post date if known, and a one-line fit assessment. Mark strong matches clearly (e.g. `**STRONG MATCH**`). If any postings were dropped for being outside the recency window, note the count at the bottom (don't list them individually).

6. If there is at least one strong match, send exactly one `PushNotification` summarizing the count and the best 1-2 matches by name (keep under 200 characters, per that tool's own limit). If there are postings in the digest but none are strong matches, do not push — the digest file is enough.

7. Commit the updated `state/` and `digests/` files to git with a short message like `chore: job tracker run YYYY-MM-DD`, and **push to origin main**. This is essential: each scheduled run starts from a fresh checkout of this repo, so if the updated state isn't pushed, tomorrow's run won't know what was already seen today and will re-notify on the same postings.

## Notes

- Never scrape LinkedIn or Handshake as part of this task — those require login and prohibit automated access. This runbook is scoped to the 3 public sites in `config.json` only.
- If a site's structure has changed enough that extraction fails, note that in the digest instead of guessing, and skip diffing for that site this run.
