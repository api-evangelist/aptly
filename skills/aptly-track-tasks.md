---
name: Search and manage Aptly tasks
description: Query tasks for a company, create a task (optionally mirrored on a card), read it back, and keep it in sync.
api: openapi/aptly-openapi-original.yml
operations: [searchTasks, createTask, getTask, updateTask]
---

# Search and manage Aptly tasks

Use this to drive task/checklist work for a property team.

## Auth
- Header `x-token: <API_KEY>` (or a delegate token with the `tasks:*` scope) against `https://core-api.getaptly.com`.

## Steps
1. **Search.** `searchTasks` (`POST /api/tasks/search`) — all body fields are optional filters (assignee, completion/pinned state, priority, board/card, stream/channel, and `dueAt`/`checkedAt`/`updatedAt` ranges as `{ startDate, endDate }`). Set `useCount: true` to get `{ count }` instead of `{ tasks }`.
2. **Create.** `createTask` (`POST /api/tasks`) — when `aptletInstanceId` is set the task is also mirrored as a checklist entry on that card.
3. **Read.** `getTask` (`GET /api/tasks/{taskId}`) returns the task with card/board context and resolved attachments; pass `includeMetadata=true` for `priorityLabel`/`statusLabel` and the resolved `assignee`.
4. **Update.** `updateTask` (`PUT /api/tasks/{taskId}`) updates the task and keeps its card-checklist mirror entry in sync.

## Rules
- Offset pagination and the 120 req/min limit apply; honor `Retry-After` on `429`.
- `{ "error", "message" }` error envelope; `404` = task not found in your company.
