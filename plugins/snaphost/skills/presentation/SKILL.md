---
name: presentation
description: "Build an on-brand slide presentation as a live link: full screen, keyboard and swipe, and a PDF handout."
---

Use this skill to build a slide presentation the person can send as a link: on their brand, readable on a
phone and a desktop, and good enough on the first pass that they only check it, never redesign it.
The source can be anything: a PDF, a Word or PowerPoint file, a spreadsheet, figures pasted into
the chat, or an existing page.

## Steps

1. **Read the source first.** Work out what it is, who it is for, its language, its period and
   its key numbers before asking anything. When it is a summary (a press release, a news item,
   a slide someone forwarded), find the document it summarises and read that: the comparisons
   that give a number meaning (the budget, last year, the limit) live only in the primary
   document.
2. **Brief, in one message.** Call `get_design_guide` with `kind: "presentation"` (no traits
   yet) and ask only the brief questions the source did not answer, all at once, each with the
   default you will use. Accept "you choose". Add `printable` when they want a handout; the deck then prints one slide per landscape page.
3. **Workspace.** Unless one was chosen this session, call `list_workspaces`; with more than one
   workspace or a `note`, run `/snaphost:workspace` and pass its `workspace_id` on every call.
4. **Brand.** Call `get_brand_kit`. With a kit, paste its `tokens_css` first in the page's
   style block and its `fontLinkHtml` (when present) into `<head>`, use its logo URL, its
   fonts, its locale and its tone. With none, offer
   `/snaphost:brand` once ("I can set up your brand from your guideline or website so every
   page matches"); if they decline, use the neutral palette in the guide and carry on.
5. **Guide.** Call `get_design_guide` again with the traits the brief settled on, and read all
   of it. It is the bar.
6. **Build** from `get_page_recipe "presentation-shell"` and the parts in `list_page_recipes`. Replace
   every [bracketed] placeholder. Format every number and date for the kit locale.
   Eight slides is the shape of a deck of numbers: a cover with one visual and the sentence to
   remember; the year in four key figures with change against budget and last year; plan to
   outcome (the `waterfall-chart` recipe, with a toggle that takes the one-off out); where the
   money comes from (`donut-chart`); where it goes; the trend over years against any limit
   (`line-chart`); risks and the auditor's opinion (the shell's accordion beside its card); a
   summary of four numbers. Reach a shorter deck by merging views, never by dropping the
   comparisons. Every data slide has a sentence headline, a lede, one visual, its comparison and
   a source line, in two columns on desktop, and one control where it adds understanding (a view
   toggle, a one-off toggle, hover values), each a button with `aria-pressed`. Choose each
   slide's layout from the shell's vocabulary by its content shape (grid, split, chart, compare,
   statement, quote, list); never give two consecutive data slides the same layout, and open with
   the cover treatment the kit's `style.cover` names. The subject decides the shape: the guide
   names five. Speaker notes go in the hidden notes aside. A headline, a lone number and a
   sentence is not a slide.
7. **Review loop, before publishing.** Score the page against every rubric item in the guide, in
   writing, as pass or fail, at 375, 768 and 1280 pixels wide (and in print preview when
   printable). Fix every failure, then score again. If you can render the page (a browser or
   screenshot tool), look at it at each width; otherwise read the CSS for each width.
   Do not publish with a failing item. A slide or chapter that is a headline, one number and a
   sentence fails however clean its CSS is.
8. **Publish** with `publish_site` (or `update_site` for an existing site), passing
   `document_kind: "presentation"` and `printable` when it must print. Use a stable slug.
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
