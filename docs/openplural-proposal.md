---
title: "OpenPlural v0.1 Proposal"
nav_active: proposal
permalink: /proposal.html
---

# OpenPlural v0.1 Proposal

## Goals

OpenPlural is a JSON file shape for moving plurality data between apps without losing what matters:

- App developers implement one exporter and one importer.
- Users keep meaningful data when moving between apps.
- Apps with smaller feature sets can import a subset while reporting losses clearly.
- Source IDs and raw app-specific data can be preserved so future converters can recover detail.

This proposal is a starting point, not a finished spec.

## Design Principles

1. **JSON is the interchange.** Every researched app already produces or accepts it; nothing else clears that bar in v0.1.
2. **Core records stay boring.** Systems, members, groups, taxonomy labels, custom fields, front history, notes, and assets — the things every app actually has. Anything fancier becomes a module.
3. **Modules are optional.** Chat, polls, habits, reminders, friend sharing, safety plans, and proxy settings — a missing one of these doesn't block core portability, so they live outside the required shape.
4. **`source_refs` preserves identity.** Original app IDs travel with the record so users (and future sync tools) can reconcile after a round-trip.
5. **`extensions` preserves the rest.** Source-specific fields go into namespaced extensions. Other apps can ignore them; the data doesn't disappear.
6. **Timestamps are UTC ISO-8601.** Source timezone metadata is preserved as an extension when an app has it.
7. **Files/assets are separate records.** Keeps profile records small enough to read and lets binary content be deduplicated, replaced, or omitted independently.
8. **Warnings are part of the file.** Skipped, degraded, or repaired data is documented at export time so importers don't have to guess what's missing.

## Envelope

```json
{
  "openplural_version": "0.1",
  "exported_at": "2026-04-29T18:00:00Z",
  "producer": {
    "app": "Example App",
    "app_version": "1.2.3",
    "exporter_version": "0.1.0"
  },
  "capabilities": {
    "modules": [
      "systems",
      "members",
      "front_periods",
      "groups",
      "taxonomy",
      "custom_fields",
      "notes",
      "assets"
    ]
  },
  "systems": [],
  "members": [],
  "groups": [],
  "group_memberships": [],
  "taxonomy_terms": [],
  "taxonomy_assignments": [],
  "custom_fields": [],
  "custom_field_values": [],
  "front_periods": [],
  "front_events": [],
  "front_comments": [],
  "notes": [],
  "assets": [],
  "extensions": {},
  "warnings": []
}
```

## IDs And Source References

Every portable record should have a file-local `id`. Prefer UUIDv7/ULID/UUIDv4, but importers must not assume a specific ID algorithm.

Every converted record may include `source_refs`:

```json
{
  "id": "mem_01HV...",
  "source_refs": [
    {
      "app": "simply_plural",
      "collection": "members",
      "id": "abc123"
    },
    {
      "app": "pluralkit",
      "collection": "members",
      "id": "abcde",
      "uuid": "..."
    }
  ]
}
```

Source refs are essential for round-tripping and sync-aware imports.

## Systems

```json
{
  "id": "sys_01HV...",
  "name": "Example System",
  "display_name": null,
  "description": "Markdown or plain text",
  "tag": "| ex",
  "color": "#66ccff",
  "avatar_asset_id": "asset_avatar",
  "banner_asset_id": "asset_banner",
  "parent_system_id": null,
  "archived": false,
  "privacy": {
    "visibility": "private",
    "source": {}
  },
  "settings": {},
  "source_refs": [],
  "extensions": {}
}
```

OpenPlural should allow multiple systems in one file because Lighthouse and Ampersand can represent nested systems/subsystems and some users maintain more than one system-like grouping.

## Members

```json
{
  "id": "mem_01HV...",
  "system_id": "sys_01HV...",
  "name": "Alex",
  "display_name": "Alex",
  "pronouns": "they/them",
  "description": "Bio",
  "age": null,
  "birthday": {
    "value": "2001-04-29",
    "precision": "day",
    "year_visible": true
  },
  "color": "#88ccaa",
  "avatar_asset_id": null,
  "banner_asset_id": null,
  "proxy_tags": [
    { "prefix": "a:", "suffix": null }
  ],
  "is_custom_front": false,
  "archived": false,
  "created_at": "2026-04-29T18:00:00Z",
  "sort_order": 0,
  "privacy": {},
  "source_refs": [],
  "extensions": {}
}
```

Use `is_custom_front` for Simply Plural custom fronts and Ampersand custom fronts. Apps that can't represent custom fronts can import them as archived members, taxonomy labels, or report a loss.

