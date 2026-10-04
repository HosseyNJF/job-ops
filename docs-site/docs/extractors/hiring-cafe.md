---
id: hiring-cafe
title: Hiring Cafe Extractor
description: Hiring Cafe search extraction integrated into the pipeline source selector.
sidebar_position: 7
---

## What it is

Original website: [hiring.cafe](https://hiring.cafe)

Special thanks: Initial implementation inspiration came from [umur957/hiring-cafe-job-scraper](https://github.com/umur957/hiring-cafe-job-scraper).

Hiring Cafe reads search and job detail pages and maps results into pipeline jobs.

Implementation split:

1. The extractor builds search state and reads search pages.
2. When a search hit lacks a full description, it requests the job detail page.
3. The pipeline imports the mapped jobs and shows source warnings.

## Why it exists

Hiring Cafe adds another non-credentialed source that can be enabled from the existing source picker, without adding new settings UI.

It also supports term-by-term search and country-aware search state using the same pipeline knobs you already set for automatic runs.

## How to use it

1. Open **Run jobs** and choose **Automatic**.
2. **Hiring Cafe** is enabled by default in **Sources** (toggle it off if you do not want it for this run).
3. Set your existing automatic run knobs:
   - `searchTerms` drive per-term Hiring Cafe `searchQuery`.
   - selected country maps into Hiring Cafe location search state.
   - run budget path (`jobspyResultsWanted`) is reused as the max jobs-per-term cap.
   - optional **Search cities** narrow results by city.
   - workplace type is forwarded from the automatic run modal as a global run filter.
4. Start the run and watch progress in the pipeline progress card.

Defaults and constraints:

- No new Hiring Cafe settings fields were added.
- `worldwide` and `usa/ca` run in broad mode without a strict country location filter.
- Hiring Cafe is enabled by default in source selection.
- Full job descriptions are loaded from Hiring Cafe detail pages when the search result payload only includes summary fields.
- If a job detail page is challenged, the extractor keeps all search results and pauses the pipeline for a human solve. After the solve, it retries the challenged detail URL and reruns Hiring Cafe before importing jobs. A source warning includes the upstream HTTP status when available.
- If a search page is challenged, results from earlier search terms are kept while the pipeline waits for a solve. The challenged page cannot supply new results until it is solved.
- After trying **Solve** on a challenged search page, **Skip this search** appears. It keeps jobs already found, skips that search term and location, and continues with the next Hiring Cafe search. Other Hiring Cafe searches still run. The button is not offered for a challenged job detail page.
- If another search or detail page is challenged during a retry, the pipeline shows the new challenge and pauses again. It imports the collected jobs only after Hiring Cafe finishes without another challenge. You can cancel the run while it is waiting.
- The normalized job payload now preserves structured location evidence from the formatted workplace and city/state/country fields.
- `HIRING_CAFE_DATE_FETCHED_PAST_N_DAYS` controls recency window when running extractor directly (default `7`).
- In the default Map radius mode, Hiring Cafe receives the selected coordinates and radius directly (default `50` miles).
- In Manual cities mode, each city uses Hiring Cafe's city-radius search with the same `50`-mile default and strict city post-filtering.
- Workplace type is global to the run and is not configured separately per city in this integration.
- City geocoding is resolved through Nominatim (OpenStreetMap data); if you scale extractor traffic, add attribution and cache repeated city lookups.

Local run example:

```bash
HIRING_CAFE_SEARCH_TERMS='["backend engineer"]' \
HIRING_CAFE_COUNTRY='united kingdom' \
HIRING_CAFE_MAX_JOBS_PER_TERM='50' \
npm --workspace hiringcafe-extractor run start
```

## Common problems

### Hiring Cafe returns 429 / Vercel security checkpoint

- If a detail request is challenged, solve the challenge in the offered browser. The pipeline keeps the listing results while paused, retries the detail URL afterward, and imports the richer description when the retry succeeds. Check the source warning for the HTTP status; a `429` usually means rate limiting. If a detail request remains challenged, the pipeline pauses again instead of importing partial descriptions.
- If a search page is challenged, use the offered browser solver. The solve only succeeds when a reusable clearance cookie is saved. If no clearance cookie is issued, retry later. If a different search page is challenged after a solve, solve the new browser check; the same run stays active.
- If you cannot solve a challenged search page, try **Solve** once and then select **Skip this search**. Jobs collected before the challenge remain in the run; Hiring Cafe continues with the next term or location.

### Hiring Cafe does not appear in sources

- Check that client is running on latest build containing the new source list.
- Hiring Cafe is source-only and does not require credentials, so it should appear once the new build is loaded.

### Results are lower than expected

- Cap is tied to automatic run budget path (`jobspyResultsWanted`) and search term count.
- Country mapping can narrow results when a strict country location is applied.

## Related pages

- [Extractors Overview](/docs/next/extractors/overview)
- [Pipeline Run](/docs/next/features/pipeline-run)
- [Settings](/docs/next/features/settings)
