# wf01 IG_tophashtags_scraping

Manual seed workflow for creating the initial scope of Instagram posts.

## Purpose

Run this workflow once, or manually when a complete seed refresh is required. Ongoing discovery belongs to `wf02`, which appends only previously unseen shortcodes.

## Current behavior

- Trigger: manual “Execute workflow” only. The workflow is intentionally not published.
- Reads hashtags from `tags_for_scraping`.
- Uses Apify Actor `XxaKfMpcnemfuNxSc` — `khadinakbar/instagram-hashtag-scraper`, the same Actor and input contract as `wf02`.
- Requests posts from the last week, without comments, up to 50 posts per hashtag.
- Normalizes the Actor response to the `general` sheet contract.
- Upserts by `shortcode`.
- Preserves an existing non-empty `status`; assigns `scraped` only to new rows.
- Calls the shared video downloader after the sheet write.

## Why it is separate from wf02

`wf01` creates the first complete scope. `wf02` is the scheduled incremental collector and is the only workflow that should continuously add new posts.

## Import checklist

Reconnect Google Sheets and Apify credentials, select the target spreadsheet, and reconnect the child downloader workflow if its imported ID differs.
