---
name: pushary-chatgpt
version: 0.1.1
description: For ChatGPT and Codex. Plan the work, get the plan approved once, route every decision to the user's phone, and send a push when it is done. Use this whenever a request takes more than one step, contains a choice the user should make rather than you, or will finish while the user is not reading the conversation. In Claude Code, Cursor, Windsurf or Hermes, use the pushary skill instead. Triggers include keep going and ping me when it is done, ask me before you commit to anything, I am stepping away, run this and tell me how it went, and any request where you would otherwise guess at a fork in the road. Every question and answer is recorded, so there is a trail of what was asked and what was decided.
metadata:
  tags: planning, approvals, human-in-the-loop, notifications, push, chatgpt, codex
---

# Pushary: plan first, ask once, always report back

Pushary reaches the person on their phone. They answer from the lock screen or from the dashboard, and either way the question and the answer are recorded.

That changes how you should work. You no longer have to hold a task open in the chat hoping the user is still reading, and you no longer have to guess at a fork because asking would stall you. Work the loop below. It exists to make the number of interruptions small and the number of unrecorded guesses zero.

## The loop

1. Plan the work and find the decisions before you start.
2. Put the plan to the user as one question.
3. Ask through `ask_user` at every real fork. Never guess.
4. Finish with `send_notification`.

## 1. Plan before you act

Before the first action, write out the steps in order and mark the points where you would have to choose.

Finding the forks now is the whole efficiency gain. A fork you find while planning can be folded into one question with the others. A fork you find halfway through costs its own interruption, and interruptions are the expensive part.

While planning, sort every open point into one of three piles:

- **You can answer it.** It is in the request, in the conversation, or derivable from a tool you already have. Answer it and move on. This is not a decision.
- **It only matters if a later step goes a certain way.** Leave it. Ask when you get there, if you get there.
- **The user has to answer it.** Carry it to step 2 and ask it with the plan.

## 2. Put the plan to the user once

One `ask_user` call with `type: "confirm"`.

- `question`: one line, answerable at a glance. "Start on this plan?"
- `context`: the numbered steps and any assumption you made. Under 500 characters, so this is the plan, not an essay.
- `intent`: the user's own request, one line.
- `action`: what you will actually do first.
- `blocker`: why you stopped here. For a plan: "Approving once means I will not stop again unless something is irreversible."

On `value: "yes"`, start. On `"no"`, do not proceed and do not quietly re-plan. Ask what they want instead with `type: "input"`.

If the plan has one genuine fork in it, put the fork in the same call as a `select` rather than sending a confirm now and a select two minutes later.

## 3. Route every decision through ask_user

Pick the type by the shape of the decision:

- `confirm` for yes or no.
- `select` for 2 to 6 options that are mutually exclusive. The answer is the chosen option string.
- `input` for a fact only the user has.

**Fold decisions together.** Three sequential confirms is three interruptions. One `select` carrying the three real options is one.

**Never ask what you can determine.** If the answer sits in the conversation, in the plan they already approved, or behind a tool call you can make, it is a lookup, not a decision.

**Never ask the same class of question twice.** Ask once at the boundary. If you had to ask whether to contact one person, do not ask again for the second and third; ask once about contacting people.

Always ask before:

- Sending anything to a person or a system outside this conversation.
- Spending money, or committing the user to a charge.
- Publishing, deleting, or overwriting anything.
- Anything the user cannot undo themselves in one step.
- Anything they have told you to check with them about.

## Asking when the user is right there

Ask through `ask_user` even when the user is reading along. State the question in your reply too, so someone watching the conversation sees it, then make the call.

Routing it through the tool is what puts the decision on the record and lets them answer from the phone if they walk away mid-task. A decision made only inline is a decision nobody can look up later.

## 4. Waiting, and when to stop waiting

`ask_user` blocks for at most 55 seconds, but the question stays answerable for 10 minutes. Read the response rather than assuming:

- `answered: true`: `value` holds the answer. Act on it.
- `answered: false` with `timedOut: true`: call `wait_for_answer` once with the same `correlationId` and `timeoutMs: 55000`.
- `noDevices: true`: nothing is connected, so waiting is pointless. Follow `handoffAction` immediately and ask in the current client.

Every response carries `answerUrl`, the page where the question is waiting. Print it whenever you tell the user you are waiting, so they can answer in a browser instead of hunting for the notification.

After one empty poll, follow `handoffAction` when present, otherwise `nextAction`. For a live question, cancel it before asking in the current chat. If cancellation returns `handoffAction: "stop"`, stop. Otherwise, if cancellation returns false, poll once for 1 second and honor the answer that won the race. A timeout is not consent.

## 5. Retract what you no longer need

If you work the answer out yourself, or the task moves past the question, call `cancel_question` with the `correlationId`.

A stale approval arriving twenty minutes later is worse than no approval, because it reads as consent to work that has already changed.

## 6. Always finish with send_notification

Every run ends with a notification. Not most runs.

Set `context.type`:

- `task_complete` when the work finished.
- `error` when it did not. Delivered at high urgency.
- `info` for a checkpoint the user asked to be told about.

Fill the context in. A push that only says "Done" is barely better than none, because the user still has to open the conversation to learn anything.

- `summary`: what happened, one or two lines.
- `details`: the specific results, as bullets.
- `filesChanged`: anything you created or modified.
- `nextSteps`: what they should do now, if anything.
- `errorMessage` and `errorFile` when the type is `error`.

If the outcome raises a follow-up question, put it in `context.askQuestion` instead of sending a second notification. The response carries a `linkedCorrelationId` you can poll with `wait_for_answer`.

## Naming yourself

Pass `agentName` as `"{host} - {short task name}"` on every call, where host is the product you are actually running in: `"ChatGPT - invoice audit"`, `"Codex - api refactor"`. The notification then says which piece of work is asking rather than just "your AI agent".

**Never name a host you are not.** The name is what the activity feed, the fleet board and the weekly digest group by, so a Claude Code session that calls itself ChatGPT is filed under ChatGPT and the user's own records stop being true.

Pass a stable `sessionId` for the whole conversation. Parallel work is then attributed separately instead of collapsing into one session.

## In Codex

The same plugin loads in Codex, which can read and write files. There, ratify the plan with `propose_scope` rather than `ask_user`: pass `allowedPaths`, `offLimitsPaths`, `doneWhen`, and a `sessionId`. Approving once means editing inside those globs stops being a question, and only going outside them becomes one. Shell commands are not scoped by it.

Do not call `propose_scope` in ChatGPT. With no paths it tells the user "this agent is asking to touch anything", which is not true here, and it will read as a far bigger request than the one you are making.

## Tools

| Tool | Use it for |
|------|-----------|
| `ask_user` | Every decision. Blocks up to 55s and returns `value`. |
| `wait_for_answer` | Keep waiting on a `correlationId` that timed out. |
| `cancel_question` | Retract a question you no longer need answered. |
| `send_notification` | The final report, and any checkpoint the user asked for. |
| `propose_scope` | Codex only. Ratify a file scope once at the start. |
