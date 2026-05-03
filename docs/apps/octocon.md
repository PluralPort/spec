# Octocon

## Source Status

Sources:

- Backend repository: <https://github.com/OctoconDev/octocon> (Elixir / Phoenix / ScyllaDB monolith with REST API and Discord bot)
- App repository: <https://github.com/OctoconDev/app>
- Public docs: <https://octocon.app/docs>
- Contributing docs: <https://octocon.app/docs/contributing/getting-started>
- App import docs: <https://octocon.app/docs/app/importing-data>

Both `OctoconDev/octocon` and `OctoconDev/app` are now public; the contributing docs list them alongside `OctoconDev/website` without any "private" caveat. This research is based on the backend source — primarily the Ecto schemas under `lib/octocon/` and the Discord-bot export and Simply Plural import workers — supplemented by the public app docs.

Octocon has announced its own discontinuation in 2026, shortly after Simply Plural's. That context is preserved elsewhere in these notes; this document describes the shape as it stands while the app is still operational, since the realistic audience is people getting their data out before shutdown.

## Storage And API Shape

Octocon is a distributed Elixir monolith. The backend is split into:

- `octocon` — core app, business logic, ScyllaDB-backed data layer.
- `octocon-web` — Phoenix REST API, metrics, admin dashboard.
- `octocon-discord` — Discord bot (alternative interface, proxying).

Schemas use Ecto with `Exandra` for ScyllaDB. Most records are keyed by a 7-character lowercase user ID (`^[a-z]{7}$`) plus a per-record ID (UUID for tags/journals/polls/fronts, small integer up to 32,767 for alters).

A user-facing export exists today only through the Discord bot: `lib/octocon_discord/commands/export.ex` exposes a slash command that calls `Octocon.Accounts.gather_export_data/1` and then either `format_pk_export/1` (PluralKit-compatible v2 datafile) or `format_full_export/1` (Octocon-native JSON, currently not consumed by any other platform). The "full" variant is the closest thing to a portable Octocon shape and is what an OpenPlural importer would target.

## Records

### User / system

`Octocon.Accounts.User` (`lib/octocon/accounts/user.ex`):

- `id` — 7-char lowercase string, primary key.
- `email`, `discord_id`, `apple_id`, `google_id` — at least one is required.
- `username`, `avatar_url`, `description`.
- `lifetime_alter_count` — used to mint new alter IDs.
- `primary_front` — integer alter ID or nil.
- `discord_settings` — embedded (autoproxy mode `:off | :front | :latch`, system tag, case-insensitive proxy flag, pronouns-on-proxy flag).
- `fields` — embedded list of `Octocon.Accounts.Field` (custom-field definitions).
- `salt`, `encryption_initialized`, `encryption_key_checksum` — for client-side encrypted journal content.

The "full" export passes through `username`, `description`, `id`, `avatar_url`, and `fields` (definitions only).

### Alter (member)

`Octocon.Alters.Alter` (`lib/octocon/alters/alter.ex`):

- `user_id`, `id` (integer ≤ 32,767).
- `name` (≤ 80), `pronouns` (≤ 50), `description` (≤ 3,000).
- `alias` — Discord-facing alias.
- `security_level` — enum `:public | :friends_only | :trusted_only | :private` (default `:private`).
- `avatar_url`, `extra_images` (array, max 3, "currently unused"), `color` (hex `#rrggbb`).
- `untracked` (boolean) — used for Simply Plural custom fronts on import.
- `archived` (boolean), `pinned` (boolean), `last_fronted` (UTC datetime).
- `discord_proxies` — array of strings in the form `prefixtextsuffix` (literal `text` separator).
- `proxy_name` — display name used for Discord proxying.
- `fields` — embedded list of `Octocon.Alters.Field` records (`id` references a `Octocon.Accounts.Field` definition; `value` is a string).
- `inserted_at`, `updated_at`.

The "full" export emits `id`, `name`, `pronouns`, `description`, `color`, `avatar_url`, `proxy_name`, `discord_proxies`, and `fields` (each value as `{id, value}`). Notably it drops `security_level`, `untracked`, `archived`, `pinned`, `alias`, `extra_images`, and `last_fronted`. The PK export maps `discord_proxies` into PluralKit `proxy_tags` (split on the literal `text`).

### Fronts

`Octocon.Fronts.Front` (`lib/octocon/fronts/front.ex`):

