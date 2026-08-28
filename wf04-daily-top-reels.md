# wf04 Daily_Top_Reels

Daily ranking and delivery workflow for candidate Reels.

## Current behavior

- Reads the `general` sheet and the snapshot Data Table.
- Keeps Reel rows that pass the configured minimum views and like-rate gates.
- Scores candidates from reach, engagement, recency, and the optional momentum bonus.
- Excludes rows already marked as added from new picks.
- Appends selected rows to `for_generating`, updates their source status, and sends Slack messages.

## Known limitation

The scoring code expects joined snapshot fields named `prev_views`, `prev_likes`, and `prev_seen_ts`. The current snapshot table/export writes source metric columns instead, so executions observed during the audit treated every candidate as new. Ranking by reach, engagement, and recency still runs, but the momentum bonus is not reliable until the snapshot read/write contract is aligned.

The Slack output intentionally avoids unstable Instagram thumbnail image blocks; messages use the safer text-based format.

## Import checklist

Reconnect Google Sheets and Slack credentials, select the snapshot Data Table, confirm the destination sheet and channel, and verify the schedule timezone.
