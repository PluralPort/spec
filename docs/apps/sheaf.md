# Sheaf

## Source Status

Sources:

- Repository: https://github.com/sheaf-project/sheaf
- Export route: `sheaf/api/v1/export.py`
- Async export builder: `sheaf/services/export_builder.py`
- Sheaf-import endpoints: `sheaf/api/v1/sheaf_import.py`, `sheaf/services/sheaf_import.py`
- Coverage: `tests/test_export.py`, `tests/test_account_export_completeness.py`
- Changelog: `CHANGELOG.md`

## Storage And API Shape

Sheaf is a FastAPI/PostgreSQL application. It uses SQLAlchemy models and application-level encryption for sensitive member, journal, revision, and notification-secret content.

The current sync `/v1/export` route returns JSON `version: "2"`. There is also an async `POST /v1/export/jobs` flow that writes a zip containing the same `export.json` plus `images/<key>` blobs for uploaded files.

```json
{
  "version": "2",
  "system": {},
  "members": [],
  "fronts": [],
  "groups": [],
  "tags": [],
  "custom_fields": [],
  "journals": [],
  "revisions": [],
  "watch_tokens": [],
  "uploaded_files": []
}
```

If a user has no system, export returns `system: null` and empty arrays for every top-level collection above. The older v1 `fields` vs `custom_fields` inconsistency is gone in the current source: the empty branch now also uses `custom_fields`.

## Records

### System

`System`:

- `id`, `user_id`.
- `name`, `description`, `tag`, `avatar_url`, `color`.
- Encrypted `note` (separate from `description`; decrypted on export).
- `privacy`: `public`, `friends`, or `private`.
- `date_format`: `dmy`, `mdy`, or `ymd`.
- `replace_fronts_default`: whether starting a front ends open fronts by default.
- `delete_confirmation`: auth tier enum reused by System Safety (`none`, `password`, `totp`, `both`).
- System Safety settings:
  - `grace_period_days`.
  - Per-category toggles for `members`, `groups`, `tags`, `fields`, `fronts`, `journals`, `images`, `revisions`, and `notifications`.
  - `auto_pin_first_revision`.
- Retention overrides:
  - `journal_max_revisions`.
  - `journal_max_revision_days`.
  - `pinned_revision_max_per_target`.

Current export includes both profile fields and these preference/safety/retention blocks:

- `id`, `name`, `description`, `note`, `tag`, `avatar_url`, `color`, `privacy`.
- `replace_fronts_default`, `date_format`, `delete_confirmation`.
- `safety`.
- `retention`.

### Members

`Member`:

- `id`, `system_id`.
- Encrypted `name` and `description`; `name_hash` blind index.
- `display_name`, `pronouns`, `avatar_url`, `color`, `birthday`, `emoji`.
- `pluralkit_id`: optional 5-char PluralKit member hid for users who sync with PK.
- `is_custom_front`: boolean flag distinguishing custom fronts from members.
- Encrypted `note` (separate from `description`; decrypted on export).
- `privacy`.
- Relationships to fronts, groups, tags, custom field values.

Export decrypts name, description, and note:

- `id`, `name`, `display_name`, `description`, `pronouns`, `avatar_url`, `color`, `birthday`, `pluralkit_id`, `emoji`, `is_custom_front`, `privacy`, `note`, `created_at`.

### Fronts

`Front`:

- `id`, `system_id`.
- `started_at`, optional `ended_at`.
- Encrypted `custom_status` (free-text per-front status string; decrypted on export).
- Many-to-many members through `front_members`.

Export:

- `id`, `started_at`, `ended_at`, `member_ids`, `custom_status`.

Sheaf supports co-fronting through the join table.

### Groups

`Group`:

- `id`, `system_id`, `name`, `description`, `color`.
- Optional `parent_id` for nesting.
- Many-to-many members through `group_members`.

Export:

- `id`, `name`, `description`, `color`, `parent_id`, `member_ids`.

### Tags

`Tag`:

- `id`, `system_id`, `name`, `color`.
- Many-to-many members through `member_tags`.

Export:

- `id`, `name`, `color`, `member_ids`.

### Custom Fields

`CustomFieldDefinition`:

- `id`, `system_id`, `name`, `field_type`, `options`, `order`, `privacy`.
- `options` is JSONB `dict | None` — typically a record of select/multiselect choices, not a string array.

Supported field types:

- `text`
- `number`
- `date`
- `boolean`
- `select`
- `multiselect`

`CustomFieldValue`:

- `id`, `field_id`, `member_id`, `value` as JSON.

Export nests values inside each field definition:

```json
{
  "id": "...",
  "name": "...",
  "field_type": "text",
  "options": null,
  "order": 0,
  "privacy": "private",
  "values": [
    { "member_id": "...", "value": { "v": "..." } }
  ]
}
```

