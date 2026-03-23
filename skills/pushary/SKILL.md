---
name: pushary
description: Push notifications and human-in-the-loop for AI coding agents. Send alerts when tasks finish, ask yes/no questions via push, get answers from the user's lock screen. Works with Cursor, Claude Code, Windsurf, and any MCP client.
metadata:
  tags: notifications, push, mcp, human-in-the-loop, cursor, claude, windsurf, alerts
---

# Pushary — Push Notifications for AI Agents

Pushary is an MCP server that gives you two capabilities:

1. **Send push notifications** to the user's phone or desktop when a task finishes or an error occurs.
2. **Ask yes/no questions** via push and wait for the user's answer — human-in-the-loop from their lock screen.

Use these tools proactively. Do not wait for the user to ask for notifications.

## When to Use

**Send a notification when:**
- You finish a task that took 3 or more steps
- A build, test suite, or deployment fails
- An error or blocker requires the user's attention
- A long-running process completes (migration, refactor, generation)

**Ask a yes/no question when:**
- You hit a decision point that could go either way
- You need confirmation before a destructive or irreversible action
- The user needs to choose between two approaches

**Do NOT notify when:**
- The task is trivial or single-step
- The question can be answered from context without user input
- You already sent 3 notifications for the current task (unless the user explicitly asked for more)

## Setup

Add Pushary to your MCP configuration:

```json
{
  "mcpServers": {
    "pushary": {
      "url": "https://pushary.com/api/mcp/mcp"
    }
  }
}
```

Sign up at https://pushary.com/sign-up?from=ai-coding to get your API key.

## Tools

### send_notification

Send a one-way push notification to the user.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| title | string | Yes | Notification title (max 100 chars, aim for under 60) |
| body | string | Yes | Notification body (max 500 chars, aim for under 200) |
| url | string | No | URL opened when tapped. Omit to open the dashboard. |
| iconUrl | string | No | Custom notification icon URL |
| imageUrl | string | No | Large image shown in the notification |
| subscriberIds | string[] | No | Target specific subscriber IDs |
| externalIds | string[] | No | Target by external IDs |
| tags | string[] | No | Target by subscriber tags |

**Example — task completed:**

```json
{
  "title": "Refactoring complete",
  "body": "Extracted 3 shared components across 12 files. 0 lint errors."
}
```

**Example — error alert:**

```json
{
  "title": "Build failed",
  "body": "TypeScript error in auth.ts:42 — needs your input"
}
```

### ask_user_yes_no

Send a yes/no question to the user via push notification. Returns a `correlationId` that you pass to `wait_for_answer` to get the response.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| question | string | Yes | The question to ask (max 300 chars) |
| subscriberIds | string[] | No | Target specific subscriber IDs |
| externalIds | string[] | No | Target by external IDs |
| tags | string[] | No | Target by subscriber tags |

**Example:**

```json
{
  "question": "Delete the legacy API routes or keep backward compat?"
}
```

**Returns:** `{ "correlationId": "uuid" }`

### wait_for_answer

Long-poll for the user's response to a question sent via `ask_user_yes_no`. Blocks until the user responds or the timeout is reached.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| correlationId | string (uuid) | Yes | The correlationId from ask_user_yes_no |
| timeoutMs | integer | No | How long to wait (default 30000, max 55000) |

**Returns:**
- `{ "answered": true, "value": "yes" }` — user tapped yes
- `{ "answered": true, "value": "no" }` — user tapped no
- `{ "answered": false }` — timeout reached, no answer yet

### cancel_question

Cancel a pending question so it can no longer be answered. Use when the question becomes irrelevant (e.g., you found the answer another way or the user responded in chat).

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| correlationId | string (uuid) | Yes | The correlationId of the question to cancel |

## Human-in-the-Loop Flow

Follow this exact sequence when you need a decision from the user:

1. Call `ask_user_yes_no` with a clear, concise question.
2. Immediately call `wait_for_answer` with the returned `correlationId` and `timeoutMs: 55000`.
3. If `wait_for_answer` returns `{ "answered": false }` or errors, retry the same `wait_for_answer` call up to 3 times. The answer persists in Redis for 10 minutes, so it will be there when the user responds.
4. Once you receive `{ "answered": true, "value": "yes" }` or `"no"`, act on the decision.
5. If the user answers in chat before the push response arrives, continue normally and skip further polling. Call `cancel_question` to clean up.

**Pseudocode:**

```
result = ask_user_yes_no({ question: "Delete old migrations?" })
correlationId = result.correlationId

for attempt in 1..3:
    answer = wait_for_answer({ correlationId, timeoutMs: 55000 })
    if answer.answered:
        if answer.value == "yes":
            // proceed with deletion
        else:
            // skip deletion
        break

if not answer.answered after 3 attempts:
    // user did not respond — pick the safe default or ask in chat
```

## Notification Etiquette

- **Titles under 60 characters.** They get truncated on phone lock screens.
- **Bodies under 200 characters.** Concise summaries, not full explanations.
- **Max 3 notifications per task** unless the user explicitly requests more.
- **Prefer ask_user_yes_no** for binary decisions. Use send_notification for one-way alerts.
- **Never pass a url param** to send_notification unless you have a specific page to link to. The default opens the user's Pushary dashboard.
- **Write questions as if talking to a busy person.** The user is on their phone, possibly away from their computer. Be specific: "Delete the 3 unused migration files?" is better than "Should I clean up?"
