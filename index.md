---
title: OpenPlural — a draft proposal for a shared plurality data shape
nav_active: home
footer: home
---

# OpenPlural
{: id="top" }

<p class="sub"><span class="draft">draft v0.1</span>A shared file shape for plurality apps. Nothing's fixed yet.</p>

If you maintain a plurality app — or you use one and care about getting your data out of it — we'd like your eyes on this.

## what we're trying to figure out

There are a lot of plurality apps. Each one stores roughly the same things — systems, members, fronting history, custom fields — but in pairwise-incompatible shapes. Moving from one app to another means writing a converter, or losing data, or both.

OpenPlural is a proposed file shape that any app can export to and import from. Not a new app and not a service — apps keep their internal models; they just agree on what an export looks like. The goal is to turn the export/import problem from "implement N pairwise converters" into "implement OpenPlural once."

## where it stands

Two app maintainers — [Prism](adopt.html#prism) and [Sheaf](adopt.html#sheaf) — have said they'd try this if the spec is good. We've [researched seven others](apps.html) to make sure the shape covers what real apps actually store. The spec is at draft v0.1 — meaning we'd rather argue with you about a wrong spec now than ship a confident one that's wrong later. Major changes within the current draft are tracked on the [changelog](changelog.html).

Things that are still open:

- Whether the [fronting model](spec-fronting.html) handles every app's edge cases.
- Whether [optional modules](spec-modules.html) like chat, polls, and habits belong in v1 or get cut.
- Whether the extension namespace mechanism is enough breathing room for app-specific data we haven't seen yet.

## what we're asking for

- **App maintainers** — read the [spec draft](spec.html) and tell us where it doesn't fit your data model. The [adoption guide](adopt.html) has the mapping tables for Prism and Sheaf as worked examples.
- **Users of plurality apps** — if you've migrated between apps, what did you lose? What was painful? Send a note.
- **Anyone** — the [per-app research](apps.html) covers nine apps. If we got something about your app wrong, please correct us.

## what's in the file

```
system
  members          pronouns, bios, proxy tags, custom fields
  groups           folders, subsystems, memberships
  fronting         periods, events, comments
  taxonomy         roles, tags, sources, relationships
  notes            member notes and journals
  assets           avatars, banners, media

optional modules   chat, polls, habits, reminders, sharing, safety
extensions         namespaced raw data — anything an app wants to preserve
```

Full field-by-field tables on the [records page](spec-records.html) and [fronting page](spec-fronting.html).

## why this exists at all

Apps come and go. Simply Plural announced discontinuation in March 2026. Octocon, which had positioned itself as a Simply Plural successor, announced its own shutdown shortly after. Users build years of records inside an app and then have to choose between staying on something unmaintained or losing their history.

A common file shape doesn't fix that on its own. But it makes "leave with your data" a normal operation instead of a project.

## reading order

- Short on time → just the [proposal draft](proposal.html).
- A little more time → [spec hub](spec.html), then [apps](apps.html), then [adopt](adopt.html).
- Want to see what's changed recently → [changelog](changelog.html).
- Want everything → the per-app research lives in [docs/](https://github.com/skylartaylor/openplural/tree/main/docs).

---
