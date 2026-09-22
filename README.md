# SnapHost for AI agents

[SnapHost](https://snaphost.ai) turns a document or a built site into a private, shareable live
website. This repository is the public home of its MCP server listing and Claude Code plugin.

The server itself is hosted: `https://app.snaphost.ai/api/mcp` (Streamable HTTP, OAuth 2.1 with
PKCE and dynamic client registration). There is nothing to run locally and no token to paste; on
first use your browser opens so you can click **Authorize**.

## Install

**Any client, one command** (Claude Code, Cursor, VS Code, Windsurf, Codex CLI, Gemini CLI):

```bash
npx snaphost connect
```

**Claude Code plugin** (adds the `/snaphost:*` skills and the connection in one go):

```
/plugin marketplace add snaphost-ai/snaphost
/plugin install snaphost@snaphost
```

**Cursor**: [Add to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=snaphost&config=eyJ1cmwiOiJodHRwczovL2FwcC5zbmFwaG9zdC5haS9hcGkvbWNwIn0=)

**VS Code**: [Install in VS Code](vscode:mcp/install?%7B%22name%22%3A%22snaphost%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapp.snaphost.ai%2Fapi%2Fmcp%22%7D)

**Claude Code by hand**:

```bash
claude mcp add --transport http snaphost https://app.snaphost.ai/api/mcp
```

**Claude on the web or the desktop app**: Settings, Connectors, Add custom connector, paste the
endpoint URL. Claude detects OAuth; leave the defaults and click Add.

## What the AI can do

Publish a page or a bundle and get a stable link; update it in place; control who can view it
(public, allowlist, password, expiry); take it offline and back; roll back to an earlier version;
put it on a custom domain; and give the page a data backend (forms, waitlists, RSVPs,
leaderboards) through `window.SNAPHOST`. The full tool list with read/write/destructive
annotations is at https://snaphost.ai/docs/mcp-server.

## Layout

- `server.json`: the entry published to the [official MCP registry](https://registry.modelcontextprotocol.io) as `ai.snaphost/snaphost`.
- `.claude-plugin/marketplace.json` and `plugins/snaphost/`: the Claude Code plugin marketplace.
- `glama.json`: maintainers for the Glama listing.

These files are generated from the SnapHost application and refreshed by
[`sync.yml`](.github/workflows/sync.yml); the version bumps there whenever a tool, a skill, or a
description changes. Please open issues for problems with the listing or the plugin; the server's
source is not in this repository.

## Links

- Docs: https://snaphost.ai/docs/mcp-server and https://snaphost.ai/docs/skills
- Privacy: https://snaphost.ai/privacy
- Support: hello@snaphost.ai
