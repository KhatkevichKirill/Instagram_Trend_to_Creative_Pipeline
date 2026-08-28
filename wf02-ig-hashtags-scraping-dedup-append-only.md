# wf02 IG_hashtags_scraping — dedup append only

Scheduled incremental discovery workflow.

## Current behavior

- Reads hashtags from `tags_for_scraping`.
- Uses Apify Actor `XxaKfMpcnemfuNxSc` — `khadinakbar/instagram-hashtag-scraper`.
- Requests posts from the last week, without comments, up to 50 posts per hashtag.
- Preloads known shortcodes before scraping.
- Normalizes Actor output to the `general` sheet contract.
- Drops invalid records and every shortcode already present in the sheet.
- Appends only new rows with `status = scraped`.
- Calls the shared downloader for newly appended posts.

This workflow is append-only by design. It does not update existing post metrics; `wf03` owns metric refreshes.

## Import checklist

Reconnect Google Sheets and Apify credentials, select the spreadsheet, confirm the schedule/timezone, and reconnect the child downloader workflow if needed.