Round-trip is lossy for apps (e.g. Sheaf) that don't preserve a dedicated custom-front flag — they encode the distinction in the description, and importers may need to re-parse if the source app is known.

## Groups And Taxonomy

Use groups for structured member organization:

```json
{
  "id": "grp_01HV...",
  "system_id": "sys_01HV...",
  "name": "Subsystem",
  "description": null,
  "color": "#ffaa00",
  "emoji": null,
  "parent_group_id": null,
  "sort_order": 0,
  "source_refs": [],
  "extensions": {}
}
```

Membership is normalized:

```json
{
  "id": "gm_01HV...",
  "group_id": "grp_01HV...",
  "member_id": "mem_01HV..."
}
```

Use taxonomy for non-hierarchical labels such as roles, tags, source labels, relationship categories, and app-local categories. Don't make `role` a privileged core string on members: Simply Plural, PluralKit, Sheaf, OpenSelves, and Octocon don't expose a universal role field, while Plural Star, Ampersand, and Lighthouse represent role-like data differently.

Terms:

```json
{
  "id": "term_01HV...",
  "system_id": "sys_01HV...",
  "kind": "role",
  "name": "caretaker",
  "description": null,
  "color": null,
  "parent_term_id": null,
  "source_refs": [],
  "extensions": {}
}
```

Assignments:

```json
{
  "id": "ta_01HV...",
  "term_id": "term_01HV...",
  "subject_type": "member",
  "subject_id": "mem_01HV...",
  "scope": "profile",
  "source_refs": [],
  "extensions": {}
}
```

Recommended `kind` values:

- `role`
- `tag`
- `source`
- `relationship`
- `identity`
- `topic`
- `status`
- `custom`
- `unknown`

Groups stay for folder-like hierarchy and subsystem organization. Tags map to taxonomy when the source treats them as labels rather than navigational groups. Custom fields are the fallback for arbitrary named values.

## Custom Fields

Definitions:

```json
{
  "id": "field_01HV...",
  "system_id": "sys_01HV...",
  "name": "Source",
  "field_type": "text",
  "options": null,
  "supports_markdown": false,
  "date_precision": null,
  "sort_order": 0,
  "privacy": {},
  "source_refs": [],
  "extensions": {}
}
```

Values:

```json
{
  "id": "fieldval_01HV...",
  "field_id": "field_01HV...",
  "subject_type": "member",
  "subject_id": "mem_01HV...",
  "value": "Example value",
  "source_refs": [],
  "extensions": {}
}
```

Recommended `field_type` enum:

- `text`
- `markdown`
- `number`
- `boolean`
- `color`
- `date`
- `date_range`
- `timestamp`
- `month`
- `year`
- `month_year`
- `month_day`
- `select`
- `multiselect`
- `json`

Importers should preserve unsupported types as `json` plus source extension data.

## Front History

Fronting is the hardest cross-app mismatch:

- PluralKit stores switch events.
- Prism, Simply Plural, OpenSelves, and Ampersand store per-member intervals.
- Sheaf stores grouped intervals with many members.
- Plural Star stores tiered periods with primary/co-front/co-conscious roles.

OpenPlural should support both canonical periods and optional source events.

### Front Periods

```json
{
  "id": "front_01HV...",
  "system_id": "sys_01HV...",
  "started_at": "2026-04-29T12:00:00Z",
  "ended_at": "2026-04-29T15:00:00Z",
  "assignments": [
    {
      "member_id": "mem_01HV...",
      "front_role": "primary",
      "confidence": null,
      "presence": null,
      "mood": null,
      "energy": null,
      "location": null,
      "note": null
    }
  ],
  "status": null,
  "note": "Optional whole-period note",
  "source_kind": "interval",
  "source_refs": [],
  "extensions": {}
}
```

Recommended front roles:

- `primary`
- `co_front`
- `co_conscious`
- `influencing`
- `member`
- `custom_status`
- `unknown`

Apps with flat co-fronting can use `member` or one `primary` plus `co_front` assignments. Apps that only know "these members were in the switch" can use `member`. `front_role` describes a fronting tier, not a member profile role.

### Front Events

```json
{
  "id": "event_01HV...",
  "system_id": "sys_01HV...",
  "at": "2026-04-29T12:00:00Z",
  "assignments": [
    { "member_id": "mem_01HV...", "front_role": "member" }
  ],
  "source_refs": [],
  "extensions": {}
}
```

`front_events` are optional but should be used for PluralKit imports to preserve the exact switch log. Exporters can derive `front_periods` from adjacent events for apps that need durations.

### Front Comments

Use time anchoring instead of only record anchoring:

