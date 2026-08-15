# The ChatGPT and Codex plugin

This package is two distributions of one skill set. `.claude-plugin/plugin.json` is the
Claude Code plugin. `.codex-plugin/plugin.json` is the ChatGPT and Codex plugin. They share
`skills/`, `logo.png` and `.mcp.json`; nothing is duplicated.

Published plugins land in **one directory shared by ChatGPT and Codex**, so this is a single
submission that reaches both.

## Testing it locally, before OpenAI is involved

The repo root carries `.agents/plugins/marketplace.json`, a local marketplace pointing at
this directory. That makes the plugin installable and runnable now, with no plugin id, no
submission and no review:

```bash
codex plugin marketplace add .           # from the repo root
codex plugin marketplace list            # expect "pushary-local"
```

Then install `pushary` from the Plugins directory and run the evaluation set: a direct ask
("notify me when this finishes"), an indirect one ("I'm stepping away"), an incomplete one,
and a case that should *not* activate it. This is the loop to iterate the skills in.

## What is deliberately not here

**Hooks.** `.claude-plugin/plugin.json` points at `hooks/hooks.json`; this manifest does
not. Those hooks are Claude Code's shapes (`PreToolUse` JSON on stdin, `pushary-hook` and
friends), and OpenAI's guidance is explicit that hooks must not be required for the core
ChatGPT workflow. Shipping them here would at best do nothing and at worst break an install.
Codex enforcement stays with the CLI, which is the stronger integration anyway.

**`.app.json`.** For a hosted plugin, OpenAI wants the MCP server registered through the
portal and referenced by id, not the keyed HTTP config in `.mcp.json`. That id does not
exist yet. `.mcp.json` is still correct for a Codex user running locally with
`PUSHARY_API_KEY` set, so it stays; `.app.json` is added alongside it once we hold an id.

**Screenshots.** `interface.screenshots` is omitted rather than pointed at files that are
not in this package. The submission form collects them separately.

## What has to happen before submission

1. Resolve the auth-challenge conflict. ChatGPT wants a 200 carrying
   `_meta["mcp/www_authenticate"]`; Claude's lazy auth wants a 401. See
   `docs/planning/CHATGPT-PLUGIN-NON-TECHNICAL-PATH-2026-08.md` §4.2. Until this lands, the
   sign-in a ChatGPT user starts cannot complete.
2. Register the MCP server in ChatGPT Developer mode, copy the plugin id from the browser
   URL, and set `NEXT_PUBLIC_CHATGPT_PLUGIN_ID` so the one-click button appears in
   onboarding and settings.
3. Publisher identity verification and domain verification, both of which have queues and
   neither of which depends on code.

## Keeping the two manifests honest

`version` is tracked per distribution and they will drift; that is fine, they are separate
artifacts. What must not drift is the description of what the skill does, because OpenAI
review checks description accuracy against behaviour. The skill bodies themselves are
generated from one source and gated in CI by `node scripts/sync-skill.mjs --check`.