- `user_id`, `id` (UUID), `alter_id` (integer), `comment` (≤ 50), `time_start`, `time_end`.
- One row per alter per fronting interval. `time_end` is nil while currently fronting.
- Several denormalized companion tables exist for ScyllaDB read paths: `current_fronts`, `fronts_by_alter`, `fronts_by_time`, `fronts_by_end_time`. They all reduce to the same logical `Front` shape.

Co-fronting is represented as concurrent rows for different alters, not via a join table — closer to Simply Plural's per-member intervals than to Sheaf's grouped front. `comment` is used by the Simply Plural importer to carry SP's `customStatus`.

### Tags (hierarchical, used as groups)

`Octocon.Tags.Tag` (`lib/octocon/tags/tag.ex`):

- `user_id`, `id` (UUID).
- `name` (≤ 100), `description` (≤ 1,000), `color` (hex).
- `security_level` — same enum as alters.
- `parent_tag_id` (UUID, nullable) — supports hierarchy.

`Octocon.Tags.AlterTag` (`lib/octocon/tags/alter_tag.ex`) is the join table (`user_id`, `tag_id`, `alter_id`).

Octocon doesn't have a separate "groups" record. The Simply Plural import worker (`lib/octocon/workers/simply_plural_import_worker.ex`) maps SP groups onto tags, preserving hierarchy via SP's `parent` field (`"root"` becomes nil). The PK exporter emits Octocon tags as PluralKit `groups`.

### Custom fields

Two-part shape:

- Definitions live on the user: `Octocon.Accounts.Field` (`lib/octocon/accounts/field.ex`) with `id` (UUID), `name` (≤ 100), `type` enum `:text | :number | :boolean`, `locked` flag, `security_level`.
- Values live on each alter: `Octocon.Alters.Field` (`lib/octocon/alters/field.ex`) — `id` (UUID matching the definition) and `value` (string).

Values are stringly typed regardless of definition `type`; the type is treated as a hint for clients. Simply Plural import creates SP-linked definitions as `type: :text` with `security_level: :private`.

### Journals

Two journal records, both currently absent from the public export pipeline:

- `Octocon.Journals.GlobalJournalEntry` (`lib/octocon/journals/global_journal_entry.ex`) — system-wide. `title` (≤ 100), `content` (validated to ≤ 50,000 in the changeset, despite the moduledoc claim of 20,000), `color`, `pinned`, `locked`. Linked to alters via `Octocon.Journals.GlobalJournalAlters` (join table; `alters` is a virtual array).
- `Octocon.Journals.AlterJournalEntry` (`lib/octocon/journals/alter_journal_entry.ex`) — per-alter, same shape minus the alters join.

User-level encryption fields on `Accounts.User` (`encryption_initialized`, `encryption_key_checksum`, `salt`) suggest journal content can be client-side encrypted at rest, though the encryption flow itself wasn't surveyed in detail.

### Polls

`Octocon.Polls.Poll` (`lib/octocon/polls/poll.ex`):

- `user_id`, `id` (UUID), `title` (≤ 100), `description` (≤ 2,000).
- `type` enum `:vote | :choice`.
- `data` — opaque map (poll-options/results structure not surveyed).
- `time_end`.

The full export passes polls through verbatim. The internal shape of `data` isn't surveyed.

### Friendships and sharing

- `Octocon.Friendships.Friendship` (`lib/octocon/friendships/friendship.ex`) — bidirectional, with `level` enum `:friend | :trusted_friend` and a `since` timestamp.
- `Octocon.Friendships.Request` — pending requests with `from_id`, `to_id`, `date_sent`.

The friendship `level` lines up with the alter/tag `security_level` enum: a `:friends_only` alter is visible to anyone with a `:friend`-level friendship; `:trusted_only` requires `:trusted_friend`.

### Discord proxying

Lives directly on the alter (`discord_proxies`, `proxy_name`) plus user-level `discord_settings` (autoproxy mode, system tag, options). No separate proxied-message archive in the export pipeline.

### Messages, polls subscribers, server settings, etc.

`lib/octocon/messages.ex`, `lib/octocon/server_settings/`, `lib/octocon/channel_blacklists/` and similar exist but are out of scope for OpenPlural's portable model and shape not surveyed in detail.

## Export Shape

`Octocon.Accounts.format_full_export/1` returns:

