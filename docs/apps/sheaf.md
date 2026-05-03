# Sheaf

## Source Status

Sources:

- Repository: https://github.com/sheaf-project/sheaf

## Storage And API Shape

Sheaf is a FastAPI/PostgreSQL application. It uses SQLAlchemy models and application-level encryption for sensitive member/journal content.

The current `/v1/export` route returns JSON `version: "1"`.

```json
{
  "version": "1",
  "system": {},
  "members": [],
  "fronts": [],
  "groups": [],
  "tags": [],
  "custom_fields": []
}
```

If a user has no system, export returns `system: null` and empty arrays. The inspected no-system branch uses `fields` while the normal export uses `custom_fields`, which looks like an edge-case naming inconsistency to account for in importers.

## Records

### System

`System`:

- `id`, `user_id`.
- `name`, `description`, `tag`, `avatar_url`, `color`.
- `privacy`: `public`, `friends`, or `private`.
- `date_format`: `dmy`, `mdy`, or `ymd`.
- `replace_fronts_default`: whether starting a front ends open fronts by default.
- System Safety settings for destructive actions:
  - Auth tier (`none`, `password`, `totp`, `both`).
  - Grace period days.
  - Per-category toggles for members, groups, tags, fields, fronts, journals, images.
- Journal retention overrides.

The export route includes only profile fields: id, name, description, tag, avatar URL, color, privacy.

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

### Journals And Files

Sheaf has models for journals and uploaded files, but the current export route doesn't include them.

`JournalEntry` includes:

- System/member association.
- Encrypted title/body.
- Visibility.
- Fallback author user.
- Frozen author member ID/name snapshot.
- Image storage keys.
- Revision retention support via services.

`UploadedFile` includes:

- User ID, storage key, purpose, content type, size, created time.

### Client Settings

`ClientSettings` stores per-user/per-client JSON settings, also not currently included in `/v1/export`.

## Imports

Sheaf has:

- A Simply Plural import service.
- A Sheaf import service for its own export format.

Simply Plural import maps:

- Members.
- Custom fronts as members marked in description.
- Custom fields and embedded member `info` values.
- Groups and group memberships.
- Front history.
- System profile.

The service currently skips Simply Plural notes with a warning until journal import is implemented.

## Import/Interoperability Notes

Sheaf is close to a small OpenPlural core:

- System.
- Members.
- Front intervals with co-fronting.
- Hierarchical groups.
- Tags.
- Custom field definitions and values.

It also shows a useful implementation boundary: models may exist before export/import support. OpenPlural conformance should test actual exported modules, not just database tables.
