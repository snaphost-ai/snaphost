---
name: brand
description: "Set up or update the workspace's brand kit (colours, fonts, logo, locale, tone) from a brand guideline or website, so every page your AI builds is on brand."
---

Use this skill to give a SnapHost workspace its brand kit, once, so every page, report, deck and
dashboard built in it starts on brand: the palette, the fonts, the logo, the language and locale,
and how the brand writes. The person brings the brand; you do the extracting.

## Steps

1. **Workspace.** Call `list_workspaces` and, with more than one, run `/snaphost:workspace`. The
   kit belongs to a team workspace on a paid plan, and only an owner or admin may save it.
2. **What exists.** Call `get_brand_kit`. If a kit exists, show it and ask what should change.
3. **Find the brand.** Ask for whichever they have, best first: a brand guideline (PDF), their
   website, or a screenshot of a page they consider on brand. Read it and extract:
   - colours for every role: primary, primaryStrong (a darker primary for hover and links),
     secondary, accent, ink (body text), muted (captions), line (hairlines), surface (the page),
     surfaceTint (a faint panel), onPrimary and onSecondary (text on those colours), all #rrggbb;
   - the heading and body fonts, and whether they are Google Fonts or system fonts (a licensed
     font that is neither becomes its nearest Google Fonts match, said out loud);
   - the logo and a small mark or emblem: an https URL of a PNG, SVG or WebP file on their site
     (open the site's HTML to find the real file, not a page that shows it), or, when the logo is
     drawn inline in the page's HTML, the `<svg>` markup itself for `logo_svg`. A logo that only
     exists as a JPEG or a photo is uploaded on the workspace's Brand page instead; say so and
     save the kit once that is done;
   - the language and locale (`is`, `is-IS`), a few sentences of tone, and any words to avoid.
4. **Confirm.** Show the proposed kit as a table with each colour's hex and role, the fonts, the
   logo URL and the locale. Point out anything you guessed. Wait for a yes.
5. **Save** with `set_brand_kit`. Relay every warning (low contrast is the usual one) and offer a
   corrected colour.
6. **Use it.** Offer to rebuild one existing page with the kit, or to start a report with
   `/snaphost:report`.

## Notes

- The logo is fetched and hosted by SnapHost; pages use the kit's own logo URL from
  `get_brand_kit`, never the source URL.
- A kit with Google fonts hands pages `fontLinkHtml` to paste into `<head>`.
- A kit with Google fonts has those fonts allowed on every page it publishes.
- The kit can also be edited on the workspace's Brand page in the dashboard.
