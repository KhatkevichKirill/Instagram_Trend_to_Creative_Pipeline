# IG video sccrapper Download

Shared child workflow used by `wf01` and `wf02`.

## Current behavior

- Accepts scraped rows from another workflow or a manual run.
- Filters rows that require download.
- Extracts the Instagram CDN URL and hostname.
- Resolves an IPv4 address and builds a download URL that keeps the original CDN `Host` header.
- Downloads the binary with browser-like headers and redirects enabled.
- Uploads the file to Google Drive.
- Updates the matching `general` row with `status = downloaded`, the Drive file ID, and a share link.

The IPv4 resolution path avoids failures caused by an unusable IPv6 route while preserving the CDN hostname for HTTP/TLS handling.

## Import checklist

Reconnect Google Sheets and Google Drive credentials, select the destination Drive/folder, and reconnect callers if the imported workflow ID changes.