```json
{
  "user": { "username": "...", "description": "...", "id": "...", "avatar_url": "...", "fields": [{"id": "...", "name": "...", "type": "text", "locked": false, "security_level": "private"}] },
  "alters": [ { "id": 1, "name": "...", "pronouns": "...", "description": "...", "color": "#...", "avatar_url": "...", "proxy_name": "...", "discord_proxies": [...], "fields": [{"id": "...", "value": "..."}] } ],
  "fronts": [ { "id": "...", "alter_id": 1, "comment": "", "time_start": "...", "time_end": "..." } ],
  "tags":   [ { "id": "...", "name": "...", "description": "...", "color": "#...", "security_level": "private", "parent_tag_id": null, "alters": [1, 2], "inserted_at": "...", "updated_at": "..." } ],
  "polls":  [ { "id": "...", "title": "...", "description": "...", "type": "vote", "data": {...}, "time_end": "...", "inserted_at": "...", "updated_at": "..." } ]
}
```

Gaps versus the on-disk model:

- No journals (global or alter).
- No friendships, friend requests, or shared-with metadata.
- No `discord_settings` block; only per-alter proxy strings.
- Alters drop `security_level`, `untracked`, `archived`, `pinned`, `alias`, `last_fronted`, `extra_images`, timestamps.
- No system-tag / autoproxy info.

The PK export (`format_pk_export/1`) is a PluralKit datafile v2: `name`, `description`, `avatar_url`, `members[]` (with `proxy_tags` parsed from `discord_proxies`), `groups[]` (built from tags, indexed by position), `switches: []` (always empty — switch history isn't ported). It exists for one-way migration to PluralKit, not as an OpenPlural source.

## Imports

Two import workers in `lib/octocon/workers/`:

### Simply Plural — `simply_plural_import_worker.ex`

Hits the Apparyllis API (`https://api.apparyllis.com/v1/`). Imports:

- `/me` system data → user `description`.
- `/customFields/{id}` → `Octocon.Accounts.Field` definitions, all created as `type: :text`, `security_level: :private`. SP custom-field IDs are mapped to fresh Octocon UUIDs.
- `/members/{id}` → alters with `untracked: false`.
- `/customFronts/{id}` → also alters, but flagged `untracked: true`. (This is how Octocon represents SP custom fronts.)
- `members[].info` map → `Octocon.Alters.Field` values keyed by the per-system field-association map.
- `/groups/{id}` → `Octocon.Tags.Tag` records; `parent` becomes `parent_tag_id` (with `"root"` flattened to nil), so SP's hierarchical groups become hierarchical Octocon tags.
- `/frontHistory/{id}` (chunked over 6-month windows from epoch 2015-01-01) → `Octocon.Fronts.Front` rows. SP's `customStatus` maps to `Front.comment` (truncated to 50 chars).
- `/fronters/` → currently-open `Front` rows (with `time_end: nil`) plus a `CurrentFront` companion entry.
- Avatars are pulled from `https://spaces.apparyllis.com/avatars/{uid}/{avatarUuid}` and re-stored on Octocon's CDN.

All imported alters and tags are created at `security_level: :private`.

### PluralKit — `plural_kit_import_worker.ex`

Per the public app docs and the worker filename, imports members and groups only. Worker internals not surveyed in detail.

## Import/Interoperability Notes

For OpenPlural's purposes:

- Octocon's alter shape is recognizable: name/pronouns/description/avatar/color plus typed custom-field values. The `untracked` flag is a first-class concept for "this is a non-alter front identity (e.g. SP custom front)" and maps cleanly to a Member with that flag in extensions, similar to Simply Plural's `is_custom_front`.
- Fronting is per-member intervals with a short comment, not grouped periods. An OpenPlural converter should emit one `FrontPeriod` + one `FrontAssignment` per Octocon `Front` row, with `comment` carried on the assignment.
- "Tags" play the role of both groups and tags: hierarchical, colored, named, with their own `security_level`. Mapping them onto `Group` (with `parent_group_id`) is closer to Octocon's actual usage than mapping onto a flat tag taxonomy. The PK export already calls them groups.
- Custom-field definitions live on the user, not on a separate definitions table; values live inline on alters. The type space (`text | number | boolean`) is narrower than Sheaf's or Plural Star's, and values are always stringly typed at rest.
- `security_level` on alters/tags pairs with friendship `level` to drive sharing visibility. Mapping the four-level enum into OpenPlural's privacy fragment can stay lossless via `extensions.octocon.security_level` and `extensions.octocon.friendship_level`.
- Journals exist in the schema but are not in the official export. An OpenPlural importer that reads Octocon data directly (rather than via the `/export` slash command) could include them; one that reads the export file cannot.
- Polls, Discord proxy data, and friendship metadata should stay in optional modules or extensions.
- The PK-format export drops switch history entirely. People migrating from Octocon to PluralKit through this path lose all front history — worth flagging in adoption docs if anyone writes an Octocon → OpenPlural converter that funnels through PK.
