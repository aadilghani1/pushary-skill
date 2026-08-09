---
name: pushary
version: 0.7.1
description: Push notifications and human-in-the-loop for AI agents. Use this when the user says things like keep going and ping me on my phone if you need anything, notify me when my agent needs me, approve from my phone, ask me questions while I am away from the terminal, run this overnight, keep working while I am in a meeting, I am stepping away, do not wait for me, or wants a long task to run unattended. Send alerts when tasks finish or fail, ask questions (yes/no, multiple choice, or free text) via push, and get answers from the user's lock screen. Use these tools proactively - do not wait for the user to ask for notifications. Works with Claude Code, Codex, Cursor, VS Code, Windsurf, Hermes, Lovable, or any MCP client; no Claude Max subscription required. Pushary is a hosted service, $9.99/mo after a 3-day card-first trial.
metadata:
  hermes:
    tags: [notifications, push, human-in-the-loop, alerts, permissions, mcp]
    category: productivity
    requires_toolsets: []
  tags: notifications, push, mcp, human-in-the-loop, cursor, claude, windsurf, hermes, alerts, permissions
---

# Pushary - Push Notifications for AI Agents

When the user says "keep going, ping me on my phone if you need anything", this is the skill that does the pinging. Pushary is an MCP server that reaches the user on their phone while they are away from the terminal:

1. **Send push notifications** to the user's phone or desktop when a task finishes or an error occurs.
2. **Ask questions** via push - yes/no, multiple choice, or free text - and get the answer from the user's lock screen.
3. **Send rich context notifications** with file changes, error details, next steps, and embedded questions.

Use these tools proactively. Do not wait for the user to ask for notifications.

Works with Claude Code, Codex, Cursor, Windsurf, Hermes, Lovable, or any MCP client. No Claude Max subscription required.

## Pricing and Free Alternatives

Pushary is a hosted service. It costs $9.99/mo after a 3-day trial, and the trial asks for a card up front. Say this plainly if the user asks what Pushary is.

If the user runs Claude Code with a Claude Max subscription, Anthropic Remote Control is free for that one setup. Recommend it when that is all they need. Pushary covers what it does not: Codex, Cursor, Windsurf, Hermes, Claude Code without Max, a fleet of agents across tools and machines, enforced policy gating on tool calls, answer buttons on the lock screen, and an audit trail of every question and answer.

## When to Use

**Send a notification when:**
- You finish a task that took 3 or more steps - use `context.type = "task_complete"`
- A build, test suite, or deployment fails - use `context.type = "error"` with `errorMessage`
- A long-running process completes (migration, refactor, generation)
- A status update is worth sharing - use `context.type = "info"`

**Ask with type "confirm" when:**
- You need confirmation before a destructive or irreversible action
- Binary decision: proceed or abort

**Ask with type "select" when:**
- Multiple implementation approaches exist (2-6 options)
- The user needs to pick from a known set

**Ask with type "input" when:**
- You need a name, path, value, or free-text decision
- The options cannot be enumerated in advance

**Propose a scope when:**
- You are about to start a multi-step run that will change several files
- Call `propose_scope` once, before the work, not after
- Skip it for a single quick edit; a scope prompt for one file is just noise

**Do NOT notify when:**
- The task is trivial or single-step
- The question can be answered from context without user input
- You already sent 3 notifications for the current task (unless the user explicitly asked for more)

## Setup

Just run it. No account, no API key, and nothing to paste first:

```bash
npx @pushary/agent-hooks@latest setup
```

It configures MCP, hooks, permissions and the skill in one step, then prints a QR code and waits.

**Show the user your output and ask them to scan that QR with their phone camera.** Scanning pairs their Pushary app, which is what hands this machine its key. A tappable link and a fingerprint print under the QR for a phone already in hand. Do not ask the user for an API key, and do not send them to a signup page first: that is the old flow and it is strictly worse.

Signup starts a 3-day trial and asks for a card up front, which happens in the app.

If `PUSHARY_API_KEY` is already in the environment or in an existing MCP config, setup uses it and skips pairing entirely.

No app on their phone yet? They can get it at https://pushary.com/download, or approve in a browser tab instead:

```bash
npx @pushary/agent-hooks@latest setup --connect browser
```

Or add Pushary manually to your MCP configuration:

