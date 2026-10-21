# Simply Plural

## Source Status

Sources:

- API source: https://github.com/ApparyllisOrg/SimplyPluralApi
- Discontinuation announcement: https://apparyllis.com/simply-plural-will-be-discontinued/
- Cross-referenced against Prism's and Lighthouse's importers, which consume the public token API.

Simply Plural announced discontinuation on March 7, 2026, with servers shutting down July 1, 2026. Its API and export shape still matter because it has a large installed base and many migration paths.

## API And Export Shape

The public API exposes token-authenticated endpoints for members, fronting, groups, fields, notes, polls, chat, reminders, privacy buckets, friends, and integration data.

The full data export is more raw: the API source builds a JSON object keyed by MongoDB collection name for the account's `uid`. The exporter skips account credentials and decrypts chat messages before including them.

Representative export shape:

```json
{
  "users": [],
  "private": [],
  "members": [],
  "frontHistory": [],
  "frontStatuses": [],
  "groups": [],
  "customFields": [],
  "notes": [],
  "comments": [],
  "polls": [],
  "channels": [],
  "channelCategories": [],
  "chatMessages": [],
  "boardMessages": [],
  "automatedReminders": [],
  "repeatedReminders": [],
  "privacyBuckets": [],
  "friends": []
}
```

## Records

### User/System Profile

`users` stores system-level public profile data:

- `_id`/uid, `username`, `desc`, `color`.
- Avatar URL/UUID.
- Markdown and cosmetic/profile settings.
- Legacy custom field data may be present.

`private` stores account/system settings:

- Categories, timezone/location, notification tokens, dashboard/default settings, audit/security preferences, and other private app state.

### Members

`members` records include:

- `_id`, `uid`, `name`, `desc`, `pronouns`, `pkId`.
- `color`, `avatarUrl`, `avatarUuid`, avatar frame/cosmetic data.
- Privacy flags such as `private`, `preventTrusted`, `preventsFrontNotifs`.
- `info`: custom field values keyed by custom field ID.
- Markdown flag, archived flag, privacy bucket references.

### Custom Fronts

`frontStatuses` are non-member fronting entities:

- Name, description, color, avatar, privacy, bucket settings.
- Front history can reference these as custom statuses.

### Front History

`frontHistory` is per-member/per-status interval data:

- `_id`, `uid`, `member`.
- `startTime`, `endTime` as epoch milliseconds.
- `live` for currently active fronts.
- `custom`, `customStatus` for custom front/status references.

Co-fronting is modeled by overlapping rows. A live entry has no `endTime`.

`comments` can attach text to `frontHistory` rows:

- `_id`, `uid`, `documentId`, `collection: "frontHistory"`, `text`, `time`, markdown flags.

### Groups

`groups` records:

- `_id`, `uid`, `name`, `desc`, `color`, `emoji`.
- `parent`, with `"root"` for top-level groups.
- `members` array of member/custom front IDs.
- Privacy bucket settings.

### Custom Fields

`customFields` definitions:

- `_id`, `uid`, `name`, `type`, `order`, `supportMarkdown`, privacy/bucket data.

Observed type IDs:

| Type | Meaning |
| --- | --- |
| `0` | Text/string |
| `1` | Color |
| `2` | Full date |
| `3` | Month |
| `4` | Year |
| `5` | Month/year |
| `6` | Timestamp |
| `7` | Month/day |

Values are embedded on members in `members.info`, not stored in a separate values collection.

### Notes

`notes` are per-member notes:

- `_id`, `uid`, `member`, `title`, `note`, `color`, `date`, markdown support.

### Communication

Simply Plural has two communication surfaces:

- `boardMessages`: member-addressed board messages with title/body, written by/for, timestamps, read state, markdown.
- `channels`, `channelCategories`, `chatMessages`: Discord-like chat. Messages include content, channel, writer, timestamp, reply/edit metadata, reactions/mentions in newer versions, and encryption metadata. Export decrypts message text.

### Polls

`polls` include:

- `_id`, `uid`, `name`, `desc`, custom poll flag.
- `endTime`, abstain/veto settings for standard polls.
- Custom `options`.
- Embedded `votes` with voter ID, vote choice, and optional comment.

### Reminders, Privacy, Friends

Other collections include:

- `automatedReminders`, `repeatedReminders`.
- `privacyBuckets` for fine-grained sharing.
- `friends`, filters, tokens, and security logs.

## Import/Interoperability Notes

Simply Plural is one of the broadest data models and a likely stress test for PluralPort. It argues for:

- Raw source ID preservation.
- Both interval-based fronting and custom front/status support.
- Hierarchical groups.
- Custom field definitions plus per-member value maps.
- Privacy bucket data as an extension rather than a hard core requirement.
- Optional modules for chat, boards (Simply Plural's `boardMessages`), polls, reminders, and friend sharing.
