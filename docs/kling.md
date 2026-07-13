# Kling Branch

The Kling branch consists of two workflows:

1. [Kling prompt generator](../workflows/creative-production/kling/cp03-kling-prompt-generator.md)
2. [Kling video generator via Kie.ai](../workflows/creative-production/kling/cp04-kling-video-generator-kie.md)

## Setup

1. Complete `cp01` and `cp02`; the row must have `status=adapted` and a non-empty `Adapter` value.
2. Import Kling `cp03` and reconnect `Google Sheets account` and `OpenAI API account`.
3. Copy the same `Pipeline Config` values used in `cp01` and `cp02`.
4. Run `cp03`. It writes the Kling prompt and validation state to the branch-specific columns.
5. Review rows marked `kling_prompt_needs_review` or `kling_prompt_parse_error` before generation.
6. Import Kling `cp04`; reconnect Google Sheets, Google Drive, and `Kie.ai API key`.
7. In `Upload file`, replace `YOUR_GOOGLE_DRIVE_ID` and `YOUR_OUTPUT_FOLDER_ID`.
8. Confirm `generation.kling_model` matches a currently supported Kie.ai model ID, then run `cp04`.

## Prompt Contract

The prompt generator stores JSON in `final_prompt_kling`. Kling's prompt is limited to 2,500 characters and is intentionally literal: physical action, camera, pacing, setting, short dialogue, and no generated interfaces or readable text.

The video generator only processes rows where:

- `final_prompt_kling` is not empty; and
- `output_gd_kling` is empty.

The generator normalizes duration, aspect ratio, and mode before creating the Kie.ai task. It saves the Kie.ai result URL, downloads the video, uploads it to Google Drive, and writes the final Drive link.