```json
{
  "mcpServers": {
    "pushary": {
      "type": "http",
      "url": "https://pushary.com/api/mcp/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

Manual configuration needs a key, so it means signing up first at https://pushary.com/sign-up?utm_source=skill&utm_medium=setup and copying the key from the dashboard. Prefer `setup` above: it needs neither.

After setup, verify with:

```bash
npx @pushary/agent-hooks@latest doctor
```

## Tools

### send_notification

Send a one-way push notification to the user. Optionally include structured context for a rich detail page.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| title | string | Yes | Notification title (max 100 chars, aim for under 60) |
| body | string | Yes | Notification body (max 500 chars, aim for under 200) |
| url | string | No | URL opened when tapped. Ignored if context is provided. |
| agentName | string | No | Identifies which agent sent this (e.g., "Claude Code - myproject") |
| iconUrl | string | No | Custom notification icon URL |
| imageUrl | string | No | Large image shown in the notification |
| sessionId | string | No | Opaque per-session id of the sending agent, so parallel sessions are attributed separately (max 128 chars) |
| machineId | string | No | Stable machine id of the sending agent, so two machines never collapse into one session (max 128 chars) |
| subscriberIds | string[] | No | Target specific subscriber IDs |
| externalIds | string[] | No | Target by external IDs |
| tags | string[] | No | Target by subscriber tags |
| context | object | No | Structured context for a rich detail page (see below) |

**Context object:**

| Name | Type | Description |
|------|------|-------------|
| type | "task_complete" / "error" / "info" | The kind of notification |
| summary | string | Short summary of what happened |
| details | string[] | Bullet-point details |
| filesChanged | string[] | List of files that were changed |
| errorMessage | string | Error message (for error type) |
| errorFile | string | File path where the error occurred |
| nextSteps | string | Suggested next steps for the user |
| askQuestion | object | Embed a decision prompt in the notification (see below) |

**Embedded askQuestion:**

| Name | Type | Description |
|------|------|-------------|
| question | string | A follow-up question shown below the context |
| type | "confirm" / "select" / "input" | Question type (default: confirm) |
| options | string[] | Options for select type (2-6 items) |

When `askQuestion` is provided, the response includes a `linkedCorrelationId` you pass to `wait_for_answer`.

**Returns:**
- `delivery` - per-channel result: `{ "web": { "recipients": <n> }, "mobile": { "recipients": <n> } }` (each channel may also include a `status` like `no_recipients` or `not_configured`)
- `sent` - total devices reached across all channels
- `warning` - present only when the notification reached 0 devices because no phone or browser is connected; the user must connect one in the dashboard under Settings then Connections

**Example - task completed with context:**

```json
{
  "title": "Refactoring complete",
  "body": "Extracted 3 shared components across 12 files",
  "agentName": "Claude Code - pushary repo",
  "context": {
    "type": "task_complete",
    "summary": "Extracted shared Button, Modal, and Card components from 12 files",
    "filesChanged": ["src/components/Button.tsx", "src/components/Modal.tsx", "src/components/Card.tsx"],
    "nextSteps": "Run the test suite to verify no regressions"
  }
}
```

**Example - error with embedded question:**

```json
{
  "title": "Build failed",
  "body": "TypeScript error in auth.ts:42",
  "agentName": "Claude Code - api-server",
  "context": {
    "type": "error",
    "errorMessage": "Type 'string' is not assignable to type 'AuthToken'",
    "errorFile": "src/auth.ts:42",
    "summary": "The auth token type changed upstream and this file needs updating",
    "askQuestion": {
      "question": "Should I update the type or revert the upstream change?",
      "type": "select",
      "options": ["Update the type in auth.ts", "Revert the upstream change", "Skip for now"]
    }
  }
}
```

### ask_user

Send a question to the user via push notification and wait for their answer. By default, this tool **blocks** until the user responds or the timeout is reached - no need to call `wait_for_answer` separately.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| question | string | Yes | The question to ask (max 500 chars) |
| type | "confirm" / "select" / "input" | No | Question type (default: confirm) |
| options | string[] | No | Choices for select type (2-6 options). Required when type is select. |
| placeholder | string | No | Placeholder text for input type (max 200 chars) |
| context | string | No | What the agent is working on, shown above the question (max 500 chars) |
| wait | boolean | No | Wait for the answer before returning (default: true). Set false for manual polling. |
| timeoutMs | integer | No | Max wait time in ms (max 55000). Uses site policy if omitted. |
| agentName | string | No | Identifies which agent is asking. Format: "{Agent} - {project}" (e.g., "Claude Code - myproject") |
| sessionId | string | No | Opaque per-session id of the asking agent, so parallel sessions are attributed separately (max 128 chars) |
| machineId | string | No | Stable machine id of the asking agent, so two machines never collapse into one session (max 128 chars) |
| toolName | string | No | The tool this approval is for (e.g. "Bash"), so the user can choose to always-allow it (max 100 chars) |
| toolTarget | string | No | Compact target of the tool call (e.g. command head "git push" for Bash, or a file extension like ".ts" for Edit/Write). Used to mine always-allow policy suggestions (max 80 chars) |
| callbackUrl | string | No | Webhook URL to POST the answer to when the user responds |
| subscriberIds | string[] | No | Target specific subscriber IDs |
| externalIds | string[] | No | Target by external IDs |
| tags | string[] | No | Target by subscriber tags |

**Returns (when wait=true, default):**
- `{ "answered": true, "value": "yes", "correlationId": "uuid" }` - user responded
- `{ "answered": false, "timedOut": true, "correlationId": "uuid" }` - timeout reached

**Returns (when wait=false):**
- `{ "correlationId": "uuid", "status": "pending", "expiresInSeconds": 600 }` - use `wait_for_answer` to poll

**Returns (when the site policy is notify_only):**
- `{ "correlationId": "uuid", "status": "notified", "answered": false, "mode": "notify_only" }` - the question was pushed but no answer was awaited (the user gets a heads-up, not a blocking prompt). Call `wait_for_answer` if you want to poll for a response anyway.

**Example - confirm (yes/no):**

```json
{
  "question": "Delete the 3 unused migration files?",
  "type": "confirm",
  "context": "Cleaning up old database migrations in db/migrate/",
  "agentName": "Claude Code - myproject"
}
```

**Example - select (multiple choice):**

```json
{
  "question": "Which auth strategy should I use?",
  "type": "select",
  "options": ["JWT tokens", "Session cookies", "OAuth2 + PKCE"],
  "context": "Setting up authentication for the new API endpoints",
  "agentName": "Claude Code - api-server"
}
```

**Example - input (free text):**

```json
{
  "question": "What should the new API endpoint path be?",
  "type": "input",
  "placeholder": "/api/v2/...",
  "context": "Creating a new REST endpoint for user preferences",
  "agentName": "Cursor - frontend"
}
```

### wait_for_answer

Poll for the user's response to a question sent via `ask_user` with `wait: false`. Not needed when using the default blocking mode.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| correlationId | string (uuid) | Yes | The correlationId from ask_user |
| timeoutMs | integer | No | How long to wait (default 30000, max 55000) |

**Returns:**
- `{ "answered": true, "value": "yes" }` - user responded
- `{ "answered": false }` - timeout reached, no answer yet

### cancel_question

Cancel a pending question so it can no longer be answered. Use when the question becomes irrelevant (e.g., you found the answer another way or the user responded in chat).

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| correlationId | string (uuid) | Yes | The correlationId of the question to cancel |

### propose_scope

Propose what a run will touch and block until the user ratifies it. Call **once**, at the start of a multi-step run, before doing work.

The user sees the paths you intend to change, the areas you promise to leave alone, and your definition of done, and approves the whole thing in one tap. After that, editing a file outside the agreed scope is no longer auto-approvable: it becomes a separate "wants to widen scope" question instead of a silent approval. Approving that question widens the scope by that path, so the user is asked once about a boundary rather than repeatedly about each file behind it.

Use glob syntax (`src/**`, `**/*.test.ts`). Shell commands are **not** scoped here; they stay governed by the permission policy.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| doneWhen | string | Yes | What "finished" means for this run. Carried for the human to judge against, never enforced automatically |
| sessionId | string | Yes | Your per-session id. A scope with no session cannot be enforced and must never leak into another run |
| allowedPaths | string[] | No | Globs you intend to change. Omit to propose no path restriction, which the user is told plainly |
| offLimitsPaths | string[] | No | Globs you promise not to touch. These win wherever they overlap `allowedPaths` |
| agentName | string | No | Name of the agent asking, format `"{Agent} - {project}"` |
| timeoutMs | integer | No | How long this call blocks, max 55000 |

**Returns:**
- `{ "ratified": true, "answered": true, "value": "yes", "contract": {...} }` - the contract is live
- `{ "ratified": false, "answered": true, "value": "no" }` - the user declined. Ask what scope they want; do **not** proceed as if they agreed
- `{ "ratified": false, "answered": false }` - no answer yet. The scope is **not** in force

**What enforcement depends on.** The contract is recorded and shown to the user by any MCP client. Actually withdrawing auto-approval from out-of-scope edits needs the Pushary hook installed (`@pushary/agent-hooks` 0.59.0 or later), which is how Claude Code, Codex and Gemini CLI run. Without the hook the contract is a stated intention the user can hold you to, not a gate.

Scope lives for the session only and is never inherited by another run.

**When not to use it.** A single quick edit does not need a scope. And do not propose a new scope mid-run to widen an old one: do the work and let the approval that follows widen it, which is what that flow is for.

### list_sessions

Read-only. Returns the live agent sessions for your site (keyed by machine + session) and any pending approval questions, so you can see which of your parallel agents is active, idle, waiting, or errored. Does NOT start, stop, or steer agents, and sends no notification. Useful when you are one of several agents and want to check whether another session is blocked on a question before acting.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| status | "active" / "idle" / "waiting" / "errored" | No | Filter to sessions with this computed status |
| limit | integer | No | Max number of sessions to return, most recently seen first (1-100) |

**Returns:**
- `{ "sessions": [...], "pendingQuestions": [...] }` - current sessions and open questions for the site

## Permission Gating (REQUIRED)

Before executing any of the following, you MUST call `ask_user` with type "confirm" and wait for approval. Do NOT proceed without an explicit "yes" from the user:

- File deletion (`rm`, `unlink`, any destructive file operation)
- Database mutations (`DROP`, `DELETE`, `TRUNCATE`, migrations)
- Deployment commands (`deploy`, `push`, `publish`, `release`)
- System administration (`systemctl`, `service`, package install/remove)
- Git operations that rewrite history (`reset --hard`, `push --force`, `rebase`)
- Network configuration changes (firewall, DNS, proxy)
- Any command the user has flagged as dangerous

If `ask_user` returns `answered: false`, do NOT execute the command. Send a notification that the operation was skipped due to no response.

This is not optional. Treat it as a hard constraint, not a suggestion.

## Human-in-the-Loop Flow

One tool call - `ask_user` blocks and returns the answer:

```
result = ask_user({
  question: "Which auth strategy should I use?",
  type: "select",
  options: ["JWT tokens", "Session cookies", "OAuth2 + PKCE"],
  context: "Setting up authentication for the new API",
  agentName: "Claude Code - myproject"
})

