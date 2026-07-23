---
name: Draft and send Aptly email
description: List a user's inboxes, create an outbound email draft, and send it — optionally linked to a card or replied into an existing thread.
api: openapi/aptly-openapi-original.yml
operations: [listUserInboxes, createEmailDraft, sendEmail]
---

# Draft and send Aptly email

Use this to send resident/owner communication from a user's connected inbox.

## Auth
- Header `x-token: <API_KEY>` (or a delegate token with the `email:*` scope) against `https://core-api.getaptly.com`.

## Steps
1. **Find the inbox.** `listUserInboxes` (`GET /api/users/{userId}/inboxes`) returns the email inboxes (Hermes/Nylas channels) the user can access — `kind: "personal"` or `"shared"`. Take the `channelId`.
2. **Create a draft.** `createEmailDraft` (`POST /api/email/create-draft`) from `to`/`cc`/`bcc`/`subject`/`body`/`channelId`. Returns `streamId` and `draftUuid`. Optionally set `aptletInstanceId` to link the outbound to a card, or `discussionId` to reply into an existing thread.
3. **Send.** `sendEmail` (`POST /api/email/send`) — either from the draft (`discussionId` + `uuid`) or on-the-fly from bare email fields. Supplying `discussionId` without `uuid` appends a reply to that thread.

## Attachments (optional)
- Upload each file first: `getFileUploadUrl` (`POST /api/files/upload-url`) → POST the bytes to the presigned S3 `url` (fields first, then `file`) → `completeFileUpload` (`POST /api/files/upload-complete`). Pass the returned `fileId`s as `attachmentIds`. Inline images embedded via `<img src=...download url...>` are auto-detected — don't also list them in `attachmentIds`.

## Rules
- 120 req/min per key; honor `Retry-After` on `429`.
- `{ "error", "message" }` error envelope; `401` = inbox not accessible / bad credential.
