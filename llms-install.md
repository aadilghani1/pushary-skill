# Installing Pushary

Pushary is a hosted remote MCP server. There is nothing to clone, build, or run locally.

## If you can run shell commands, do this instead

One command, and it needs no account and no API key up front:

```bash
npx @pushary/agent-hooks@latest setup
```

It writes the MCP config, hooks and permissions for you, then prints a QR code.
Show the user your output and ask them to scan it with their phone camera: that
pairs their Pushary app, which is what hands this machine its key. Everything
below is only for clients that cannot run a command.

## What you need for a manual install

A Pushary API key in the format `pk_xxx.sk_xxx`. Get one at https://pushary.com/sign-up
(the agent plan is $9.99/mo after a 3 day trial). The key is the only required value.

## Steps

1. Ask the user for their Pushary API key. Do not invent one and do not continue without it.

2. Add the server to the MCP settings file for this client. For Cline that is
   `cline_mcp_settings.json`; for Claude Code it is `~/.claude/settings.json`:

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

Replace `pk_xxx.sk_xxx` with the user's key.

3. Reload MCP servers. `tools/list` should return `send_notification`, `ask_user`,
   `wait_for_answer` and `cancel_question` among others.

## Verifying the install

Call `send_notification` with a short title and body. The user should get a push within a
few seconds. If nothing arrives, they have not paired a device yet: send them to
https://pushary.com/dashboard/agent/settings to connect their phone or browser.

## Notes for clients that cannot send headers

The key can ride in the URL instead:

```
https://pushary.com/api/mcp/mcp?api_key=pk_xxx.sk_xxx
```

An SSE transport is available at `https://pushary.com/api/mcp/sse`.

`initialize` and `tools/list` answer without a key, so introspection works before setup.
Every tool call needs the key.

## When to use the tools

- `ask_user` when you need a decision from the user, then `wait_for_answer`. Types are
  `confirm` for yes/no, `select` for a fixed set of options, `input` for free text.
- `send_notification` when a long task finishes or you hit an error you cannot resolve.
- `cancel_question` to retract a pending question that is no longer needed.

Pass `agentName` so the user knows which session is asking.
