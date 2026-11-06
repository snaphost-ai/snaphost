---
name: publish
description: "Publish the current document to SnapHost as a live, shareable site."
---

Use this skill to publish something the user just built or has to hand (an HTML file, a
report, a notebook, a Word document, a slide deck, a spreadsheet, a PDF, an image, a
bundle) to SnapHost as a live site with a stable, shareable link.

## Steps

1. Identify the document to publish. If it is a single self-contained HTML file, read it.
   If it is a folder of text files (a page with its CSS and JS), it goes as `files` in step 4;
   a folder with images or fonts is zipped (a shell step, see step 4; entry document at the
   root, e.g. `index.html`). A single
   file that is not HTML (Markdown, a notebook, a PDF, a .docx/.pptx/.xlsx, a CSV, an image,
   a video, an audio, a Mermaid diagram or a text file, or a single React component as a
   .tsx/.jsx with a default export) is sent as it is: see step 4. A component may import
   only react, react-dom, recharts, lucide-react, d3, framer-motion and @radix-ui/react-*;
   rewrite any other import (a local `./component`, a UI kit) into the file before sending it.
2. Choose a stable, human `slug` derived from the title (e.g. "Q3 Investor Update" →
   `q3-investor-update`). The slug makes publishing idempotent.
3. Unless a workspace was already chosen in this session, call `list_workspaces`. If it returns
   more than one workspace, or a `note`, run `/snaphost:workspace` to choose one, then come
   back here: a publish without a `workspace_id` is refused on such a connection, and the
   personal space is usually not where a team's sites live. Pass the chosen `workspace_id` on
   every call below, the upload calls included: an upload belongs to the workspace it was
   created in.
4. Call `publish_site` with a `title`, the `slug`, and the bundle. For a single HTML document
   use `html`. At most 3.5 MB once JSON-escaped (a control character other than backspace, tab, newline, form feed and carriage return, `<`, `>`, `&` or any non-ASCII character counts 6 bytes, an emoji 12, and `"`, `\` and `/` 2, so non-ASCII text may count three times its size and markup-heavy text up to six): in practice about 3 MB of plain HTML, about 2 MB when tag-dense; the rest of the 4 MB request body is for the call itself. A bigger page is trimmed, split, or sent through
   `create_bundle_upload` (a shell step, see below). For a page with its own CSS, JS, JSON or
   SVG files, or for one Markdown, CSV, Mermaid, notebook, code or .tsx/.jsx component file, use
   `files`: an array of `{path, content}` in plain text (name the entry `index.html`; a lone
   file is rendered as its kind under its path). At most 100 entries and 2 MB of text in total (UTF-8), and 3.5 MB once JSON-escaped (a control character other than backspace, tab, newline, form feed and carriage return, `<`, `>`, `&` or any non-ASCII character counts 6 bytes, an emoji 12, and `"`, `\` and `/` 2, so non-ASCII text may count three times its size and markup-heavy text up to six): in practice about 2 MB of plain English or 1 MB of non-Latin or tag-dense text. The rest of the 4 MB request body is for the call itself. `html` and `files` need no shell, no encoding and no
   network of your own, so they are the routes when you have only a chat window. Everything
   below this line needs a shell that makes the SnapHost tool calls itself. A shell here is a coding agent in a terminal (Claude Code, Cursor, Codex CLI); a chat app's code sandbox does not count, since its output would have to be retyped into the call. Without a shell, do not open an upload and do not build base64. Ask the person to drop the file (downloaded from the chat if it lives there; a .zip of the built folder with index.html at its root for an app) on the dashboard, after switching the dashboard to the workspace you name: the drive (https://app.snaphost.ai/) for a new site, or the site's own page (https://app.snaphost.ai/sites/<site_id>, "Upload a new version") to keep an existing link; the site_id comes from list_sites or the publish result. Then list_sites shows it (a new site is the newest, named after the file) and every sharing tool applies. For a single binary file (a PDF, a
   .docx/.pptx/.xlsx, an image, a video, an audio), call
   `create_bundle_upload` with `filename` (its real name, extension included), send the
   file's own bytes against the `upload_id` it returns, and pass that `upload_id` so it is
   rendered as its kind. A Word, PowerPoint or Excel file is shown as its page alone: readers
   can download the file itself only when `include_original` is true, since it holds speaker
   notes, hidden sheets, formulas and comments the page does not show; ask the user before
   turning that on. A workbook you wrote with a library carries no computed values, so its
   formula cells show the formulas; the result says so in `render_notices`. Relay every
   render notice to the user, and offer to open and save the file in Excel or Numbers first
   when values matter. For a multi-file app with binaries (a built Vite/React site, an
   image-heavy page) call `create_bundle_upload` first, send the .zip against the `upload_id`
   it returns, then pass that `upload_id` instead (up to the 50 MB bundle cap). Two ways to send it:
   - `upload_bundle_part`, one numbered slice per call (split the .zip into slices of at most
     `tool_part_max_bytes`, base64 each on its own, send `part` 0, 1, 2, ... in order). These
     are tool calls on the same connection as every other SnapHost tool, so they work with no
     network of your own.
   - PUT the .zip to `upload_url` (`Content-Type: application/zip`; above `max_part_bytes`
     split it and PUT each piece to `upload_url/parts/0`, `/parts/1`, ... in order). Fewer,
     larger requests, when your shell can reach the SnapHost app host.
   Your tool calls and your shell do not share a network. If a PUT is refused (a 403 naming the
   host, a connection that never leaves), switch to `upload_bundle_part` and carry on: that
   route is unaffected, and nobody needs to change a network setting. Reach for `zipBase64`
   only for a multi-file bundle that fits the request body once encoded, as a last resort: base64 has to be emitted byte for byte,
   and a long string often is not, so it is checked and rejected when it does not verify.
5. Check the `Workspace:` line of the result. If the site landed in the wrong workspace, publish
   it again with the right `workspace_id` and `delete_site` the stray copy: a site cannot be
   moved between workspaces, and the stray copy is live until deleted.
6. Report the returned `share_url` to the user. Tell them the link is stable: re-running
   this skill with the same slug updates the same site in place, and anyone already viewing
   is offered a Refresh to the new version.

## Notes

- New sites default to private (allowlist) on Pro. Use `/snaphost:share` to open it
  up or add viewers.
- To change an existing site instead of creating a new one, use `/snaphost:update`.
- Building a report, a presentation or a dashboard rather than publishing one that exists? Use
  `/snaphost:report`, `/snaphost:presentation` or `/snaphost:dashboard`: they follow the
  workspace brand kit and SnapHost's design guide.
- A multi-file app gets client-side routing for free: clean routes like `/about` deep-link
  and survive a refresh. Build with `BrowserRouter` and a relative base (`base: './'`), and
  derive the router basename from the injected `<base>` tag (`document.querySelector('base')`).
