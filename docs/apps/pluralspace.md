# PluralSpace

## Source Status

Sources:

- Website: https://pluralspace.app/
- Two sample GDPR exports inspected, generated 2026-05-03 (mock data). The second was produced after creating groups in the app to capture group records.
- Maintainer implementation notes shared 2026-05-06 (stack/runtime and fronting-storage overview). These are useful architectural context, but they are not a published schema or source audit.

PluralSpace is a separate app from Plural Star, despite a shared naming history: the React Native app formerly called "Plural Space" rebranded to "Plural Star", while the unrelated web app at `pluralspace.app` kept the "PluralSpace" (no space) name. They are different products with different data models and should be treated as distinct apps for OpenPlural purposes. See [`plural-star.md`](plural-star.md) for the Plural Star format.

The current export is framed as a regulatory data-portability dump: the manifest cites GDPR Article 15 (Right of Access) and Article 20 (Right to Data Portability). This isn't the same affordance as a round-trippable backup. The ZIP and `data.json` shape documented below is reconstructed from inspected sample exports — there's no published schema for it.

The public [developers page](https://pluralspace.app/developers) lists a REST API as "Coming Soon" and "actively in development" and shows a preview, but the API isn't a usable surface yet. If and when it ships it'd likely be a better target for an OpenPlural converter than the GDPR export, but that's speculative until it's public. The research below should be read as a snapshot of the GDPR export specifically.

## Runtime And Deployment Context

Per maintainer notes shared 2026-05-06, PluralSpace's current stack is:

- Backend/API: PHP with Laravel.
- Web frontend: currently Laravel Livewire + Tailwind CSS + FluxUI, with an in-progress rewrite toward Vue.js.
- Mobile apps: CapacitorJS with Vue.js, reusing website components where possible.
- Database/search/cache: PostgreSQL, Redis, Memcache, Meilisearch.
- Queues/realtime/runtime: Laravel queues with Horizon, Soketi for Pusher-compatible websockets, FrankenPHP as the long-lived PHP runner.
- Ops/services: self-hosted deployment; Gatus for status, GlitchTip for error monitoring, telemetry disabled on self-hosted OSS components.
- Topology: 2 app nodes behind a least-connections load balancer, 2 worker nodes running Horizon, plus separate nodes for Meilisearch, Gatus, and GlitchTip. The help bot is written in SapphireJS and calls the PluralSpace API.

This doesn't change the export mapping directly, but it is useful context for future API/realtime documentation: the app is a Laravel/Postgres system with queue-backed notifications and websocket-delivered UI updates, not a local-only client.

## Export Shape

The export is a ZIP containing:

```
manifest.json
data.json
media/         # empty in inspected sample
```

`manifest.json`:

```json
{
  "export_date": "2026-05-03T13:23:40+00:00",
  "system_name": "Example System",
  "format_version": "1.0",
  "user_email": "user@example.com",
  "regulation_reference": "GDPR Article 15 (Right of Access), Article 20 (Right to Data Portability)"
}
```

`data.json` is a single object keyed by record category:

```json
{
  "system": {},
  "members": [],
  "fronts": [],
  "journal_entries": [],
  "chat_channels": [],
  "polls": [],
  "thoughts": [],
  "member_groups": [],
  "custom_fields": [],
  "media_files": []
}
```

Fields use snake_case throughout. IDs are numeric (likely DB primary keys), not UUIDs. Timestamps are ISO-8601 with explicit offsets (`+00:00`).

## Records

### System

```json
{
  "name": "Example System",
  "slug": "example-system",
  "description": null,
  "color": "#7c3aed",
  "visibility": "private",
  "created_at": "2026-05-03T00:25:27+00:00"
}
```

- `slug`: URL-safe handle, presumably for the public profile route.
- `visibility`: observed `"private"`. Other values not confirmed; likely `"public"` and possibly `"unlisted"` based on common patterns.
- No tag, no banner, no avatar field at the system level in the inspected export.

### Members

```json
{
  "id": 1523074,
  "name": "Evren",
  "display_name": null,
  "pronouns": "they/them, xe/xem",
  "description": "...",
  "color": "#818cf8",
  "role": ["Host", "Core"],
  "is_archived": false,
  "is_custom_front": false,
  "avatar_path": null,
  "avatar_media_path": null,
  "groups": ["younger"],
  "custom_field_values": [],
  "created_at": "2026-05-03T00:35:40+00:00"
}
```

Notable shape choices:

- `pronouns` is a single free-text string and accommodates multiple sets in one field (`"they/them, xe/xem"`).
- `role` is an **array of free-text strings** (e.g. `["Host", "Core"]`, `["Caretaker", "Protector"]`, `["Trauma Holder", "Teen"]`, `["Little", "Child Alter"]`). Not a controlled vocabulary. Some members have `role: null`.
- `is_custom_front` is a flag on the member record. No separate custom-fronts collection. The inspected exports contained zero members with `is_custom_front: true`, so the actual export shape of a custom front is unverified — `name`, `pronouns`, `description`, `color`, `avatar_*` are presumably reused; whether `role`/`groups` apply is unknown.
- `groups` is a **list of group names (strings)**, not group IDs. Group records also carry their own `members: [{id, name}]` list, so membership is denormalized in two places. Renaming a group requires updating both sides.
- `custom_field_values` is presumed to embed populated field values on the member (parallel structure to `groups`), but **no populated value was observed in either inspected export** despite four field definitions existing. Shape unverified.
- `avatar_path` and `avatar_media_path` are paths into the `media/` directory. Empty in the inspected samples.
- No birthday field. No banner field. No per-member privacy/visibility field.

### Fronts

```json
{
  "id": 10245325,
  "member_id": 1524250,
  "member_name": "Mira",
  "type": "front",
  "type_name": "Front",
  "started_at": "2026-05-03T01:23:38+00:00",
  "ended_at": "2026-05-03T01:28:45+00:00",
  "comment": "Evren and Mira are co-fronting this afternoon...",
  "is_live": false
}
```

- One row per member per period. **Co-fronting is represented by multiple rows sharing identical `started_at`/`ended_at`**, not by an array of members on one row.
- `member_name` is denormalized onto each row.
- `type`/`type_name` suggests a future taxonomy (e.g. background, co-conscious) but only `"front"` was observed.
- `comment` is duplicated across all co-front rows that share the interval.
- `is_live` flags an open front. `ended_at` is non-null even on archived rows.
- No tier distinction (primary vs. co-front vs. co-conscious) — flatter than Plural Star.

Maintainer-provided storage notes line up with that export shape:

- The relational core is described as `fronts` plus `systems`, `members`, and `front types`, with `fronts` keyed by `system_id`, `member_id`, `front_type_id`, `started_at`, and `ended_at`.
- That reinforces our current interpretation that co-fronting is reconstructed from concurrent per-member rows, not from a separate grouped front record.
- It also makes the export's `type` / `type_name` pair easier to interpret: those likely come from a joined front-type relation rather than a flat enum stored directly on the row.
- The maintainer also described a start/end/change pipeline that emits a `FrontChanged` event, flushes front-related cache via an observer, and queues user notifications. That's useful product/architecture context, but not part of the GDPR export contract documented here.

### Journal Entries

```json
{
  "id": 54210,
  "title": "Therapy was rough today",
  "content": "## Session notes...",
  "visibility_level": 5,
  "members": [{ "id": 1523074, "name": "Evren" }],
  "date": "2026-01-15T01:30:00+00:00",
  "created_at": "2026-05-03T00:57:49+00:00",
  "updated_at": "2026-05-03T00:57:49+00:00"
}
```

- `content` is markdown.
- `visibility_level` is numeric. Only `5` observed; meaning of the scale is undocumented in the export. Likely a privacy enum (system / private / friends / public bands).
- `members` is an array of `{id, name}` snapshots — likely "this entry is about/by these members" rather than an author field.
- `date` and `created_at` are separate: `date` is the entry's logical date (back-dated entries supported), `created_at` is the write time.
- No author user/member ID on the entry itself.

### Chat Channels

```json
{
  "id": 38577,
  "name": "general",
  "created_at": "2026-05-03T01:47:11+00:00",
  "messages": [
    {
      "id": 1705019,
      "member_name": "Alex",
      "content": "...",
      "created_at": "2026-05-03T02:17:38+00:00"
    }
  ]
}
```

- Channels embed their messages directly (no separate `messages` collection).
- Messages carry `member_name` only — **no `member_id` reference**, which is lossy if members are renamed.
- No reply, edit, reaction, or attachment fields observed.

### Polls

```json
{
  "id": 9638,
  "title": "What should we prioritize for system wellness this month?",
  "description": "...",
  "status": "open",
  "allows_multiple_votes": false,
  "created_by_member": { "id": 1523074, "name": "Evren" },
  "closes_at": null,
  "created_at": "2026-05-03T02:31:18+00:00",
  "options": [
    { "id": 39355, "text": "...", "vote_count": 0, "votes": [] }
  ]
}
```

- Polls are **system-level**, not per-member (unlike Plural Star's `MemberPoll` with `targetMemberId`).
- `votes` is an array embedded on each option. `vote_count` is denormalized.
- `created_by_member` is a `{id, name}` snapshot rather than a foreign key.

### Member Groups

```json
{
  "id": 716977,
  "name": "younger",
  "color": null,
  "description": null,
  "members": [
    { "id": 1523074, "name": "Evren" },
    { "id": 1547795, "name": "Alex" }
  ],
  "created_at": "2026-05-03T15:01:12+00:00"
}
```

- Group fields: `id`, `name`, `color`, `description`, `members[]`, `created_at`.
- `members[]` carries `{id, name}` snapshots — proper member IDs, unlike the inverse pointer.
- **No nesting field in the export** — no `parent_id`, `parent_group_id`, or similar — even though PluralSpace supports nested groups in the app UI. The Groups panel renders children indented under their parent and shows a "N subgroup(s)" badge, so the relationship is tracked server-side; it just doesn't appear in `data.json`.
- In the inspected export, "younger" had a "1 subgroup" badge in-app with "subgroup" as its child, but both serialized as flat siblings with no parent reference. An OpenPlural converter can't recover the tree from this file alone.
- Per-member `groups: ["younger"]` is the inverse pointer **using group names as strings**. Combined with the snapshot member list above, group membership is denormalized in two places, with one side using names and the other using IDs.

### Custom Fields

```json
{
  "id": 163615,
  "name": "Favorite Food",
  "field_type": "text",
  "is_multiple": false,
  "values": []
}
```

- `field_type`: observed `"text"` and `"date"`.
- `is_multiple` toggles single vs. multi-value semantics on a definition.
- `values` is nested inside the definition (Sheaf-style), not embedded on members.
- **No populated value was observed** in either inspected export, despite `members[].custom_field_values` existing as a parallel structure. The value record shape (whether values live in `custom_fields[].values` or `members[].custom_field_values` or both) is unverified.

### Thoughts And Media Files

Both collections were empty arrays in both inspected exports.

- `thoughts` isn't documented elsewhere; schema unknown. The name suggests a microblog-style feature.
- `media_files` is presumably the registry that `avatar_path`/`avatar_media_path` references resolve against, plus journal/chat attachments. The empty `media/` ZIP directory and empty `media_files[]` array are consistent in the inspected exports.

## Mapping To OpenPlural V0.1

### Clean mappings

| PluralSpace | OpenPlural |
| --- | --- |
| `system` | `systems[0]` (one system per export observed) |
| `members[]` | `members[]` |
| `fronts[]` (collapsed by `started_at`/`ended_at`) | `front_periods[]` with one assignment per matching row |
| `journal_entries[]` | `notes[]` |
| `chat_channels[].messages[]` | `chat` module (post-v0.1) |
| `polls[]` | `polls` module (post-v0.1) |
| `member_groups[]` | `groups[]` + `group_memberships[]` |
| `custom_fields[]` | `custom_fields[]` + `custom_field_values[]` |
| `media_files[]` + `avatar_path` | `assets[]` |

### Frictions

1. **Front collapsing.** PluralSpace stores one row per member with `started_at`/`ended_at`, so an importer must group rows by identical timestamps to build a `front_period` with multiple `assignments`. Confidence is high when timestamps match exactly, but if the exporter ever quantizes differently per row, group-by becomes lossy. Worth a `warnings[]` entry if collapse is heuristic.

2. **No front roles or tiers.** All assignments come back as `front_role: "member"`. The `type`/`type_name` field hints at a future taxonomy that should map to `extensions.pluralspace.front_type` until enumerated.

3. **`role` as array of free-text strings.** OpenPlural recommends taxonomy terms with `kind: "role"` rather than a privileged member field. Each entry in `role[]` becomes a `taxonomy_terms` record (deduped per system) plus a `taxonomy_assignments` row pointing at the member. Free-text means terms must be created lazily from the values seen.

4. **Group hierarchy is lost in the GDPR export.** PluralSpace supports nested groups in the app, but the GDPR export emits a flat `member_groups[]` with no `parent_id` field. An OpenPlural converter built against the GDPR export can't reconstruct the tree. The forthcoming API/export is the right path here, not a converter workaround — flagged with maintainers 2026-05-03.

5. **Group membership uses names, not IDs.** `members[].groups[]` stores group **names**. Group records carry their own `members[]` with member IDs. An OpenPlural converter should prefer the group → member side (which uses IDs) for `group_memberships[]`, and treat the member-side names as a denormalized cross-check. If the two disagree (e.g. mid-rename), the ID-side wins.

6. **Custom fronts not observed in either export.** `is_custom_front` is a member flag, but no member with `is_custom_front: true` appeared in either inspected export despite the user attempting to include some. Either the export filters them out, the flag is set elsewhere, or custom fronts live in a different surface entirely. The shape of an exported custom front is **unverified**.

7. **`visibility_level` numeric scale.** Without documentation of the band semantics, importers can't safely map it to OpenPlural's `visibility` enum. The numeric value should ride along in `extensions.pluralspace.visibility_level` and the `visibility` field should be set conservatively (e.g. `"private"`) until the scale is known.

8. **Chat messages reference `member_name` only.** OpenPlural's chat module (post-v0.1) will want a member ID. An importer can resolve names to IDs at import time, but renamed members or duplicates will collide. The original name should be preserved in `extensions.pluralspace.author_name`.

9. **`created_by_member` and journal `members` are snapshots, not foreign keys.** Resolution by name is fragile. Preserve both `id` and `name` from the snapshot via `source_refs` so future syncs can reconcile.

10. **System-level polls.** OpenPlural's polls module should accommodate both system-scoped and member-scoped polls (Plural Star has the latter). PluralSpace polls have no target member.

11. **GDPR framing ≠ round-trip backup.** The manifest names regulation articles, not a re-import contract. Producer info exists (`format_version: "1.0"`) but there's no statement of what the export omits — and we now have at least one confirmed omission (group hierarchy). An OpenPlural converter should emit `warnings[]` for `hierarchy_dropped`, plus any category it can't inspect (e.g. `thoughts`, empty `media_files` despite `avatar_path` references).

12. **No banners, no birthdays, no per-member privacy.** Standard OpenPlural fields are absent. No data loss on export *from* PluralSpace; on import *into* PluralSpace these fields would need to drop with a warning.

13. **`thoughts` is undocumented.** Until sampled, an OpenPlural converter has to either skip it with a warning or pass it through opaquely as `extensions.pluralspace.thoughts`.

### Recommendation

PluralSpace is well-served by the proposed v0.1 core plus the planned chat and polls modules. The mapping is mostly mechanical apart from front-row collapsing and the `role` array → taxonomy expansion. Open questions worth flagging:

- **Group hierarchy** isn't in the GDPR export. The forthcoming API is the right path; building a converter against the export today would bake in a loss that goes away on its own.
- **Identity-by-name** in chat messages, member→group pointers, `created_by_member`, and journal `members[]` is fragile. `source_refs` plus extension-preserved names mitigate it for converters built against the GDPR export; the API surface is likely to be more consistent.
- **Unverified shapes** for custom fronts, populated custom field values, and `thoughts` — these need either a fully-populated GDPR sample or the API documentation to pin down.

PluralSpace also reinforces an OpenPlural design choice independent of which surface a converter targets: documenting `format_version` and producer at the envelope level is necessary but not sufficient. A `capabilities.modules` declaration and a `warnings[]` block at export time would make the difference between a GDPR dump and a portability-grade export explicit to importers.
