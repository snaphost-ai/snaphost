---
name: update
description: "Update an existing SnapHost site in place, keeping its URL."
---

Use this skill to change the content of an existing SnapHost site without changing its
URL, visibility, or allowlist.

## Steps

1. Determine the target site. Unless a workspace was already chosen in this session, call
   `list_workspaces`; with more than one workspace, or a `note`, run `/snaphost:workspace`
   and pass its `workspace_id` on every call here. If the user gave a site id, use it.
   Otherwise call `list_sites` and match by title, confirming the right one with the user if
   ambiguous. A site that is not listed is in another workspace: sites are looked up per
   workspace, so pick the right one rather than concluding it is gone.
2. Fetch the current content with `get_site_content` (the site id). It returns the entry
   document: `content` as UTF-8 text for HTML/Markdown, or base64 for binary, with
   `encoding` telling you which.
3. Apply the user's requested change to that content. Edit the real current document; do
   not regenerate it from scratch, so wording, structure, and styling are preserved.
4. Save with `update_site` (the site id) passing the edited bundle: `html` for a single
   document (the safe default at any size), or an `upload_id` from `create_bundle_upload` for
   a multi-file rebuild (send the .zip first, with `upload_bundle_part` on this connection or
   a PUT to its `upload_url` where your shell can reach that host). `zipBase64` is a last
   resort for a small multi-file bundle; never base64 a single document, and never retry a
   base64 payload that was rejected, switch route instead.
5. Confirm: the share URL is unchanged and anyone already viewing it is offered the new
   version with a Refresh button. Mention you can roll back via `list_versions` + `rollback_site`.

## Notes

- `get_site_content` is the read half of the round-trip; always read before you write so
  you patch the live document rather than overwrite it blindly.
- The token is configured on the MCP connection; never request or paste it.
