# Wf03 IG_tophashtags_scraping_UPDATE_Metrics

Refreshes view metrics only while a post continues to grow.

## Selection rules

- Reads all rows from `general` and joins them to `IG Metrics Tracking State` by URL.
- Permanently excludes rows with `tracking_status = stopped_stagnant` before any paid Apify request.
- A post is due at most once per day.
- At most 1,000 due posts are sent to Apify per execution; already tracked due posts are prioritized.
- Growth is evaluated only from views: `max(play_count, ig_play_count)`.
- Each evaluation window lasts 72 hours.
- At the end of a window:
  - growth below 10% sets `tracking_status = stopped_stagnant`;
  - growth of 10% or more keeps the post active and starts a new 72-hour window.
- Likes and comments are refreshed but do not affect the stop decision.

## Tracking table

| Column | Type | Meaning |
| --- | --- | --- |
| `url` | string | Upsert key |
| `shortcode` | string | Instagram shortcode |
| `tracking_status` | string | `active` or `stopped_stagnant` |
| `last_checked_at` | date | Last successful refresh |
| `last_views` | number | Latest view count |
| `delta_views` | number | Change since the previous refresh |
| `window_started_at` | date | Start of the current 72-hour window |
| `window_start_views` | number | Views at window start |
| `growth_3d_pct` | number | Window growth percentage |
| `stopped_at` | date | When tracking stopped |
| `stop_reason` | string | Human-readable reason |

Legacy like/comment and no-growth columns may remain for compatibility, but they are not used in the stop decision.

## Verified scenario

An isolated test confirmed that 8% growth after 72 hours is stopped, 20% remains active, and stopped or not-yet-due rows are excluded before Apify.
