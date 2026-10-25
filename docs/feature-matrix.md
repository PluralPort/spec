# Feature Matrix

This table compares the researched apps at the level OpenPlural needs for portability. "Yes" means the feature is present in an exported/API shape or strongly supported by source. "Partial" means the app has some related data, but not enough for a full portable mapping. "Unknown" means public docs/source didn't expose the shape.

| App | Source status | Export/API shape | System profile | Members | Fronting | Groups/tags | Custom fields | Journals/notes | Chat/messages | Polls | Reminders/habits | Assets/media | Privacy/sharing |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Prism | Local source | Encrypted `.prism` JSON envelope | Yes | Yes | Per-member intervals plus sleep | Hierarchical groups | Definitions and values | Notes | Internal chat with media; member board posts (sync-only, not in export envelope) | Yes | Habits and reminders | Yes | Local friend/share metadata |
| Sheaf | OSS | `/v1/export` JSON v2; OpenPlural JSON; `.openplural.zip` with `openplural.json` + `assets/` | Yes | Yes | Co-front intervals | Hierarchical groups and tags | Definitions and values | Journals plus revision history | Board messages (system + per-member walls) with single-level replies | Polls with options, votes, and an audit log | Reminders (time-of-day, day-of-week, cron, or member-event triggers); plus safety/retention settings | Avatar URLs + uploaded-file inventory; OpenPlural bundle includes image blobs under `assets/`; referenced inline avatars, banners, and note attachments import | Privacy enum, safety controls, member/group relationship graph, protective share guards, watch-token notification config |
| PluralSpace | Closed server; public client inspected | Older GDPR ZIP with `data.json`; newer OpenPlural ZIP with root `openplural.json` + empty `media/`; shipped OpenPlural JSON/ZIP import UI | Yes | Yes | Per-member intervals; co-fronting via overlapping rows | Nested groups in newer OpenPlural fixture | Definitions; populated value shape unverified | Journal entries with numeric visibility levels | Channels/messages in OpenPlural fixture | System-level polls | Not observed | `media/` dir observed, but no asset bytes in inspected OpenPlural fixture; importer advertises ZIP media support | `visibility` on system; numeric scale on entries |
| Simply Plural | OSS API | Mongo collection export plus token API | Yes | Yes | Per-member/status intervals | Hierarchical groups | Definitions, values embedded on members | Notes | Chat and board messages | Yes | Automated/repeated reminders | Avatar/media URLs | Privacy buckets and friends |
| PluralKit | OSS | API and datafile v2 JSON | Yes | Yes | Switch events with member list | Flat groups | No | No | Discord proxy records outside core export | No | No | Avatar/banner URLs | Rich privacy model |
| Octocon | OSS | Discord-bot slash export: PK datafile v2 or Octocon-native "full" JSON | Yes | Yes | Per-alter intervals with short comment | Hierarchical tags (used as groups) | Definitions on user, string values on alters; `text`/`number`/`boolean` | System + per-alter journals (in schema, not in export) | Discord proxy fields on alter | Polls record exists, internal shape not surveyed | Partial (no habit/reminder module observed) | Avatar URLs (extra-images field unused) | Four-level alter/tag security; bidirectional friendships with `friend`/`trusted_friend` |
| Plural Star | OSS | Local backup JSON v1.2 | Yes | Yes | Tiered history: primary/co-front/co-conscious | Flat groups | Definitions and values | Journal | Channels/messages | Member polls | Notifications/settings, no habit module found | Avatar/banner dictionaries | Share/settings data |
| Lighthouse | OSS | ZIP of CSVs plus narrow token API | Yes | Yes | Not implemented in inspected schema | Subsystems as systems | No generic custom fields found | Journals and communal journals | Forum/thread posts | No system polls found | Safety/BDA plans, rules, wishlist | Image URLs/blobs | Token permissions |
| OpenSelves | OSS | Sync log DTOs, no export found | No separate system | Yes | Per-member intervals | No | No | No | No | No | No | Member image | Account auth only |
| Ampersand | OSS | Native JSON export consumed by PluralPort's OpenPlural converter; `.ampar` self-backup; imports SP / Octocon / PK / Tupperbox | Yes | Yes | Per-member intervals with main/influencing/custom-status/presence | Typed tags and nested systems | Definitions, string values on members | Journal posts | Board messages | Polls on board messages | Reminders | Inline data-URI assets + asset manager | Local app/security config (excluded by PluralPort converter) |
| Tupperbox | Closed-source Discord bot | `tul!export` JSON | No | Yes | No | Flat groups, one-per-member | No | No | No (proxy bot only) | No | No | Avatar URL (CDN) + banner | No privacy model |

## Fronting Models

| App | Model | OpenPlural implication |
| --- | --- | --- |
| Prism | Overlapping per-member intervals. | Need interval records and co-front inference. |
| Simply Plural | Overlapping per-member/custom-status intervals. | Need custom front/status assignment support. |
| PluralKit | Switch events, duration inferred from next event. | Need event form or lossless source event extension. |
| Plural Star | One history entry can contain primary, co-front, and co-conscious tiers. | Need role/tier on front assignments. |
| PluralSpace | One row per member per period; co-fronting via overlapping rows with identical timestamps. | Importer must group-by timestamps to build periods with multiple assignments. |
| Octocon | Per-alter intervals with optional short comment; no grouped period record. | Need interval records and single-member assignments per row. |
| Sheaf | One front interval with many members. | Need grouped interval periods with assignments. |
| OpenSelves | One member per interval. | Need simple subset import path. |
| Ampersand | One member per interval with main/influencing/custom-status/presence. | Need optional assignment metadata. |
| Lighthouse | Stub only in inspected source. | Do not assume support. |

## Custom Field Models

| App | Definition location | Value location | Types |
| --- | --- | --- | --- |
| Prism | `customFields` | `customFieldValues` | Typed, includes date precision. |
| Simply Plural | `customFields` | `members.info` map | Numeric date/text/color variants. |
| Plural Star | `customFieldDefs` | Member custom values | Rich text/date/range/number/toggle/color variants. |
| PluralSpace | `custom_fields[]` | Nested under definition (Sheaf-style); empty in inspected sample | `text`, `date` observed; `is_multiple` toggle. |
| Sheaf | `custom_fields[]` (export; DB tables are `custom_field_definitions`/`custom_field_values`) | Nested `custom_fields[].values[]` in `/v1/export` | Text, number, date, boolean, select, multiselect. |
| Ampersand | `customFields` | `members.customFields` map | No type enum; values are always strings. |
| PluralKit | None | None | Not applicable. |
| OpenSelves | None | None | Not applicable. |
| Lighthouse | Rich fixed alter columns | Table columns | Not generic custom fields. |
| Octocon | `Accounts.Field` (definitions live on the user) | `Alters.Field` (embedded list on each alter) | `text`, `number`, `boolean`; values are stringly typed regardless. |

## Broadest Common Core

The strongest common set across serious migration targets is:

- Export metadata.
- One or more systems.
- Member profiles.
- Front history.
- Groups plus taxonomy/tags.
- Custom fields.
- Notes/journals.
- Assets/profile images.
- Source references and extension data.

Chat, polls, reminders, habits, friend sharing, safety plans, and Discord proxy settings should be optional modules rather than required core.
