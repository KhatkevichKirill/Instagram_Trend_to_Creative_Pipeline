# Instagram Trend-to-Creative Pipeline

This repository contains an end-to-end n8n + Google Sheets pipeline for discovering Instagram Reels that are starting to trend, tracking their growth, selecting the strongest candidates, adapting their creative structure to a product, and generating new videos with Seedance or Kling through Kie.ai.

The repository is intentionally split into two modules. Trend discovery can run on its own, and creative production can accept rows from any compatible source through the `for_generating` sheet contract.

## What It Does

The discovery module monitors selected Instagram hashtags, collects top and recent Reels, deduplicates already-known posts, refreshes performance metrics over time, and surfaces the best-performing videos every day.

The creative-production module analyzes shortlisted videos with Gemini, applies configurable market and product-fit gates, adapts the reusable concept to a product, generates model-specific prompts, and creates videos through either Seedance or Kling.

Both modules communicate through `for_generating`. You can use the full pipeline or populate that sheet yourself and start directly from creative production.

## Spreadsheet Structure

The workflows expect a Google Spreadsheet with these sheets:

| Sheet | Purpose |
| --- | --- |
| `tags_for_scraping` | The list of hashtags to monitor. Each row is a hashtag input for the scraping workflows. |
| `general` | The main database of scraped Instagram posts and Reels. Stores URLs, captions, authors, metrics, media links, Google Drive links, and workflow status. |
| `shortcode` | A lightweight lookup sheet used by the dedup workflow to avoid appending Reels that are already present. |
| `for_generating` | The output queue. Daily top Reels are appended here so the creative team can use them for generation or cloning workflows. |

## n8n Data Tables

The pipeline also uses native n8n Data Tables for workflow state.

### `IG Metrics Tracking State`

Used by `wf03` to decide whether each post should be checked again through Apify.

Recommended columns:

| Column | Type | Purpose |
| --- | --- | --- |
| `url` | string | Post/Reel URL used as the upsert key. |
| `shortcode` | string | Instagram shortcode. |
| `tracking_status` | string | One of `active`, `cooldown`, or `paused_no_growth`. |
| `last_checked_at` | dateTime | Last time metrics were refreshed through Apify. |
| `last_growth_at` | dateTime | Last time the Reel showed metric growth. |
| `last_views` | number | Last known view count. |
| `last_likes` | number | Last known like count. |
| `last_comments` | number | Last known comment count. |
| `delta_views` | number | View growth since the previous check. |
| `delta_likes` | number | Like growth since the previous check. |
| `delta_comments` | number | Comment growth since the previous check. |
| `no_growth_checks` | number | Consecutive checks with no growth. |
| `next_check_at` | dateTime | Next time this URL should be sent to Apify. |

### Daily Top Reels Snapshot Table

Used by `wf04` to remember previous metrics for scoring and momentum detection. The imported workflow includes a sticky note with the required setup details. At minimum, it stores the Reel `shortcode`, previous views, previous likes, and the previous seen timestamp.

## Apify Actors

The workflows use these Apify actors:

| Actor | Used by | Purpose |
| --- | --- | --- |
| Instagram Hashtag Posts Scraper (`breathtaking_anthem/instagram-hashtag-posts-scraper`) | `wf01`, `wf02` | Scrapes top or recent posts for each tracked hashtag. |
| Instagram Post Scraper (`apify/instagram-post-scraper`) | `wf03` | Refreshes metrics for already-collected post/Reel URLs in batches of up to 50 URLs. |

## Workflow Overview

```mermaid
flowchart TD
  A["tags_for_scraping"] --> B["wf01 initial top hashtag scrape"]
  A --> C["wf02 scheduled recent hashtag scrape"]
  B --> D["general sheet"]
  C --> D
  D --> E["wf03 metrics refresh + tracking state"]
  E --> D
  D --> F["wf04 daily scoring"]
  F --> G["for_generating sheet"]
  B --> H["IG video downloader"]
  C --> H
  H --> D
  G --> I["cp01 Gemini analysis + product-fit gate"]
  I -->|accepted| J["cp02 product adapter"]
  I -->|rejected| K["skipped"]
  J --> L["cp03 Seedance prompt"]
  J --> M["cp03 Kling prompt"]
  L --> N["cp04 Seedance via Kie.ai"]
  M --> O["cp04 Kling via Kie.ai"]
```

## Workflows

Detailed workflow documentation:

- [wf01: IG Top Hashtags Scraping](wf01-ig-tophashtags-scraping.md)
- [wf02: IG Hashtags Scraping - Dedup Append Only](wf02-ig-hashtags-scraping-dedup-append-only.md)
- [wf03: IG Top Hashtags Scraping - Update Metrics](wf03-ig-tophashtags-scraping-update-metrics.md)
- [wf04: Daily Top Reels](wf04-daily-top-reels.md)
- [IG Video Scrapper Download](ig-video-scrapper-download.md)
- [Creative production overview](docs/creative-production.md)
- [`for_generating` sheet contract](docs/for-generating-schema.md)
- [Seedance setup](docs/seedance.md)
- [Kling setup](docs/kling.md)

Each creative-production JSON file also has a detailed Markdown guide beside it, matching the documentation pattern used by the discovery workflows.

### `wf01 IG_tophashtags_scraping.json`

Initial backfill workflow for building the first dataset from hashtag top posts.