### Journals

`JournalEntry` is now in `/v1/export`:

- `id`.
- `member_id` or null for system-wide entries.
- Decrypted `title`, `body`.
- `visibility`.
- `author_user_id`.
- Frozen `author_member_ids`, `author_member_names`.
- `image_keys`.
- `created_at`, `updated_at`.

Important nuance: the current journal model only actively uses `visibility: "system"`; other enum-ish values are reserved in the model for later.

### Revisions

`ContentRevision` is also exported now. This is edit history for markdown-bodied content, currently:

- `target_type: "journal_entry" | "member_bio"`.
- `target_id`.
- `user_id`.
- Frozen `editor_member_ids`, `editor_member_names`.
- Decrypted `title`, `body`.
- `image_keys`.
- `pinned_at`.
- `created_at`.

These are historical snapshots of superseded content; the current body still lives on the journal or member row itself.

### Messages

Sheaf exports board messages from two surfaces: a system-wide global board, and per-member walls. Both share the same shape.

`Message`:

- `id`, `system_id`.
- `board_kind`: `"system"` or `"member"`.
- `board_member_id`: the recipient member for member-wall posts; null for system-board posts.
- `author_member_id`: nullable; null when the author has been deleted.
- `parent_message_id`: nullable; single-level reply pointer. The UI renders flat with a "Replying to X" backlink rather than a tree. `parent_message_id` may itself be a reply, so a chain forms naturally.
- Encrypted `body` (markdown, decrypted on export).
- `deleted_at`: soft-delete tombstone; the export excludes soft-deleted rows.
- `created_at`, `updated_at`.

Edit history rides the same polymorphic `content_revisions` surface as journals and member bios (`target_type: "message"`).

Export omits soft-deleted rows:

- `id`, `board_kind`, `board_member_id`, `author_member_id`, `parent_message_id`, `body`, `created_at`, `updated_at`.

### Polls

`Poll`:

- `id`, `system_id`.
- Encrypted `question`, optional encrypted `description` (decrypted on export).
- `kind`: poll type (single, multi-select, ranked, etc.).
- `results_visibility`: when results are visible to voters.
- `closes_at`: poll deadline.
- `retention_days`: how long the poll is kept after closing.
- `include_custom_fronts`: whether custom fronts are eligible to vote.
- Options with encrypted `text` and a positional `order`.
- Votes with `voted_as_member_id` and `option_ids[]`.
- Audit events: cast, change, withdraw, close, with frozen `voted_as_member_id`, `fronting_member_ids`, and an `actor_user_id`.

Export:

- `id`, `question`, `description`, `kind`, `results_visibility`, `closes_at`, `retention_days`, `include_custom_fronts`, `created_at`.
- `options[]` with `id`, `text`, `position`.
- `votes[]` with `voted_as_member_id`, `option_ids[]`, `created_at`, `updated_at`.
- `events[]` with `id`, `voted_as_member_id`, `action`, `option_ids[]`, `fronting_member_ids[]`, `actor_user_id`, `created_at`.

### Reminders

`Reminder`:

- `id`, `system_id`, `channel_id` (references a notification channel).
- `name`.
- Encrypted `title`, optional encrypted `body` (decrypted on export).
- `enabled`.
- `trigger_type`: time-based or member-event-based.
- Member-event triggers: `trigger_member_id`, `trigger_event` (e.g., on-front-start), `delay_seconds`.
- Time-based triggers: `schedule_kind`, `schedule_time`, `schedule_dow_mask`, `schedule_dom`, `schedule_tz`, or a raw `cron_expression`.
- Scope: all members, current fronters, or specific member set via `scope_member_ids[]`.
- `digest_when_absent`: whether to digest if no member matches scope at fire time.

Export omits runtime state (pending queue, last_fired_at) but preserves the trigger and scope configuration.

### Watch Tokens And Notification Channels

Sheaf now exports owner-side front-change notification config:

- `watch_tokens[]` with `id`, `label`, `revoked_at`, `created_at`.
- Nested `channels[]` with:
  - destination type/config.
  - base visibility filters.
  - start/stop/cofront triggers.
  - redaction/sensitivity/debounce/aggregation settings.
  - quiet-hours config.
  - group rules and member rules.

The export intentionally omits non-portable or security-sensitive per-instance state:

- activation hashes/tokens.
- recipient redemption state.
- `last_delivered_at`.
- webhook secret ciphertext.

### Files And Images

`UploadedFile` is now partially exported:

- `id`, `key`, `size_bytes`, `content_type`, `created_at`.

This sync JSON is only a file inventory. The binary bytes do **not** ride along in `/v1/export`.

