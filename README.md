<p align="center">
  <h1 align="center">Pushary Agent Skill</h1>
  <p align="center">
    Push notifications and human-in-the-loop for AI agents. Approve from your phone.
    <br />
    <a href="https://pushary.com/ai-coding"><strong>Get started</strong></a>
    &nbsp;&middot;&nbsp;
    <a href="https://skills.sh/">Skills directory</a>
    &nbsp;&middot;&nbsp;
    <a href="https://github.com/pushary/pushary-skill/issues">Report a bug</a>
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License" /></a>
  <a href="https://skills.sh/"><img src="https://img.shields.io/badge/skills.sh-listed-brightgreen" alt="skills.sh" /></a>
  <a href="https://www.npmjs.com/package/@pushary/agent-hooks"><img src="https://img.shields.io/npm/v/@pushary/agent-hooks" alt="npm" /></a>
  <a href="https://pypi.org/project/hermes-plugin-pushary/"><img src="https://img.shields.io/pypi/v/hermes-plugin-pushary" alt="PyPI" /></a>
  <a href="https://smithery.ai/servers/aadil/pushary"><img src="https://smithery.ai/badge/aadil/pushary" alt="Smithery" /></a>
  <a href="https://glama.ai/mcp/servers/Pushary/pushary-skill"><img src="https://glama.ai/mcp/servers/Pushary/pushary-skill/badges/score.svg" alt="Glama score" /></a>
</p>

---

Your AI agent finishes a 20-minute refactor while you're making coffee. Without Pushary, you'd never know until you checked back. With Pushary, you get a push notification on your phone the moment it's done - or a question on your lock screen when the agent needs a decision. It's the tool behind "keep going, ping me on my phone if you need anything".

Pushary is a hosted service: $9.99/mo after a 3-day card-first trial. It works with Claude Code, Claude Cowork, Codex, Cursor, Windsurf, Hermes, Lovable, or any MCP client, and it does not need a Claude Max subscription. If you run Claude Code with Claude Max, Anthropic Remote Control covers that one setup for free. Pushary covers what it does not: Codex, Cursor, Claude Code without Max, a cross-agent fleet, enforced policy gating, lock screen answer buttons, and an audit trail of every question and answer.

## How It Works

```
AI Agent  ->  MCP Protocol  ->  Pushary API  ->  Push Notification  ->  Your Phone
```

