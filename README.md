# Instagram Trend-to-Creative Pipeline

An n8n + Google Sheets pipeline for discovering growing Instagram Reels, controlling paid metric refreshes, selecting daily candidates, and handing them to creative production.

## Discovery workflows

| Workflow | Role |
| --- | --- |
| `wf01 IG_tophashtags_scraping` | Manual, unpublished seed collector for the initial one-week scope |
| `wf02 IG_hashtags_scraping - dedup append only` | Scheduled incremental collector; appends only unseen shortcodes |
| `Wf03 IG_tophashtags_scraping_UPDATE_Metrics` | Refreshes views while posts keep growing and permanently stops stagnant rows |
| `wf04 Daily_Top_Reels` | Ranks eligible Reels and sends daily picks to `for_generating` and Slack |
| `IG video sccrapper Download` | Shared Instagram CDN to Google Drive downloader |

`wf01` and `wf02` use Apify Actor `XxaKfMpcnemfuNxSc` (`khadinakbar/instagram-hashtag-scraper`) with a one-week window, no comments, and a maximum of 50 posts per hashtag.

## Spreadsheet contract

| Sheet | Purpose |
| --- | --- |
| `tags_for_scraping` | Hashtags monitored by wf01/wf02 |
| `general` | Source-of-truth rows, metrics, Drive links, and workflow status |
| `shortcode` | Lightweight dedup lookup used by wf02 |
| `for_generating` | Queue of selected Reels for creative production |

## View-growth cost control in wf03

`wf03` checks a post at most daily and evaluates a 72-hour view-growth window. Growth uses `max(play_count, ig_play_count)`. If growth is below 10%, the row becomes `stopped_stagnant` and is excluded before future Apify requests. Growth of at least 10% starts a new window. A run sends at most 1,000 due posts to Apify, prioritizing already tracked rows.

The `IG Metrics Tracking State` table needs these core fields:

| Field | Type |
| --- | --- |
| `url`, `shortcode`, `tracking_status`, `stop_reason` | string |
| `last_views`, `delta_views`, `window_start_views`, `growth_3d_pct` | number |
| `last_checked_at`, `window_started_at`, `stopped_at` | date |

## Known wf04 limitation

The current `wf04` scoring code expects snapshot aliases `prev_views`, `prev_likes`, and `prev_seen_ts`, while the snapshot write contract currently stores source metric field names. Observed runs therefore treated candidates as new. Reach/engagement/recency ranking works, but momentum should be considered disabled until that contract is aligned.

## Import and security

Exports contain placeholder/template resource selections. After import, reconnect credentials and choose your own Google Sheets, Google Drive, n8n Data Tables, Slack channel, and child-workflow references. Do not commit credential IDs, tokens, or private sheet URLs.

Detailed workflow notes are in the matching Markdown files in the repository root. Creative-production documentation remains under `docs/creative-production/`.
