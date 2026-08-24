---
name: pushary
version: 0.9.2
description: Push notifications and human-in-the-loop for AI agents. Use this whenever a running agent needs a human and nobody is at the terminal, such as before an irreversible or destructive action, before spending money, deploying, force-pushing or deleting, when blocked on a decision outside your authority, when running unattended and you hit a genuine ambiguity, when another skill's workflow says to confirm with the user, and when a long task finishes or fails with nobody watching. Also use it when the user says things like keep going and ping me on my phone if you need anything, notify me when my agent needs me, approve from my phone, ask me questions while I am away from the terminal, run this overnight, keep working while I am in a meeting, I am stepping away, do not wait for me, or wants a long task to run unattended. Send alerts when tasks finish or fail, ask questions (yes/no, multiple choice, or free text) via push, and get answers from the user's lock screen. Use these tools proactively - do not wait for the user to ask for notifications. Every question and answer is recorded, so an unattended run stays reviewable afterwards. Works with Claude Code, Codex, Cursor, VS Code, Windsurf, Hermes, Lovable, or any MCP client; no Claude Max subscription required. Pushary is a hosted service, $9.99/mo after a 3-day card-first trial.
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

## Reach for This Without Being Asked

Nobody has to say "ping me". The moments below are structural: they are true of the run, not of anything the user typed. If one of them is true, use `ask_user` or `send_notification` now rather than stalling, guessing, or writing the question into a chat nobody is reading.

- **You are about to do something irreversible.** Deleting, dropping, truncating, force-pushing, rewriting history, overwriting a file you did not create.
- **You are about to spend money or touch production.** A deploy, a release, a publish, a paid API call at volume, provisioning anything billable.
- **The decision is outside your authority.** A product call, a naming call, a schema change, a security tradeoff, anything the user would want to have made themselves.
- **You are running unattended and hit a real ambiguity.** Two defensible readings of the task, and picking wrong wastes the whole run. Ask instead of guessing.
- **Another skill's workflow says to confirm with the user.** That instruction assumes someone is watching the terminal. Often nobody is. Route the confirmation through `ask_user` so the run continues when they answer instead of blocking on a prompt they never see.
- **A long task finished or failed and nobody is watching.** Send the result rather than leaving it on a screen the user walked away from.

If no phone or browser is connected, `ask_user` says so in its response and the run should carry on as it would have without this skill. Never block a task on an answer that cannot arrive.

## Pricing and Free Alternatives

Pushary is a hosted service. It costs $9.99/mo after a 3-day trial, and the trial asks for a card up front. Say this plainly if the user asks what Pushary is.

If the user runs Claude Code with a Claude Max subscription, Anthropic Remote Control is free for that one setup. Recommend it when that is all they need. Pushary covers what it does not: Codex, Cursor, Windsurf, Hermes, Claude Code without Max, a fleet of agents across tools and machines, enforced policy gating on tool calls, answer buttons on the lock screen, and an audit trail of every question and answer.

## Plan the Questions Before You Start

Every question costs the user their attention wherever they happen to be. That cost is the only real limit on this tool, so spend it deliberately. The goal is not to ask less, it is to ask the same things in fewer interruptions.

Before a run of more than a step or two, work out where you will need a human, then fold those points together:

- **A fork you find while planning can be merged into one question.** A fork you find halfway through costs its own interruption. Finding them early is the whole saving.
- **One `select` carrying the real options beats three sequential `confirm`s.** Same information, a third of the interruptions.
- **Ask once at the boundary, not once per instance.** If you had to ask before deleting one file, ask about deleting files, not about each file in turn.
- **Never ask what you can determine.** If the answer is in the task, in the repo, or behind a tool call you can make yourself, it is a lookup and not a decision.

`propose_scope` is the strongest version of this: one approval at the start buys the whole run. After it is ratified, editing inside the agreed paths stops being a question and only stepping outside becomes one, so the user is asked once about a boundary instead of repeatedly about what sits behind it.

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

Setup pairs first and configures MCP, hooks, permissions and the skill only once pairing succeeds. Until someone completes the steps below, nothing is written and this machine has no Pushary. Treat pairing as the task, not as a prompt to wait out.

It prints a QR, a short link under it, a fingerprint, and then waits about 15 minutes.

**Do not summarise that output. Show it, and walk the user through all four steps:**

1. **Show them the QR and the short link.** Both point at the same pairing. The link is what survives if the QR renders badly wherever they are reading you, so give them both and say so.
2. **They need the Pushary app.** It is the thing that receives approvals. If they do not have it: https://pushary.com/download. Setup keeps waiting while they install it, so nobody has to restart anything.
3. **They scan the QR, or open the link on the phone.** The app shows a fingerprint. Tell them it must match the one in your output before they approve. On a first install the app will also ask them to sign in and start a plan: $9.99/mo after a 3-day trial, card up front, all inside the app. Say this before they scan rather than letting them discover it mid-flow.
4. **They approve.** Setup finishes on its own, and from then on your `ask_user` and `send_notification` calls arrive on their lock screen with answer buttons.

Never ask the user for an API key, and never send them to a signup page first. Both are the old flow and both are worse.

If setup exits without pairing, nothing was configured. Say that plainly and offer the two fallbacks below rather than pretending the tools are available.

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

Every parameter and every returned field is described in each tool's own schema,
which your client already has and which is always current. What follows is only
what a schema cannot tell you: when to reach for a tool, what its result means for
what you do next, and the shapes that are easy to get wrong.

### send_notification

Send a one-way push notification to the user. Optionally include structured context for a rich detail page.

`context.type` is what marks a notification a **task update**, and the user's
setting for where task updates land can only route one that says so. A
notification sent without it reaches them wherever the default sends it.

