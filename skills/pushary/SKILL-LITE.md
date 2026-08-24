---
name: pushary
version: 0.6.2
description: Push notifications and human-in-the-loop for AI agents. Use this whenever a running agent needs a human and nobody is at the terminal, such as before an irreversible or destructive action, before spending money, deploying, force-pushing or deleting, when blocked on a decision outside your authority, when running unattended and you hit a genuine ambiguity, when another skill's workflow says to confirm with the user, and when a long task finishes or fails with nobody watching. Also use it when the user says ping me on my phone, notify me when my agent needs me, approve from my phone, or run this overnight. Ask questions, send alerts, get answers from the user's lock screen.
metadata:
  tags: notifications, push, mcp, human-in-the-loop
---

# Pushary - Push Notifications for AI Agents

Three tools. Use proactively - don't wait for the user to ask.

Reach for them without being asked when: you are about to do something irreversible, spend money, or touch production; the decision is outside your authority; you are unattended and hit a real ambiguity; another skill's workflow says to confirm with the user; or a long task finished with nobody watching.

## ask_user

Ask a question via push and **wait for the answer** (blocks by default).

```json
{ "question": "Delete unused migrations?", "type": "confirm", "agentName": "Codex - myproject" }
```

Returns `{ "answered": true, "value": "yes" }` or `{ "answered": false, "timedOut": true }`.

Types: `confirm` (yes/no), `select` (2-6 options via `options` array), `input` (free text).

## send_notification

One-way alert. Use when a task completes, errors, or needs attention.

Always pass `context.type`: `task_complete`, `error`, or `info`. It is what marks
the notification as a task update, and the user's setting for where task updates
land can only route one that says so.

```json
{ "title": "Build complete", "body": "All tests passing", "agentName": "Codex - myproject",
  "context": { "type": "task_complete", "summary": "17 tests, 0 failures" } }
```

## cancel_question

Cancel a pending question: `{ "correlationId": "..." }`

## propose_scope

Agree the boundary of a multi-step run up front, once: `{ "doneWhen": "tests pass", "sessionId": "...", "allowedPaths": ["src/**"], "offLimitsPaths": ["**/.env*"] }`

Blocks until the user answers. `ratified: true` means it is live, and after that a file outside the scope becomes a "widen scope?" question instead of a silent approval, so the user is asked once per boundary rather than once per file. Globs only; shell commands stay governed by the permission policy. Enforcement needs the Pushary hook (`@pushary/agent-hooks` 0.59.0+); without it the contract is recorded but not gated.

## list_sessions

Read-only: see your live agent sessions and pending questions (no notification sent). Useful to check if a parallel session is blocked.

## Rules

- `agentName` format: `"{Agent} - {project}"` (e.g., `"Cursor - api-server"`)
- `propose_scope` once at the start of a multi-step run, never for a single edit, never mid-run to widen
- Max 3 notifications per task
- MUST `ask_user` type "confirm" before destructive operations (delete, deploy, force push)
- If `answered: false`, follow `handoffAction` when present, otherwise `nextAction`; after one poll, cancel the phone question before asking in the current client
