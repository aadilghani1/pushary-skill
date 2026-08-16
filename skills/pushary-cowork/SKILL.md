---
name: pushary-cowork
version: 0.3.1
description: Phone notifications and human-in-the-loop for Claude Cowork through the Pushary connector. Use inside a Cowork session whenever you need a human and nobody is watching the session, such as before an irreversible or destructive action, before spending money, deploying, force-pushing or deleting, when blocked on a decision outside your authority, when running unattended and you hit a genuine ambiguity, when another skill's workflow says to confirm with the user, and when a task finishes or fails with nobody watching. Also use it when the user says things like ping me on my phone when this is done, ask me before doing anything risky, keep me in the loop while I am away, or notify me if you get stuck. Sends completion alerts, asks questions (yes/no, multiple choice, or free text) via push, and gets answers from the user's lock screen. Cooperative only, Cowork has no hooks. Pushary is a hosted service, $9.99/mo after a 3-day card-first trial.
metadata:
  tags: notifications, push, mcp, human-in-the-loop, cowork, claude, alerts, approvals
---

# Pushary for Claude Cowork

Pushary is connected as a custom connector. It reaches the user on their phone, where they answer from the lock screen. Use it proactively. Do not wait for the user to ask.

## Ask in as few interruptions as possible

Every question costs the user their attention wherever they are. Before a run of more than a step or two, work out where you will need a human and fold those points together: one `select` carrying the real options beats three `confirm`s in a row, ask once at a boundary rather than once per instance, and never ask what you can determine yourself from the task or from a tool call you can make.

## When to reach out

- **You need a decision or a clarifying answer.** Call `ask_user` instead of guessing or stalling. Use type `confirm` for yes or no, `select` for a fixed set of options, and `input` for free text.
- **You are about to do something risky or irreversible.** Deleting or overwriting files, spending money, sending anything external, bulk changes: call `ask_user` with type `confirm` first and wait for approval.
- **A task finishes.** Call `send_notification` with a short summary of what changed, and pass `context.type` as `task_complete`. That is what marks it a task update, and the user's setting for where task updates land can only route one that says so.
- **You are blocked or hit an error you cannot resolve.** Call `send_notification` with `context.type` as `error` so the user knows, and `ask_user` if you need a decision to continue.
- **Another skill's workflow says to confirm with the user.** That instruction assumes someone is watching the session. Often nobody is. Route the confirmation through `ask_user` so the run continues when they answer, instead of stalling on a prompt they never see.

## How to wait for answers

- A question stays open for 10 minutes. `wait_for_answer` blocks for up to about 55 seconds per call. If it returns without an answer, call `wait_for_answer` again with the same question id before giving up.
- On long tasks where the user might be away, prefer `send_notification` with `context.askQuestion` over a blocking `ask_user`. The user gets a normal push and answers from the notification page whenever they pick up their phone. Poll the returned `linkedCorrelationId` with `wait_for_answer` when you need the result.
- Use `cancel_question` to retract a question that is no longer needed.

## Conventions

- Pass `agentName` as `Claude Cowork - <task name>` on every call, so the user knows which session is asking and their dashboard groups the session correctly.
- Keep questions short and decision-shaped. One sentence of context, then the ask. The user is reading a lock screen, not a report.
- Do not ask through Pushary for things you can safely decide yourself. Reserve it for real decisions, risky steps, completions, and errors, so a ping always means something.

## If the connector is missing

If no Pushary tools are available in this session, the connector is not enabled. Tell the user once: Pushary is not connected in this session. Enable it under Customize, Connectors, or set it up at https://pushary.com/docs/agents/guides/claude-desktop. Then continue the task without it.
