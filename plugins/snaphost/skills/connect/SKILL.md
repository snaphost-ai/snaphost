---
name: connect
description: "Connect this AI to SnapHost over MCP and verify it can manage your sites."
---

Connect this AI client to SnapHost over MCP so it can publish, update, and share the user's
sites. Goal: **one browser click, no token in chat, no copy-paste.** Drive it as a single clear
step at a time, and never hand the user a list of actions to do at once.

## Talking to the user (follow exactly)

- **One step at a time.** Say the single next thing to do, then stop and wait.
- **Make links clickable.** Put any authorization URL on its **own line, raw**: just the
  `https://…` with nothing else on the line (no markdown link syntax, backticks, or trailing
  punctuation). That is what renders as a clickable link.
- **One live link only.** Each new authorization attempt invalidates the previous link. Show only
  the newest URL and tell the user to ignore any earlier ones.
- **Restarts, in plain words.** A newly added MCP server loads only when Claude restarts. When that
  is needed, say it exactly like this: *"Fully quit Claude and reopen it (a reload is not enough),
  then tell me you're back."* Then stop and wait.
- **Never** ask the user to paste a secret or API token into the chat.

## Steps

1. Call `list_workspaces`. If it returns anything, you are already connected; go to step 4.
2. If SnapHost's tools are not available at all, the server is not registered yet. Register it
   one of these ways, then have the user fully quit and reopen Claude:

   - Installed from the plugin marketplace (`/plugin marketplace add snaphost-ai/snaphost`
     then `/plugin install snaphost@snaphost`)? The plugin already carries the server; a restart
     is all that is missing.
   - Otherwise add it at user scope (so it works in every project):

   ```bash
   claude mcp add --transport http snaphost https://app.snaphost.ai/api/mcp -s user
   ```

   The same one-liner for any client: `npx snaphost connect` configures Claude Code, Cursor,
   VS Code, Windsurf, Codex, and Gemini CLI in one go.

3. Once the server is loaded, start the browser flow. When the client surfaces an authorization
   URL, present it as one step: put the raw `https://…` URL on its own line and tell the user to
   **click Authorize** (the connection lasts until they revoke it). The credential is then
   negotiated machine-to-machine; no token ever enters the chat.
4. Confirm with `list_workspaces`: it proves the connection AND shows its reach. The connection
   belongs to the person and covers their personal space plus every workspace they are a member
   of, whichever workspace was open when they authorized. Show the user what came back.
   - More than one workspace, or a `note` in the result: hand off to `/snaphost:workspace`
     before doing anything else, so work lands where the user means it to (a write without a
     `workspace_id` is refused on such a connection; an unnamed read is the personal space).
   - Exactly one workspace: call `list_sites` in it. An empty list means it works (no sites
     yet); offer `/snaphost:publish`. Tell the user they're ready.

## If the Authorize page errors

The redirect lands on a `http://localhost` address handled by Claude Code itself (not SnapHost),
so a "this site can't be reached" page is usually a loopback IPv4/IPv6 mismatch or a timed-out
listener, not a SnapHost failure. Recover in order:

1. Have the user fully quit and reopen Claude, then click the fresh link **promptly**; these links
   expire within a minute or two.
2. If it still errors, the URL now in their address bar
   (`http://localhost:<port>/callback?code=…&state=…`) is still valid; ask them to paste that one
   URL so the client can finish. It is a single-use code, not a reusable secret.

## Token fallback (last resort, only if the browser flow is truly unavailable)

The user creates an API token in the dashboard under **API tokens** (shown once) and runs this in
*their own* terminal, replacing `<token>`, so the secret stays in their shell, never in this chat:

```bash
claude mcp add --transport http snaphost https://app.snaphost.ai/api/mcp --header "Authorization: Bearer <token>" -s user
```

`-s user` stores it in the user's own Claude config; never echo, write, or commit it.

## Notes

- The hosted endpoint is `https://app.snaphost.ai/api/mcp`. Any client that supports HTTP (Streamable) MCP with
  OAuth works: point it at that URL and approve the browser prompt.
- After setup the credential lives on the MCP connection; `/snaphost:publish`,
  `/snaphost:update`, and `/snaphost:share` never ask for it again.
