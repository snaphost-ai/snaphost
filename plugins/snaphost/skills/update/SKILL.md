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
   document: `content` as UTF-8 text for HTML/text, or base64 for binary, with
   `encoding` telling you which. A site rendered from a file (its `content_kind` is
   markdown, notebook, document, presentation, spreadsheet, pdf, image, video, audio,
   diagram, text or component) returns its rendered
   page, which is not the thing to edit: change the source file and send it again with
   `update_site` the way /snaphost:publish sends it (a text source such as Markdown, CSV,
   Mermaid, a notebook, code or a component as `files`; a binary one as `upload_id` plus
   `filename`, which needs a shell that makes the SnapHost tool calls itself), so the site keeps its kind. When `get_site` reports `original_included` true, the file itself
   is fetched with `get_site_content` and `original: true`. A new version keeps the live
   version's download choice unless `include_original` says otherwise.
3. Apply the user's requested change to that content. Edit the real current document; do
   not regenerate it from scratch, so wording, structure, and styling are preserved.
4. Save with `update_site` (the site id) passing the edited bundle: `html` for a single
   document, `files` (`{path, content}` entries in plain
   text) for a page with its own CSS, JS, JSON or SVG files or for one Markdown, CSV, Mermaid,
   notebook, code or .tsx/.jsx component file, or an `upload_id` from `create_bundle_upload`
   for a rebuild that carries binaries (send the .zip first, with `upload_bundle_part` on this
   connection or a PUT to its `upload_url` where your shell can reach that host). Sending the
   bytes needs a shell that makes the SnapHost tool calls itself; without a shell, a binary the client cannot send as bytes is dropped by the person on the site's own page, https://app.snaphost.ai/sites/<site_id> ("Upload a new version"), never on the drive, which would make a second site. `zipBase64` is a last resort for a multi-file
   bundle; never base64 a single document, and never retry a base64 payload that was
   rejected, switch route instead. At most 3.5 MB once JSON-escaped (a control character other than backspace, tab, newline, form feed and carriage return, `<`, `>`, `&` or any non-ASCII character counts 6 bytes, an emoji 12, and `"`, `\` and `/` 2, so non-ASCII text may count three times its size and markup-heavy text up to six): in practice about 3 MB of plain HTML, about 2 MB when tag-dense; the rest of the 4 MB request body is for the call itself. At most 100 entries and 2 MB of text in total (UTF-8), and 3.5 MB once JSON-escaped (a control character other than backspace, tab, newline, form feed and carriage return, `<`, `>`, `&` or any non-ASCII character counts 6 bytes, an emoji 12, and `"`, `\` and `/` 2, so non-ASCII text may count three times its size and markup-heavy text up to six): in practice about 2 MB of plain English or 1 MB of non-Latin or tag-dense text. The rest of the 4 MB request body is for the call itself.
5. Confirm: the share URL is unchanged and anyone already viewing it is offered the new
   version with a Refresh button. Mention you can roll back via `list_versions` + `rollback_site`.

## Notes

- `get_site_content` is the read half of the round-trip; always read before you write so
  you patch the live document rather than overwrite it blindly.
- The token is configured on the MCP connection; never request or paste it.
