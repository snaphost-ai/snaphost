---
name: data
description: "Add a data feature (contact form, waitlist, RSVP, guestbook, counter, leaderboard) to a SnapHost site so the published page can save and read data."
---

Use this skill to give a SnapHost site a backend: the published page persists and reads data
through the injected `window.SNAPHOST` SDK, with no server to run. SnapHost Data needs
Pro; a `plan_limit` error means the owner needs to upgrade.

## Adding a ready-made feature (the fast path)

1. Identify the site (a site id, or resolve by title with `list_sites`).
2. List the templates with `list_data_recipes` (contact form, waitlist/email signup, RSVP,
   guestbook, shared counter, clock-in, leaderboard). Pick the one matching what the user asked for.
3. Call `apply_data_template` with the site id and recipe id. It provisions the collection (its
   access mode and schema) and **returns an HTML snippet**.
4. Add that HTML to the user's page: fetch the current document with `get_site_content`, merge the
   snippet into the right place (do not regenerate the page from scratch), then `update_site`.
   Confirm the change is live.

## Building something custom

The injected SDK is global and needs no keys; every call returns a promise.

- **Append**: `await window.SNAPHOST.append('<collection>', { ...yourFields })`. The collection is
  created on first write; never create it first. Records are immutable and append-only; an "edit" is
  a new event carrying the same `key` field.
- **Read**: `const { records } = await window.SNAPHOST.query('<collection>', { order: 'desc', limit: 50 })`.
  Each record is `{ id, value, created, actor }`: read the owner's fields from `record.value` and
  the timestamp from `record.created`. Query options: `order` ('asc' | 'desc'), `limit`,
  `where: [{ field, value }]` (equality filters), and `latest: true` to fold to the newest record
  per `key` (current state instead of full history).
- **Who is viewing**: `const me = await window.SNAPHOST.viewer()` resolves (after `ready`) to the
  current viewer's identity. A signed-in viewer of a private / allowlist site gives
  `{ id: 'user@example.com', kind: 'viewer' }`; a public site or a password-only viewer gives
  `{ kind: 'anonymous' }` (no `id`). `viewer` shows up in `Object.keys(window.SNAPHOST)`.
- **Read config (key-value)**: `await window.SNAPHOST.get('<namespace>', '<key>')` and
  `await window.SNAPHOST.list('<namespace>')` read owner-set values (set via `set_site_data`).
- **Render safely**: write values with `textContent`, never `innerHTML`.
- **Access mode** (`set_collection_access`): `public_read` (anyone can read: a guestbook or
  counter) or `append_only` (visitors write but only the owner reads: a contact form or
  waitlist; submissions are never readable by other visitors). You can also pause writes.
  Publish/update scans the page's code and pre-configures new collections to match it (one the
  page `query()`s becomes `public_read`, one it only `append()`s to stays `append_only`) and
  reports what it did in the tool result; an owner's explicit setting is never overridden.
- **Schema** (`define_collection_schema`): an optional JSON Schema validates every appended record
  server-side. Set it for the user when their data has a fixed shape.
- **Inspect / moderate**: `list_site_collections`, `query_site_records`, `export_site_data`,
  `delete_site_record`. Attribution on each record is server-stamped from the verified viewer on a
  private site, never from the request body. Note: collections are per-site, so a page only sees its
  own site's collections.

## Attributing a record to a person ("who did this")

For any feature that shows who did something (leaderboard, high scores, guestbook, comments,
clock-in, RSVP): capture the viewer with `window.SNAPHOST.viewer()` and **write the identity into
the record's `value`** (a `who` field). `actor` is server-stamped and visible to the owner in the
dashboard and exports, but it is **redacted in-page** (only `kind`, never the id), so never read
`actor` back to display who wrote a record, and never fall back to `window.prompt()` for a name.

**Aggregation rule** for a per-person board: records that HAVE an identity fold to that person's
best (or latest) row; records with NO identity are anonymous and must each stay their own row,
never merged into a single "Anonymous". Worked snippet (capture -> stamp -> aggregate, top N):

```html
<script>
  let me = { kind: 'anonymous' }
  window.SNAPHOST.viewer().then(v => { me = v })

  async function submitScore(score) {
    const entry = { score }
    if (me.kind === 'viewer' && me.id) entry.who = me.id // stamp identity into the value
    await window.SNAPHOST.append('leaderboard', entry)
    render()
  }

  async function render() {
    const { records } = await window.SNAPHOST.query('leaderboard', { order: 'desc', limit: 500 })
    const best = new Map(), anonymous = []
    for (const r of records) {
      const who = (r.value.who || '').trim()          // never read r.actor here (redacted in-page)
      const score = Number(r.value.score) || 0
      if (!who) { anonymous.push({ label: 'Anonymous', score }); continue }
      const key = who.toLowerCase(), prior = best.get(key)
      if (!prior || score > prior.score) best.set(key, { label: who, score })
    }
    const rows = [...best.values(), ...anonymous].sort((a, b) => b.score - a.score).slice(0, 10)
    const list = document.getElementById('leaderboard')
    list.replaceChildren()
    for (const row of rows) {
      const li = document.createElement('li')
      li.textContent = row.label + ': ' + row.score // textContent, never innerHTML
      list.appendChild(li)
    }
  }
</script>
```

## Notes

- Recommend the ready-made templates first; they are the simplest path for a non-technical owner.
- Always re-publish (`update_site`) after adding a snippet, or the feature will not be live.