It reads hashtags from `tags_for_scraping`, sends each hashtag to the Instagram Hashtag Posts Scraper with `scrape_type: top`, and writes the returned posts into the `general` sheet using `shortcode` as the matching key. This workflow is useful for the first run, when you want to collect a large initial pool of high-performing posts for the tracked hashtags.

After each batch completes, it can call the helper video download workflow to save video files to Google Drive and write the resulting Drive file ID/link back into `general`.

Typical usage: run manually once during setup, or rerun occasionally when adding a new group of hashtags.

### `wf02 IG_hashtags_scraping - dedup append only.json`

Scheduled discovery workflow for new hashtag posts.

It reads existing shortcodes, then scrapes `recent` posts for every hashtag in `tags_for_scraping`. Before appending anything to `general`, it filters out posts whose shortcode is already known. This keeps the main sheet append-only for new discoveries and prevents repeated rows for the same Reel.

It also calls the helper video download workflow so newly discovered videos can be saved before temporary Instagram media URLs expire.

Typical usage: run on a schedule to keep discovering fresh posts for your monitored hashtags.

### `wf03 IG_tophashtags_scraping_UPDATE_Metrics.json`

Metric refresh and growth-tracking workflow for already-collected posts.

It reads URLs from the `general` sheet, enriches them with the `IG Metrics Tracking State` Data Table, filters out posts whose `next_check_at` is still in the future, batches due URLs in groups of up to 50, and sends each batch to the Apify Instagram Post Scraper.

After Apify returns fresh metrics, the workflow updates the latest counts in `general` and upserts tracking state into the Data Table:

- `active`: the post is new or has shown growth.
- `cooldown`: the post had no growth, but has not yet been paused.
- `paused_no_growth`: the post had no growth for several consecutive checks and should be checked less often.

This is the workflow that catches delayed breakouts. For example, a Reel can enter the system at 1,000 views and only become interesting several days later after growing to 500,000 views. The workflow keeps watching posts while they are still moving and reduces Apify usage for posts that have gone flat.

### `wf04 Daily_Top_Reels.json`

Daily ranking workflow for selecting the best creative candidates.

It reads the current `general` sheet, attaches previous snapshot metrics from an n8n Data Table, scores Reels, and appends the top picks to `for_generating`. It then marks selected rows in `general` as already moved, so the same Reel is not repeatedly queued.

The scoring logic considers:

- minimum view threshold;
- like rate quality gate;
- engagement rate;
- recency decay;
- percentile rank for views, like rate, and engagement;
- momentum versus the previous snapshot.

By default, the workflow marks the top 5 eligible Reels each day, but the limit and scoring weights can be changed in the `Score & rank reels` Code node.

### `IG video sccrapper Download.json`

Helper workflow for saving Instagram videos to Google Drive.

Instagram video URLs returned by scrapers can expire quickly. This workflow downloads the video while the URL is still valid, uploads it to Google Drive, and writes `gd_file_id`, `link_to_gdrive`, and `status: downloaded` back to the matching row in `general`.

It is called by the scraping workflows rather than being the main discovery flow.

## Recommended Operating Flow

1. Add hashtags to `tags_for_scraping`.
2. Run `wf01` once to collect an initial pool of top hashtag posts.
3. Enable `wf02` on a schedule to append newly discovered recent posts.
4. Enable `wf03` to refresh metrics and track growth while avoiding unnecessary Apify runs.
5. Enable `wf04` to move the strongest daily candidates into `for_generating`.
6. Run `cp01` to analyze queued videos and reject candidates that fail the configured gates.
7. Run `cp02` to adapt accepted creative DNA to the configured product.
8. Choose Seedance, Kling, or both: run the matching `cp03` prompt generator and `cp04` video generator.

All six creative-production workflows use manual triggers on purpose. Replace the trigger after import if you prefer a schedule, webhook, Execute Sub-workflow node, or another orchestration strategy.

## Setup Notes

- Import all JSON workflows into n8n.
- Reconnect Google Sheets, Google Drive, and Apify credentials after import.
- The included discovery and creative-production workflows point to the public template spreadsheet. Replace its ID in every Google Sheets node if you make a private copy.
- Replace `YOUR_GOOGLE_DRIVE_ID` and `YOUR_OUTPUT_FOLDER_ID` in Google Drive upload nodes.
- Update n8n Data Table IDs for your own workspace.
- Create the required n8n Data Tables before running `wf03` and `wf04`.
- Imported nodes should be checked once in the n8n UI because Data Table references and credential selections are instance-specific.
- Creative-production workflows point to the public [template spreadsheet](https://docs.google.com/spreadsheets/d/15U44PZbhgFEWRrzOYqnybEk2km_1f6bmPaI0wi_-888/edit#gid=1839004881). Make your own copy before using it with private data.
- Edit the `Pipeline Config` Code node in every creative-production workflow to change the product context, target markets, scoring thresholds, target duration, or generation model.
- Reconnect the generic Google Sheets, Google Drive, Gemini, OpenAI, and Kie.ai credential placeholders after importing the creative-production workflows.
- In both video-generator workflows, replace `YOUR_GOOGLE_DRIVE_ID` and `YOUR_OUTPUT_FOLDER_ID` in the `Upload file` node.

## Repository Description

Short GitHub description:

> End-to-end n8n pipeline for discovering trending Instagram Reels, scoring and adapting creative concepts, and generating videos with Gemini, Seedance, Kling, and Kie.ai.
