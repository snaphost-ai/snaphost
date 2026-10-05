---
name: dashboard
description: "Build an on-brand dashboard that keeps itself current and refreshes open screens on its own, for a link or an office screen."
---

Use this skill to build a dashboard the person can send as a link: on their brand, readable on a
phone and a desktop, and good enough on the first pass that they only check it, never redesign it.
The source can be anything: a PDF, a Word or PowerPoint file, a spreadsheet, figures pasted into
the chat, or an existing page.

## Steps

1. **Read the source first.** Work out what it is, who it is for, its language, its period and
   its key numbers before asking anything. When it is a summary (a press release, a news item,
   a slide someone forwarded), find the document it summarises and read that: the comparisons
   that give a number meaning (the budget, last year, the limit) live only in the primary
   document.
2. **Brief, in one message.** Call `get_design_guide` with `kind: "dashboard"` (no traits
   yet) and ask only the brief questions the source did not answer, all at once, each with the
   default you will use. Accept "you choose". A dashboard is `live`; add `kiosk` when it runs on an office screen.
3. **Workspace.** Unless one was chosen this session, call `list_workspaces`; with more than one
   workspace or a `note`, run `/snaphost:workspace` and pass its `workspace_id` on every call.
4. **Brand.** Call `get_brand_kit`. With a kit, paste its `tokens_css` first in the page's
   style block and its `fontLinkHtml` (when present) into `<head>`, use its logo URL, its
   fonts, its locale and its tone. With none, offer
   `/snaphost:brand` once ("I can set up your brand from your guideline or website so every
   page matches"); if they decline, use the neutral palette in the guide and carry on.
5. **Guide.** Call `get_design_guide` again with the traits the brief settled on, and read all
   of it. It is the bar.
6. **Build** from `get_page_recipe "dashboard-shell"` and the parts in `list_page_recipes`. Replace
   every [bracketed] placeholder. Format every number and date for the kit locale.
   Pick the guide's republish model (a scheduled routine rebuilds the page) unless the numbers
   move within the hour on a public page, in which case use the data model (a routine writes one
   key-value item per tile with `set_site_data`, and the page polls them). A private dashboard
   always uses the republish model. Say which you chose and set it up: after publishing, call
   `set_render_options` with `auto_refresh: true` so open screens reload on their own, and
   for the republish model offer to schedule the routine that updates it.
7. **Review loop, before publishing.** Score the page against every rubric item in the guide, in
   writing, as pass or fail, at 375, 768 and 1280 pixels wide (and in print preview when
   printable). Fix every failure, then score again. If you can render the page (a browser or
   screenshot tool), look at it at each width; otherwise read the CSS for each width.
   Do not publish with a failing item. A slide or chapter that is a headline, one number and a
   sentence fails however clean its CSS is.
8. **Publish** with `publish_site` (or `update_site` for an existing site), passing
   `document_kind: "dashboard"` and `printable` when it must print. Use a stable slug.
9. **Fix the warnings.** Fix every `brand_warnings` and `design_warnings` entry with
   `update_site` until both are empty or every remaining one is a declared, deliberate exception.
10. **Hand over.** Give the `share_url` and ask for exactly one round: "Open it on your phone and
    on your desktop and tell me anything that looks off." Mention `/snaphost:share` for who can
    open it.

## Notes

- The guide and the rubric are served by SnapHost and may change; always fetch them, never work
  from memory.
- Keep the person's words and numbers. Restructure and design, never invent data.
- Never paste a secret or token into the page.
