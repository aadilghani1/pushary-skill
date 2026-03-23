<p align="center">
  <h1 align="center">Pushary Agent Skill</h1>
  <p align="center">
    Push notifications and human-in-the-loop for AI coding agents.
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
</p>

---

Your AI agent finishes a 20-minute refactor while you're making coffee. Without Pushary, you'd never know until you checked back. With Pushary, you get a push notification on your phone the moment it's done — or a yes/no question on your lock screen when the agent needs a decision.

## How It Works

```
AI Agent  →  MCP Protocol  →  Pushary API  →  Push Notification  →  Your Phone
```

Pushary is an [MCP server](https://modelcontextprotocol.io/) that connects your AI coding agent to push notifications. The agent calls Pushary's tools over the Model Context Protocol, and you receive notifications instantly on any device.

## Quick Start

### 1. Install the skill

```bash
npx skills add pushary/pushary-skill
```

### 2. Add the MCP server

Add this to your agent's MCP configuration:

```json
{
  "mcpServers": {
    "pushary": {
      "url": "https://pushary.com/api/mcp/mcp"
    }
  }
}
```

### 3. Sign up for Pushary

Create your account at [pushary.com/sign-up](https://pushary.com/sign-up?from=ai-coding) to get your API key.

That's it. Your agent will start sending you notifications automatically.

## Tools

The skill exposes 4 MCP tools:

| Tool | Description |
|------|-------------|
| `send_notification` | Send a one-way push notification (task complete, error alert, etc.) |
| `ask_user_yes_no` | Ask the user a yes/no question via push, returns a `correlationId` |
| `wait_for_answer` | Long-poll for the user's response to a yes/no question |
| `cancel_question` | Cancel a pending question that's no longer relevant |

Full tool documentation with parameters, examples, and usage guidelines is in [`skills/pushary/SKILL.md`](skills/pushary/SKILL.md).

## Human-in-the-Loop

The agent can ask you binary decisions via push and wait for your answer:

```
Agent: "Delete the 3 unused migration files?"  →  push to your phone
You: tap "Yes" on your lock screen
Agent: proceeds with deletion
```

The flow uses `ask_user_yes_no` → `wait_for_answer` with automatic retries. Answers persist for 10 minutes, so there's no rush.

## Compatible Agents

Works with any agent that supports [skills.sh](https://skills.sh/) or the Model Context Protocol:

- [Cursor](https://cursor.com/)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Windsurf](https://windsurf.com/)
- [OpenAI Codex](https://openai.com/index/openai-codex/)
- And [39+ more agents](https://skills.sh/)

## Contributing

Contributions are welcome! Please read the [contributing guide](CONTRIBUTING.md) before submitting a pull request.

## Security

If you discover a security vulnerability, please report it responsibly. See [SECURITY.md](SECURITY.md) for details.

## Funding

This project is fully funded by **[RalphNex OÜ](https://ralphnex.com/)**, an Estonian software development agency.

## License

[MIT](LICENSE) — Copyright (c) 2025 RalphNex OÜ