if result.answered:
    // result.value = "JWT tokens" - proceed with the chosen approach
else:
    // user did not respond - pick the safe default or notify and skip
```

If the user answers in chat before the push response arrives, continue normally and call `cancel_question` with the `correlationId` to clean up.

**A note on how long ask_user blocks:** the wait time and whether it blocks at all are governed by the site's delivery mode, which the user configures (you do not set it). In the default smart mode and push-only mode, ask_user blocks for the policy timeout; in notify-only mode it returns immediately with `answered: false` after sending the push. Always check `answered` rather than assuming the call blocked, and pass `timeoutMs` only when you need a shorter wait than the site policy.

## Identifying Your Agent

Always pass `agentName` when you are one of multiple possible agents the user may be running. The user sees this in the notification title to know which agent is asking.

**Format:** `{Agent Type} - {project or context}`

**Examples:**
- `"Claude Code - pushary repo"`
- `"Hermes - daily-briefing"`
- `"Cursor - frontend refactor"`

## Notification Etiquette

- **Titles under 60 characters.** They get truncated on phone lock screens.
- **Bodies under 200 characters.** Concise summaries, not full explanations.
- **Max 3 notifications per task** unless the user explicitly requests more.
- **Use context for detail.** Put file lists, error traces, and next steps in the context object - not the notification body.
- **Write questions as if talking to a busy person.** The user is on their phone, possibly away from their computer. Be specific: "Delete the 3 unused migration files?" is better than "Should I clean up?"
- **Pick the right question type.** Use confirm for binary decisions, select when options are known, input when they are not.
