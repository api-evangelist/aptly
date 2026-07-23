---
name: Create and update an Aptly card
description: Discover a board's schema, create a card on it, comment on it, and attach a file — the core Aptly write flow for external systems and AI agents.
api: openapi/aptly-openapi-original.yml
operations: [listBoards, getSchema, createCard, postComment, uploadFile]
---

# Create and update an Aptly card

Use this to push work into an Aptly board (a ticket, deal, lead, screening, etc.) from an external system.

## Auth
- Send every request with the header `x-token: <API_KEY>` against `https://core-api.getaptly.com`.
- The API must be enabled on the target board (Board → Card Sources → API). A `403` means it is not enabled.

## Steps
1. **Find the board.** `listBoards` (`GET /api/boards`) returns every API-enabled board with its UUID and prebuilt endpoint URLs. Pick the target `boardUuid`.
2. **Read the schema first.** `getSchema` (`GET /api/schema/{boardId}`) returns the field definitions. You must use field **UUIDs** as keys when writing card values — never guess field keys. Check the field types (see field-types docs) so you send values in the right shape.
3. **Create the card.** `createCard` (`POST /api/board/{boardId}`) — put field UUIDs as keys in the body. This is create-or-update by design, so the same call updates existing values.
4. **Comment (optional).** `postComment` (`POST /api/board/{boardId}/{cardId}/comment`) adds a comment. Include an `id` to edit an existing comment (the `userId` must match the original author).
5. **Attach a file (optional).** `uploadFile` (`POST /api/board/{boardId}/{cardId}/file`) with `multipart/form-data`, file in the `file` field.

## Rules
- Pagination on any list is offset-based (`page` from 0, `limit` ≤ 100); stop when `(page+1)*limit >= total`.
- Respect rate limits: 120 req/min per key; on `429` honor the `Retry-After` header.
- Errors are `{ "error", "message" }` JSON (not RFC 9457). `401` = bad key, `404` = wrong board/card for your company. See `errors/aptly-problem-types.yml`.
- No idempotency-key contract exists; rely on the upsert shape of `createCard` rather than blind retries of non-upsert writes.
