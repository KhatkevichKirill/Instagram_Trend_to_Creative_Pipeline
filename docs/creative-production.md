# Creative Production Module

The creative-production module starts from rows in `for_generating`. It can follow the discovery workflows in this repository or run independently with any source that writes the required columns.

## Workflows

| Step | Workflow | Purpose |
| --- | --- | --- |
| `cp01` | [Gemini Creative DNA Extractor](../workflows/creative-production/cp01-gemini-dna-extractor.md) | Downloads the source video from Google Drive, asks Gemini to extract reusable creative DNA, scores product/market fit, and records an accept, skip, or error decision. |
| `cp02` | [Product Adapter](../workflows/creative-production/cp02-product-adapter.md) | Combines the source analysis with the configured product context and produces a source-faithful 15-second adapted concept. |
| `cp03` | [Seedance prompt generator](../workflows/creative-production/seedance/cp03-seedance-prompt-generator.md) or [Kling prompt generator](../workflows/creative-production/kling/cp03-kling-prompt-generator.md) | Converts the adaptation into a validated, model-specific prompt. You may run either branch or both. |
| `cp04` | [Seedance generator](../workflows/creative-production/seedance/cp04-seedance-video-generator-kie.md) or [Kling generator](../workflows/creative-production/kling/cp04-kling-video-generator-kie.md) | Creates the video, polls Kie.ai, saves the provider URL, uploads the result to Google Drive, and saves the Drive link. |

## Public Template

The included workflows point to the public [`for_generating` template](https://docs.google.com/spreadsheets/d/15U44PZbhgFEWRrzOYqnybEk2km_1f6bmPaI0wi_-888/edit#gid=1839004881). The sheet ID is `15U44PZbhgFEWRrzOYqnybEk2km_1f6bmPaI0wi_-888`; the `for_generating` gid is `1839004881`.

Make a copy before storing private source URLs, analysis, prompts, or generated-video links. If you use a copy, update every Google Sheets node in all imported workflows.

## Configuration

Every workflow begins with a manual trigger followed by a `Pipeline Config` Code node. The node deliberately appears in every file so each workflow remains independently importable and schedulable.

Edit these fields before running:

- `product.name`, `description`, `audiences`, `value_props`, and `target_markets`;
- `scoring.allowed_languages`;
- `scoring.max_source_duration_sec`;
- `scoring.min_product_fit_score` and `min_ai_video_generation_score`;
- `scoring.reject_local_or_cultural_concepts`;
- `scoring.target_video_duration_sec` and `generation.aspect_ratio`;
- `generation.seedance_model` or `generation.kling_model` if Kie.ai changes its model identifiers.

The defaults describe Speechify and target English-language Tier 1 markets. They are examples, not hidden constants elsewhere in the LLM prompts.

## Credentials

Reconnect these placeholders after import:

| Placeholder | Used for |
| --- | --- |
| `Google Sheets account` | Reading and updating `for_generating`. |
| `Google Drive account` | Downloading source videos and uploading generated results. |
| `Google Gemini API account` | Video analysis and creative-DNA extraction. |
| `OpenAI API account` | Product adaptation and model-specific prompt generation. |
| `Kie.ai API key` | Creating and polling Seedance/Kling jobs. Configure this as an HTTP Header Auth credential according to Kie.ai's current API instructions. |

The repository contains credential names only. Credential IDs and secret values from the original n8n instance are removed.

## Manual Triggers and Orchestration

All six workflows ship with manual triggers. This keeps orchestration policy outside the templates. After testing each workflow independently, you can replace the trigger with:

- a Schedule Trigger;
- a Webhook Trigger;
- Execute Sub-workflow from an orchestration workflow;
- a queue/event trigger supported by your n8n installation.

Keep the status filters when changing triggers. They provide idempotency and prevent already-generated rows from being processed again.

## Recommended First Run

1. Copy the public spreadsheet or confirm that you intentionally want to use it directly.
2. Import `cp01` and reconnect Google Sheets, Google Drive, and Gemini credentials.
3. Edit `Pipeline Config`, add one row with `Shortlisted=Yes` and `status=queued`, then run `cp01`.
4. Confirm that the row becomes `dna_extracted`, `skipped`, or `analysis_error`.
5. Import and test `cp02`; an accepted row should become `adapted`.
6. Follow the [Seedance](seedance.md) or [Kling](kling.md) branch instructions.

See [`for-generating-schema.md`](for-generating-schema.md) for the column and status contracts.