```json
{
  "id": "fc_01HV...",
  "system_id": "sys_01HV...",
  "front_period_id": null,
  "target_time": "2026-04-29T13:00:00Z",
  "author_member_id": "mem_01HV...",
  "body": "Comment",
  "created_at": "2026-04-29T13:05:00Z",
  "source_refs": [],
  "extensions": {}
}
```

This maps well to Prism's newer comments and can still reference Simply Plural front-history comments.

## Notes And Journals

```json
{
  "id": "note_01HV...",
  "system_id": "sys_01HV...",
  "member_id": "mem_01HV...",
  "title": "Entry title",
  "body": "Markdown body",
  "created_at": "2026-04-29T18:00:00Z",
  "updated_at": null,
  "entry_date": "2026-04-29",
  "author_member_ids": ["mem_01HV..."],
  "color": null,
  "visibility": "system",
  "pinned": false,
  "content_warning": null,
  "source_refs": [],
  "extensions": {}
}
```

This can represent Prism notes, Simply Plural notes, Plural Star journals, Lighthouse journal posts, Sheaf journal entries, and Ampersand journal posts. Apps with separate "member notes" versus "journal entries" can set `extensions.openplural.note_kind`.

## Assets

```json
{
  "id": "asset_01HV...",
  "kind": "avatar",
  "mime_type": "image/png",
  "file_name": "avatar.png",
  "uri": null,
  "data_base64": "...",
  "data_uri": null,
  "size_bytes": 12345,
  "sha256": "...",
  "width": 512,
  "height": 512,
  "duration_ms": null,
  "source_refs": [],
  "extensions": {}
}
```

For v0.1, allow either:

- Inline base64 or Data URI for self-contained exports.
- URI-only references when the source app can't include bytes.

Importers should prefer content hashes for dedupe and shouldn't rely on URLs staying alive.

## Optional Modules

The following modules should be specified after core v0.1:

- `chat`: conversations, messages, reactions, replies, attachments.
- `boards`: member-addressed posts (walls, noteboards, board messages). Distinct from chat because the unit is a post with a target member, not a message in a thread.
- `relationships` *(provisional)*: member-to-member edges (partner, parent_of, metamour, etc.) with a separate type vocabulary that supports symmetric and asymmetric types. More draft than the rest — only one inspected app currently exports any of this, and the design will likely shift once more apps implement it.
- `polls`: polls, options, votes.
- `reminders`: periodic/event reminders.
- `habits`: habits and completions.
- `sharing`: friends, scopes, privacy buckets.
- `proxy`: PluralKit/Tupperbox/Discord proxy settings.
- `safety`: safety plans, destructive-action safeguards, lock/stealth settings.
- `settings`: app preferences that a target app may choose to ignore.

## Scope And Consent

An OpenPlural file can carry highly sensitive data — Simply Plural exports include every Mongo collection except accounts; Lighthouse exports BDA plans, inner worlds, rules, journals, and more. Producers should be explicit about what's included and consider per-module scoping or redaction so users can opt out of categories they don't want shared. A future spec version may formalize consent metadata; for now, treat the file as a whole-database dump unless the producer documents otherwise.

## Loss Reporting

Each export/import should be able to record losses:

```json
{
  "warnings": [
    {
      "level": "warning",
      "code": "module_not_supported",
      "record_type": "chat_messages",
      "message": "Target app does not support internal chat; messages were not imported.",
      "count": 158
    }
  ]
}
```

This should be part of both export validation and importer results. Users need to know what didn't move.

## Extension Strategy

Every record can include:

```json
{
  "extensions": {
    "simply_plural": {
      "privacyBuckets": ["..."]
    },
    "pluralkit": {
      "privacy": {}
    }
  }
}
```

Extension keys should be reverse-DNS or registered short app IDs. OpenPlural should define common app IDs:

- `prism`
- `simply_plural`
- `pluralkit`
- `octocon`
- `plural_star`
- `lighthouse`
- `sheaf`
- `openselves`
- `ampersand`

## Validation

Near-term implementation should include:

- JSON Schema for `openplural_version: "0.1"`.
- Fixture files for each researched source app.
- A validator CLI.
- A loss-report schema.
- Minimal reference converters for Prism, Simply Plural, PluralKit, and Plural Star.

## Recommended V0.1 Minimum

The first real spec should include:

- Envelope metadata and producer info.
- Systems.
- Members.
- Groups and group memberships.
- Taxonomy terms and assignments.
- Custom fields and values.
- Front periods.
- Front events.
- Front comments.
- Notes/journals.
- Assets.
- Source refs.
- Extensions.
- Warnings/losses.

Everything else should be optional modules so the core can stabilize quickly.