Pushary is an [MCP server](https://modelcontextprotocol.io/) that connects your AI coding agent to push notifications. The agent calls Pushary's tools over the Model Context Protocol, and you receive notifications instantly on any device.

**Three question types** - yes/no confirmations, multiple choice, and free text input. Your agent picks the right one for the situation.

**Rich context notifications** - agents can include file changes, error details, and suggested next steps in a detail page you see when tapping the notification.

**Agent identification** - when you run multiple agents, each notification shows which agent is asking so you always know what you're responding to.

**Permission hooks** - route Claude Code's tool approval prompts through push notifications so you can approve or deny from your phone.

---

## Setup: Claude Code

### Option 0: Claude Code plugin (one command)

This repo is a Claude Code plugin. Install it and you get the MCP tools, the permission hooks, and the skill in one step:

```
/plugin marketplace add Pushary/pushary-skill
/plugin install pushary
```

Set `PUSHARY_API_KEY` in your environment (get a key at [pushary.com](https://pushary.com), $9.99/mo after a 3-day trial). Claude Code expands `${PUSHARY_API_KEY}` in the plugin's MCP config.

Pick one install path. If you already ran `npx @pushary/agent-hooks setup`, the same hooks live in your `~/.claude/settings.json`; installing the plugin on top runs each hook twice. Either uninstall the plugin or run `npx @pushary/agent-hooks clean` before switching.

### Option A: MCP Server (notifications + questions)

The agent calls Pushary tools when it wants to notify you or ask a question.

**1. Sign up** at [pushary.com/sign-up](https://pushary.com/sign-up?from=ai-coding) and get your API key.

**2. Install the skill:**

```bash
npx skills add Pushary/pushary-skill
```

**3. Add the MCP server** to your Claude Code settings (`~/.claude/settings.json` or project `.claude/settings.json`):

```json
{
  "mcpServers": {
    "pushary": {
      "url": "https://pushary.com/api/mcp/mcp",
      "headers": {
        "Authorization": "Bearer pk_xxx.sk_xxx"
      }
    }
  }
}
```

Replace `pk_xxx.sk_xxx` with your API key.

**4. Enable notifications** on your phone by visiting your Pushary dashboard and allowing browser notifications.

That's it. The agent will proactively send you notifications when tasks complete, errors occur, or decisions are needed.

### Option B: Permission Hooks (approve/deny tools via push)

Route Claude Code's built-in permission prompts through push notifications. When the agent wants to run a command or edit a file, you get a push notification to approve or deny from your phone.

**1. Install the hook package:**

```bash
npm install -g @pushary/agent-hooks
```

**2. Set your API key** in your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
export PUSHARY_API_KEY="pk_xxx.sk_xxx"
```

**3. Add the hook** to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "pushary-hook",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

**4. Configure timeout policies** in your [Pushary dashboard](https://pushary.com/dashboard/agent/policies):

| Tool | Timeout | If no response |
|------|---------|----------------|
| Bash | 60s | Auto-deny |
| Write | 60s | Ask in terminal |
| Edit | 45s | Ask in terminal |
| Read | 0s | Auto-approve |

When you don't respond to a push notification in time, the configured fallback kicks in: auto-approve, auto-deny, or fall back to the normal terminal permission prompt.

### Option C: Both (recommended)

Use Option A and Option B together. The MCP server handles notifications and voluntary questions. The permission hook handles tool approvals. They use the same API key and dashboard.

---

## Setup: Hermes Agent

### Option A: Native Plugin (recommended)

Full integration with native Hermes tools and automatic error notifications.

**1. Install the plugin:**

```bash
pip install hermes-plugin-pushary
```

**2. Enable it:**

```bash
hermes plugins enable pushary
```

**3. Set your API key:**

```bash
export PUSHARY_API_KEY="pk_xxx.sk_xxx"
```

**4. (Optional) Set your agent name:**

```bash
export PUSHARY_AGENT_NAME="Hermes - my-project"
```

The plugin registers 4 native tools (`pushary_notify`, `pushary_ask`, `pushary_wait`, `pushary_cancel`) and automatically sends push notifications when tools return errors.

Set `PUSHARY_AUTO_NOTIFY_SESSION_END=1` to also get notified when a Hermes session ends.

### Option B: MCP Server + Skill

If you prefer MCP over the native plugin:

**1. Add to `~/.hermes/config.yaml`:**

```yaml
mcp:
  servers:
    pushary:
      url: https://pushary.com/api/mcp/sse
      headers:
        Authorization: "Bearer ${PUSHARY_API_KEY}"
```

**2. Install the skill:**

```bash
hermes skills tap add Pushary/pushary-skill
hermes skills install pushary-hermes
```

---

## Setup: OpenAI Codex

Codex has native MCP support. One command:

**1. Set your API key** in your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
export PUSHARY_API_KEY="pk_xxx.sk_xxx"
```

**2. Add the MCP server:**

```bash
codex mcp add pushary --url https://pushary.com/api/mcp/mcp --bearer-token-env-var PUSHARY_API_KEY
```

That's it. Codex now has access to all Pushary tools - notifications, questions, and rich context.

**Or use the setup wizard** (configures everything including the API key):

```bash
npx @pushary/agent-hooks setup
```

---

## Setup: Cursor / Windsurf / Other MCP Agents

**1. Add the MCP server** to your agent's MCP config (usually `.cursor/mcp.json` or similar):

```json
{
  "mcpServers": {
    "pushary": {
      "url": "https://pushary.com/api/mcp/mcp",
      "headers": {
        "Authorization": "Bearer pk_xxx.sk_xxx"
      }
    }
  }
}
```

**2. Install the skill** (if your agent supports skills.sh):

```bash
npx skills add Pushary/pushary-skill
```

---

## Setup: Lovable

[Lovable](https://lovable.dev) is a hosted app builder, so there is no local install step (`npx skills add` does not apply). You connect the MCP server in Lovable's UI, then paste the skill into Lovable's Knowledge so the agent uses it on its own.

**1. Connect the MCP server.** In Lovable, go to **Settings -> Connectors -> Personal connectors** (paid Lovable plans), click **New MCP server**, set the URL to `https://pushary.com/api/mcp/mcp`, choose **Bearer token**, and paste your API key (`pk_xxx.sk_xxx`).

**2. Add the skill to Knowledge.** Lovable cannot install skills via `npx`, so paste the skill guidance into **Settings -> Knowledge**. Use the condensed [`SKILL-LITE.md`](skills/pushary/SKILL-LITE.md) as the source. This makes the Lovable agent notify you when a build finishes and ask before risky changes, without you prompting each time.

Lovable gets notifications and questions only (no enforced gate, since it has no permission hook). Full walkthrough: [Lovable guide](https://pushary.com/docs/agents/guides/lovable).

---

## Setup: Claude Cowork

[Claude Cowork](https://claude.com/) is Anthropic's agentic workspace inside the Claude apps. It is a hosted surface (`npx skills add` does not apply), so you connect Pushary as a custom connector and add the skill through Cowork's own skill upload.

**1. Connect the MCP server.** Get your connector link from the [Pushary dashboard](https://pushary.com/dashboard/agent/settings) (**Settings -> Connections**, Claude section). In Claude, open **Settings -> Connectors -> Add custom connector**, leave the OAuth fields empty, and paste the link. Connectors are account level, so the same connector is available inside Cowork; enable it in a session under **Customize -> Connectors**.

**2. Add the skill.** Zip the [`skills/pushary-cowork`](skills/pushary-cowork) folder and upload it in Cowork under **Customize -> Skills** (skills need code execution enabled). Alternatively, paste the standing instructions block from your Pushary dashboard into Claude **Settings -> Cowork**, so every session pings you on completion and asks before risky steps without you prompting each time.

Cowork gets notifications and questions only (no enforced gate; Cowork exposes no hooks). Full walkthrough: [Claude Cowork guide](https://pushary.com/docs/agents/guides/claude-desktop). Dedicated plugin repo: [Pushary/cowork-plugin](https://github.com/Pushary/cowork-plugin).

---

## Tools

The skill exposes 5 MCP tools:

| Tool | Description |
|------|-------------|
| `send_notification` | Send a push notification with optional rich context (file changes, errors, next steps) |
| `ask_user` | Ask the user a question via push - yes/no, multiple choice, or free text |
| `wait_for_answer` | Long-poll for the user's response to a question |
| `cancel_question` | Cancel a pending question that's no longer relevant |
| `list_sessions` | Read-only view of your live agent sessions and pending questions |

Full tool documentation with parameters, examples, and usage guidelines is in [`skills/pushary/SKILL.md`](skills/pushary/SKILL.md).

## Human-in-the-Loop

The agent can ask you decisions via push and wait for your answer:

```
Agent: "Which auth strategy should I use?"
  Options: JWT tokens / Session cookies / OAuth2 + PKCE
  -> push to your phone

You: tap "JWT tokens"

Agent: proceeds with JWT implementation
```

Supports three question types:
- **Confirm** - yes/no binary decisions
- **Select** - pick from 2-6 options
- **Input** - free text response

The flow uses `ask_user` -> `wait_for_answer` with automatic retries. Answers persist for 10 minutes, so there's no rush.

## Packages

| Package | Registry | Description |
|---------|----------|-------------|
| [`@pushary/agent-hooks`](https://www.npmjs.com/package/@pushary/agent-hooks) | npm | Claude Code permission hooks |
| [`hermes-plugin-pushary`](https://pypi.org/project/hermes-plugin-pushary/) | PyPI | Hermes Agent native plugin |

## Compatible Agents

Works with any agent that supports [skills.sh](https://skills.sh/) or the Model Context Protocol:

- [Claude Code](https://code.claude.com/) (MCP + permission hooks)
- [Claude Cowork](https://claude.com/) (custom connector + skill upload)
- [Hermes Agent](https://hermes-agent.nousresearch.com/) (native plugin or MCP)
- [Cursor](https://cursor.com/) (MCP)
- [Windsurf](https://windsurf.com/) (MCP)
- [Lovable](https://lovable.dev/) (MCP connector + skill via Knowledge)
- [OpenAI Codex](https://openai.com/index/openai-codex/) (MCP)
- And [39+ more agents](https://skills.sh/)

## Contributing

Contributions are welcome! Please read the [contributing guide](CONTRIBUTING.md) before submitting a pull request.

## Security

If you discover a security vulnerability, please report it responsibly. See [SECURITY.md](SECURITY.md) for details.

## Funding

This project is fully funded by **[RalphNex OU](https://ralphnex.com/)**, an Estonian software development agency.

## License

[MIT](LICENSE) - Copyright (c) 2025 RalphNex OU
