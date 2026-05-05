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

- `id`, `name`, `description`, `tag`, `avatar_url`, `color`, `privacy`.
- `replace_fronts_default`, `date_format`, `delete_confirmation`.
- `safety`.
- `retention`.

### Members

`Member`:

- `id`, `system_id`.
- Encrypted `name` and `description`; `name_hash` blind index.
- `display_name`, `pronouns`, `avatar_url`, `color`, `birthday`.
- `privacy`.
- Relationships to fronts, groups, tags, custom field values.

Export decrypts name and description:

- `id`, `name`, `display_name`, `description`, `pronouns`, `avatar_url`, `color`, `birthday`, `privacy`, `created_at`.

### Fronts

`Front`:

- `id`, `system_id`.
- `started_at`, optional `ended_at`.
- Many-to-many members through `front_members`.

Export:

- `id`, `started_at`, `ended_at`, `member_ids`.

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

Sheaf has:

- A Simply Plural import service.
- A Sheaf import service for its own export format.

But the current self-import path has not caught up to export v2 yet:

- `sheaf/api/v1/sheaf_import.py` still rejects any file whose `version` is not `"1"`.
- `sheaf/services/sheaf_import.py` only restores:
  - basic system profile fields (`name`, `description`, `tag`, `color`, `privacy`).
  - members.
  - fronts.
  - groups.
  - tags.
  - custom fields and values.

It does **not** currently import v2-only data such as:

- system preferences (`date_format`, `replace_fronts_default`, `delete_confirmation`).
- `safety` / `retention`.
- journals.
- revisions.
- watch tokens / channels.
- uploaded-file inventory.

So the upstream changelog's "re-importable" claim is ahead of the checked-in parser at this snapshot.

## Import/Interoperability Notes

Sheaf now covers more of the OpenPlural core directly than the original v1 research captured:

- System.
- Members.
- Front intervals with co-fronting.
- Hierarchical groups.
- Tags.
- Custom field definitions and values.
- Notes/journals.

The remaining gaps are not all the same kind:

- `revisions[]` has no first-class OpenPlural record today; preserve it under `extensions` if needed.
- `watch_tokens[]` likewise fits best in `extensions` until there is a notification/export module.
- `uploaded_files[]` in sync JSON is inventory-only metadata, not enough by itself to emit self-contained OpenPlural `assets[]`.
- The async zip is the better converter target when image portability matters, because it actually includes the `images/<key>` blobs referenced by journal `image_keys`.

It also shows a useful implementation boundary: exporter coverage has moved ahead of importer coverage. OpenPlural conformance should test actual exported and imported modules separately, not assume round-trip parity inside the source app.
