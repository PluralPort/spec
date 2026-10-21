# PluralKit

## Source Status

Sources:

- Repository: https://github.com/PluralKit/PluralKit
- API docs: https://pluralkit.me/api/
- Model docs: https://pluralkit.me/api/models/
- Source files of interest: `PluralKit.Core/Models`, `PluralKit.Core/Services/DataFileService.cs`.

## API And Export Shape

PluralKit is a Discord bot and API. Its portable export is a JSON datafile, currently `version: 2`, produced by `DataFileService.ExportSystem`.

The export merges system JSON with additional owner-only data:

```json
{
  "version": 2,
  "id": "abcde",
  "uuid": "...",
  "name": "...",
  "description": "...",
  "tag": "...",
  "members": [],
  "groups": [],
  "switches": [],
  "accounts": [],
  "config": {}
}
```

## Records

### System

System model fields include:

- `id`: short human-facing ID.
- `uuid`: stable UUID.
- `name`, `description`, `tag`, `pronouns`.
- `avatar_url`, `banner`, `color`, `created`.
- Owner-only `webhook_url`.
- Privacy object:
  - `name_privacy`
  - `avatar_privacy`
  - `description_privacy`
  - `banner_privacy`
  - `pronoun_privacy`
  - `member_list_privacy`
  - `group_list_privacy`
  - `front_privacy`
  - `front_history_privacy`

### Members

Member model fields include:

- `id`, `uuid`, optional `system`.
- `name`, `display_name`, `color`.
- `birthday`, `pronouns`, `avatar_url`, `webhook_avatar_url`, `banner`.
- `description`, `created`.
- `proxy_tags`: array of `{ "prefix": "...", "suffix": "..." }`.
- `keep_proxy`, `tts`.
- Owner-only `autoproxy_enabled`, message count, last message timestamp.
- Owner-only privacy object:
  - `visibility`
  - `name_privacy`
  - `description_privacy`
  - `banner_privacy`
  - `birthday_privacy`
  - `pronoun_privacy`
  - `avatar_privacy`
  - `metadata_privacy`
  - `proxy_privacy`

Birthdays can be year-hidden using sentinel years in the backing model.

### Groups

Group model fields:

- `id`, `uuid`, `name`, optional `system`.
- `display_name`, `description`.
- `icon`, `banner`, `color`, `created`.
- `members` array when requested/exported.
- Owner-only privacy object:
  - `name_privacy`
  - `description_privacy`
  - `banner_privacy`
  - `icon_privacy`
  - `list_privacy`
  - `metadata_privacy`
  - `visibility`

### Switches / Front History

PluralKit stores switch events, not explicit intervals:

- Each switch has a timestamp and an ordered member list.
- Export serializes switches as:

```json
{
  "timestamp": "2026-04-29T12:00:00Z",
  "members": ["abcde", "fghij"]
}
```

Durations are inferred from adjacent switch timestamps. The latest switch may be open-ended.

### Discord-Specific Data

PluralKit also stores Discord-specific data not central to most system-profile exports:

- Proxied message records.
- Guild/member guild settings.
- Autoproxy and system config.
- Linked Discord accounts.

The core datafile export focuses on systems, members, groups, switches, accounts, and config.

## Import/Interoperability Notes

PluralKit is narrower than Simply Plural or Prism but has strong identity/proxy semantics:

- Preserve both short IDs and UUIDs.
- Preserve proxy tags and privacy settings.
- Support event-based front history or an event-to-interval transform.
- Groups are flat in PluralKit, but PluralPort should support membership arrays and optional hierarchy for other apps.
