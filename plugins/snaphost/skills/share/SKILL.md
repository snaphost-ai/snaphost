---
name: share
description: "Control who can view a SnapHost site: visibility, allowlist, password, and link expiry."
---

Use this skill to manage who can view a SnapHost site and how the link behaves.

## Steps

1. Identify the site (a site id, or resolve by title with `list_sites`).
2. Apply the change the user asked for:
   - **Public vs private**: `set_visibility` with `public` (anyone with the link) or
     `allowlist` (verified viewers only).
   - **Share with more people**: `add_to_allowlist` adds emails and/or domains *without*
     removing anyone already on the list; this is the default for "share with X". Check who is
     already on it first with `list_allowlist` (or `get_site`). Use `remove_from_allowlist`
     to take specific people off, and only reach for `set_allowlist` when the user explicitly
     wants to replace the whole list (it overwrites everyone; an empty list clears it).
   - **Password**: `set_view_password` to protect it, `clear_view_password` to remove it.
   - **Expiry**: `set_link_expiry` with an ISO 8601 timestamp, or null to clear.
3. For private sites, manage pending requests with `list_access_requests` and then
   `approve_access_request` / `deny_access_request`.

## Notes

- Private sharing, allowlists, passwords, and setting a link expiry are Pro
  features; if a call returns a `plan_limit` error, tell the user which plan unlocks it.
  Clearing a password or an expiry works on every plan.
- Visibility, allowlist, and password changes never change the share URL.
