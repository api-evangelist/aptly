---
name: Look up and upsert Aptly contacts
description: Find a person by email, create or update their contact record, and read it back — the Aptly contact/CRM flow.
api: openapi/aptly-openapi-original.yml
operations: [getContactsByEmail, listContacts, upsertContact, getContact, updateContact]
---

# Look up and upsert Aptly contacts

Use this to resolve and maintain person records (residents, owners, prospects) scoped to your company.

## Auth
- Header `x-token: <API_KEY>` (or a delegate token with `contacts:read` / `contacts:write`) against `https://core-api.getaptly.com`.

## Steps
1. **Resolve by email first.** `getContactsByEmail` (`POST /api/contacts/by-email`) returns contacts whose email matches (case-insensitive, exact). Prefer this over creating duplicates.
2. **Or browse.** `listContacts` (`GET /api/contacts`) returns a paginated, company-scoped list; all filter params are optional and ANDed.
3. **Create or update.** `upsertContact` (`POST /api/contacts`) creates a new contact or updates an existing one (upsert). To update a known record by id use `updateContact` (`POST /api/contacts/{contactId}`) — the `_id` comes from the URL and any `_id` in the body is ignored.
4. **Read one back.** `getContact` (`GET /api/contacts/{contactId}`) returns a single contact with custom fields enriched by their type definitions.

## Rules
- Offset pagination (`page`/`limit`), `total` in the response.
- 120 req/min per key; honor `Retry-After` on `429`.
- `{ "error", "message" }` error envelope; `404` = no contact with that id/email in your company.
- See `conventions/aptly-conventions.yml` for the shared request/response semantics.