For bytes, the async export job zip contains:

- `export.json`.
- `README.txt`.
- `images/<key>` blobs for every uploaded file the user owns.

That means Sheaf now has two distinct portability surfaces:

- sync JSON for structured records.
- async zip for structured records plus image bytes.

### Client Settings And Account Data

`ClientSettings` is still not part of `/v1/export`. In current source it moved to the separate Article 15 account-data endpoint (`POST /v1/account/data`) alongside sessions, trusted devices, API-key metadata, and other account/audit information.

## Imports

Imports run through an async job runner (`POST /v1/imports/file`, polled for status). Sheaf ships importers for its own export, an export-with-images archive, and several foreign formats: PluralKit (file and API), Tupperbox, Simply Plural, PluralSpace, Prism, Ampersand, and OpenPlural.

The native self-importer has caught up to export v2. It accepts `version` `"1"` or `"2"` and round-trips the full export, including the data an earlier snapshot of this page noted it could not:

- system profile plus preferences (`date_format`, `replace_fronts_default`, `coalesce_contiguous_fronts`, `delete_confirmation`) and the `safety` / `retention` blocks.
- members, fronts, groups, tags, custom fields and values.
- journals and content revisions.
- board messages and polls (with their audit events).
- reminders and the watch-token / notification-channel config.

Re-import is idempotent: members dedupe against the target roster, everything else dedupes by preserved source timestamps, and a chosen conflict strategy decides skip / update / create. Every importer enforces a tier member cap, routes decoded JSON through a json-bomb-guarded loader, bounds decompressed archive size, and normalises foreign avatar URLs through the same policy gate the create API uses.

## Import/Interoperability Notes

Sheaf now covers more of the OpenPlural core directly than the original v1 research captured:

- System.
- Members (including a dedicated `is_custom_front` boolean, so Simply Plural custom fronts and PluralKit member identity both round-trip without extension fallback).
- Front intervals with co-fronting, plus a free-text `custom_status` per period.
- Hierarchical groups.
- Tags.
- Custom field definitions and values.
- Notes/journals.

The remaining gaps are not all the same kind:

- `revisions[]` has no first-class OpenPlural record today; preserve it under `extensions` if needed.
- `messages[]` map to the boards module, but the single-level reply pointer (`parent_message_id`) has no v0.1 home and lands in `extensions.sheaf` unless `BoardPost` grows a reply field.
- `polls[]` and `reminders[]` are full Sheaf surfaces but neither has a v0.1 module; both fit under `extensions.sheaf` as preserved-only until the relevant modules land.
- `watch_tokens[]` likewise fits best in `extensions` until there is a notification/export module.
- `uploaded_files[]` in sync JSON is inventory-only metadata, not enough by itself to emit self-contained OpenPlural `assets[]`.
- The async zip is the better converter target when image portability matters, because it actually includes the `images/<key>` blobs referenced by journal `image_keys`.

OpenPlural conformance should still test exported and imported modules separately rather than assuming round-trip parity inside any one app, but in Sheaf's case the importer now covers the same surfaces the exporter does.

## OpenPlural support

Sheaf ships a native OpenPlural v0.1 exporter and importer (it is one of the format's founding adopters).

Export has two shapes:

- Sync `GET /v1/export?format=openplural`: a single JSON envelope with uri-only `assets` (no bytes), emitting an `asset_uri_only` warning.
- An async `.openplural.zip` bundle (`openplural.json` plus `assets/<key>` blobs) for image portability.

Both stamp `producer` (`app`, `app_id: "sheaf"`, `app_version`, `exporter_version`) and append an `extensions.sheaf.lineage[]` entry per export. `pluralkit_id` is emitted as a `source_ref`. Everything Sheaf has that v0.1 has no core record for (the `note` fields, polls, reminders, revisions, notification config, System Safety settings, member emoji / quick-switch pin, and the board-post reply pointer) is preserved under the registered `sheaf` namespace, so a Sheaf round-trip is lossless.

Import (`source=openplural_file`) accepts either shape, sniffing the zip magic, and has a preview endpoint. It reads `front_periods` and also derives intervals from `front_events` (the switch-log shape), so an event-log file imports its fronting history. It rejects any unfamiliar `openplural_version`.

So Sheaf is not a lossy hop for files from other apps, data it cannot model (other apps' `extensions` namespaces, the `chat` and `relationships` modules, `front_comments`, and non-tag taxonomy) is preserved on import and re-merged into the next OpenPlural export. Per-record foreign `extensions` are the one tier not yet preserved (they need stable per-record identity); the importer reports them rather than dropping them silently. This is the baseline of the extensions-preservation contract discussed in the issue tracker.
