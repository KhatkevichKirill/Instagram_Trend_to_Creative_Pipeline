# Seedance Branch

The Seedance branch consists of two workflows:

1. [`cp03-seedance-prompt-generator.json`](../workflows/creative-production/seedance/cp03-seedance-prompt-generator.json)
2. [`cp04-seedance-video-generator-kie.json`](../workflows/creative-production/seedance/cp04-seedance-video-generator-kie.json)

## Setup

1. Complete `cp01` and `cp02`; the row must have `status=adapted` and a non-empty `Adapter` value.
2. Import Seedance `cp03` and reconnect `Google Sheets account` and `OpenAI API account`.
3. Copy the same `Pipeline Config` values used in `cp01` and `cp02`.
4. Run `cp03`. It skips rows that already have `final_prompt_seedance`.
5. Review rows marked `seedance_prompt_needs_review` or `seedance_prompt_parse_error` before generation.
6. Import Seedance `cp04`; reconnect Google Sheets, Google Drive, and `Kie.ai API key`.
7. In `Upload file`, replace `YOUR_GOOGLE_DRIVE_ID` and `YOUR_OUTPUT_FOLDER_ID`.
8. Confirm `generation.seedance_model` matches a currently supported Kie.ai model ID, then run `cp04`.

## Prompt Contract

The prompt generator targets a realistic vertical video and stores JSON in `final_prompt_seedance`. The `seedance.prompt` field is limited to 5,000 characters. Validation also rejects spoken use of the configured product brand and requests for readable UI/text that should be added in post-production.

The video generator only processes rows where:

- `final_prompt_seedance` is not empty; and
- `output_gd_seedance` is empty.

This makes reruns safe after partial failures. Kie.ai's provider URL is written first, followed by the uploaded Google Drive link.
