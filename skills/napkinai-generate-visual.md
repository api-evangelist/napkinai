---
name: Generate a visual from text with Napkin AI
description: >-
  Submit text to Napkin AI, wait for the asynchronous visual generation to
  finish, and download the resulting PNG/SVG/PDF/PPT files. Grounded in the
  Napkin AI developer-preview API.
api: openapi/napkinai-openapi.yml
operations: [createVisualRequest, getVisualRequestStatus, downloadVisualFile]
---

# Generate a visual from text with Napkin AI

Use this skill to turn a block of text into editable visuals (diagrams, charts,
mind maps, icons, infographics) using the Napkin AI API. The API is a developer
preview and the flow is asynchronous: create, poll, download.

## Authentication

- Send an HTTP `Authorization: Bearer <token>` header on **every** request. Use a
  Napkin account API token, or an OAuth 2.0 access token obtained with the
  `generation` scope (OAuth is beta and requires your app to be approved by
  Napkin).
- An expired or missing token returns `401`.

## Steps

1. **Create the request** — `createVisualRequest`: `POST /v1/visual` with a JSON
   body containing the `content` text (required). Optionally set `language`
   (strongly recommended; auto-detected if omitted), `style_id`, `sort_strategy`
   (use `variation` for more layout variety), and `number_of_visuals`. A `201`
   returns a `request_id`. Handle `400` (bad input), `401` (auth), and `429`
   (rate limited — back off and retry).
2. **Poll status** — `getVisualRequestStatus`:
   `GET /v1/visual/{request-id}/status`. Repeat until `status` is `completed` or
   `failed`. On `completed`, read the `generated_files` array (each item has a
   `file_id`, a download `url`, and a `content_type`). On `failed`, read the
   `error` object: `no_credits` (buy/refresh credits) or `no_visuals` (simplify
   the content). Note the `410` case — the request expires 30 minutes after
   generation.
3. **Download each file** — `downloadVisualFile`:
   `GET /v1/visual/{request-id}/file/{file-id}` with the auth header. Use the URL
   already provided in `generated_files` rather than constructing it. Persist the
   bytes and host them yourself for display — do **not** hot-link the Napkin URLs,
   and download within the 30-minute window before they expire (`410`).

## Rules and conventions

- No idempotency key is supported; do not assume safe retries of `createVisualRequest`.
- Respect the 30-minute expiry on both status and file URLs (see
  `conventions/napkinai-conventions.yml`).
- Error and warning codes are catalogued in `errors/napkinai-problem-types.yml`.