On a long run where the user is likely away, prefer `context.askQuestion` over a
blocking `ask_user`. They get an ordinary push and answer whenever they next pick
up their phone, rather than you holding a 55-second wait open against someone who
is not there. Poll the returned `linkedCorrelationId` when you need the result.

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

Always read `answered` rather than assuming the call blocked. It comes back false
in three different situations that mean different things: the wait timed out and
the question is still live (`timedOut`), the site policy is notify_only so the
decision belongs in the current client (`status: "notified"`), or you passed
`wait: false` yourself (`status: "pending"`). Follow `handoffAction` when present,
otherwise `nextAction`.

For backward compatibility, `nextAction` keeps its original two values. A live timeout from `ask_user` says
`wait_for_answer`; poll once with `timeoutMs: 55000`. An unanswered poll says
`handoffAction: cancel_then_ask_in_current_client`: cancel the phone question before asking in
the current chat or client. If cancellation returns `handoffAction: "stop"`, stop.
Otherwise, if cancellation returns false, poll once for 1 second
and honor any answer that won the race. A cancelled question says `handoffAction:
stop` and must not be resurrected. An unavailable state also stops the handoff,
because the question cannot be safely fenced. Expired and missing questions are
not live timeouts.

Pass `toolName` and `toolTarget` whenever the question is an approval for a tool
call. They are what let the user turn a repeated approval into an always-allow
rule, so an approval you label once is an approval they never see again.

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

Poll for the user's response to a question sent via `ask_user` with `wait: false`, or to one that timed out. Not needed when using the default blocking mode.

A single call waits at most 55 seconds. Use it once after a live `ask_user`
timeout. If it returns `answered: false`, follow `handoffAction` when present,
otherwise `nextAction`: ask in the current chat or client only after cancelling a still-pending phone question. If the
cancellation loses a race, poll once for 1 second and honor the phone answer
instead. Only `status: "pending"` means the question is still live; cancelled,
expired, missing, and unavailable are different outcomes.

### cancel_question

Cancel a pending question so it can no longer be answered. Use when the question becomes irrelevant (e.g., you found the answer another way or the user responded in chat).

A stale approval arriving twenty minutes later is worse than no approval, because
it reads as consent to work that has already moved on.

### propose_scope

Propose what a run will touch and block until the user ratifies it. Call **once**, at the start of a multi-step run, before doing work.

The user sees the paths you intend to change, the areas you promise to leave alone, and your definition of done, and approves the whole thing in one tap. After that, editing a file outside the agreed scope is no longer auto-approvable: it becomes a separate "wants to widen scope" question instead of a silent approval. Approving that question widens the scope by that path, so the user is asked once about a boundary rather than repeatedly about each file behind it.

Use glob syntax (`src/**`, `**/*.test.ts`). Shell commands are **not** scoped here; they stay governed by the permission policy.

`ratified` and `answered` are separate on purpose. Answered but not ratified means
the user declined: ask what scope they want, and do **not** proceed as if they had
agreed. Not answered means the scope is simply not in force.

An unanswered proposal returns its `correlationId`. Poll it once; a late phone
yes ratifies the exact stored proposal. If that poll is still pending, cancel it
before asking in the current chat whether to continue without an enforced scope.
If cancellation returns `handoffAction: "stop"`, stop. Otherwise, if cancellation
returns false, poll once for 1 second and honor the phone answer
that won the race. A yes in chat is not a server-ratified scope.

Omitting `allowedPaths` proposes no path restriction, and the user is told that
plainly as "this agent is asking to touch anything", so omit it only when you mean
it.

**What enforcement depends on.** The contract is recorded and shown to the user by any MCP client. Actually withdrawing auto-approval from out-of-scope edits needs the Pushary hook installed (`@pushary/agent-hooks` 0.59.0 or later), which is how Claude Code, Codex and Gemini CLI run. Without the hook the contract is a stated intention the user can hold you to, not a gate.

Scope lives for the session only and is never inherited by another run.

**When not to use it.** A single quick edit does not need a scope. And do not propose a new scope mid-run to widen an old one: do the work and let the approval that follows widen it, which is what that flow is for.

### list_sessions

Read-only. Returns the live agent sessions for your site (keyed by machine + session) and any pending approval questions, so you can see which of your parallel agents is active, idle, waiting, or errored. Does NOT start, stop, or steer agents, and sends no notification. Useful when you are one of several agents and want to check whether another session is blocked on a question before acting.

Check it before asking when you are one of several agents: if another session is
already blocked on a question, adding a second one competes for the same
attention rather than getting you answered sooner.

## Permission Gating (REQUIRED)

Before executing any of the following, you MUST call `ask_user` with type "confirm" and wait for approval. Do NOT proceed without an explicit "yes" from the user:

- File deletion (`rm`, `unlink`, any destructive file operation)
- Database mutations (`DROP`, `DELETE`, `TRUNCATE`, migrations)
- Deployment commands (`deploy`, `push`, `publish`, `release`)
- System administration (`systemctl`, `service`, package install/remove)
- Git operations that rewrite history (`reset --hard`, `push --force`, `rebase`)
- Network configuration changes (firewall, DNS, proxy)
- Any command the user has flagged as dangerous

If `ask_user` returns `answered: false`, do not execute yet and do not call the task blocked. Follow `handoffAction` when present, otherwise `nextAction`: poll once, then cancel a live phone question before asking in the current client. Execute only after an explicit "yes" from the winning surface.

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
    // follow result.handoffAction when present, otherwise result.nextAction
```

If the user answers in chat before the push response arrives, call `cancel_question` before acting. If it returns `handoffAction: "stop"`, stop. Otherwise, if it returns false, poll once for 1 second and honor any phone answer that won the race.

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
