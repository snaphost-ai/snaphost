---
name: workspace
description: "Pick which SnapHost workspace (the personal space or any workspace the person belongs to) to work in, then create or choose a site."
---

Use this skill to choose which SnapHost workspace you are working in, then create a new site or
pick an existing one to work on. The connection belongs to the person: it reaches their personal
space and every workspace they are a member of, so this is a choice of where to work, never of
what the connection can see.

## Steps

1. Call `list_workspaces`. The connection belongs to the person who authorized it and reaches
   their personal space plus every workspace they are a member of, whichever workspace was open
   when they connected. Each entry has a `workspace_id`, a `handle`, a `name`, a `kind`
   (`personal` or `team`), and your `role` (owner, admin, member, or viewer).
2. If there is more than one, show them to the user and ask which to work in. Remember the chosen
   `workspace_id` and pass it as `workspace_id` on every following tool call (`list_sites`,
   `publish_site`, `update_site`, `set_visibility`, and so on). If there is only one, use it
   without asking.
3. Call `list_sites` with the chosen `workspace_id` and show the user the site titles.
4. Ask what they want to do:
   - **Create a new site** → use `/snaphost:publish` (pass the same `workspace_id`).
   - **Work on an existing one** → confirm which title, then use `/snaphost:update` with its site
     id (and the same `workspace_id`), or `/snaphost:share` to change who can view it.
   - **Add a data feature** (contact form, waitlist, RSVP, guestbook, counter, clock-in, leaderboard) → use
     `/snaphost:data` to give a site a backend it can save and read data from.

## Notes

- Always carry the chosen `workspace_id` through the rest of the session so work lands in the
  right workspace. Omit it and a read acts in the person's personal space; a write without it is
  refused when the connection reaches more than one workspace.
- You can only act in workspaces `list_workspaces` returns; a `forbidden` error for another id
  means the person is not a member of it. A workspace the user expects but does not see is a
  membership question, not a connection one: they need to be a member of it in SnapHost.
- If `list_workspaces` returns a `note`, the connection is an older, single-workspace one. Relay
  the note: reconnecting SnapHost once gives a connection that covers every workspace.
- Re-run this skill any time to switch workspaces.
