# Plural Star

## Source Status

Sources:

- Repository: https://github.com/TheHanyou/Plural-Star

Plural Star is a React Native app, formerly named "Plural Space" (with a space) before its rebrand. It is **not** the same product as PluralSpace (no space) at `pluralspace.app`, which is a separate web app that kept its original name; see [`pluralspace.md`](pluralspace.md).

## Storage And Export Shape

Plural Star is a React Native app using `AsyncStorage` plus filesystem backups. Storage keys include:

- `ps:system`
- `ps:members`
- `ps:front`
- `ps:history`
- `ps:journal`
- `ps:share`
- `ps:settings`
- `ps:groups`
- `ps:palettes`
- `ps:chatChannels`
- `ps:customFieldDefs`
- `ps:noteboards`
- `ps:lastNoteboardSeen`
- `ps:polls`
- Per-channel messages under `ps:chat:<channelId>`

Critical keys are also backed up as JSON files under the app document directory.

The export payload has `_meta` plus selected categories:

```json
{
  "_meta": {
    "version": "1.2",
    "app": "Plural Star",
    "exportedAt": "..."
  },
  "system": {},
  "members": [],
  "front": {},
  "frontHistory": [],
  "journal": [],
  "groups": [],
  "chatChannels": [],
  "chatMessages": [],
  "settings": {},
  "customFieldDefs": [],
  "customMoods": [],
  "noteboards": [],
  "polls": [],
  "palettes": [],
  "avatars": {},
  "banners": {}
}
```

## Records

### System

`SystemInfo`:

- `name`
- `description`
- Optional `journalPassword`
- `avatar`
- `banner`

### Members

`Member`:

- `id`, `name`, `pronouns`, `role`, `color`, `description`.
- Optional `tags`, `groupIds`, `archived`, `avatar`, `banner`.
- Optional `customFields`.
- Optional `sortOrder`, `createdAt`.

Avatar/banner file paths are stripped during export. Image bytes can be exported separately in `avatars` and `banners` dictionaries keyed by member ID.

### Groups

`MemberGroup`:

- `id`, `name`, optional `color`.

Plural Star's local group model is flat; member membership lives on `Member.groupIds`.

### Custom Fields

`CustomFieldDef`:

- `id`, `name`, `type`, optional `sortOrder`, optional markdown flag.

Types include:

- `text`
- `markdown`
- `date`
- `dateRange`
- `number`
- `toggle`
- `color`
- `month`
- `year`
- `monthYear`
- `timestamp`
- `monthDay`

`CustomFieldValue` stores `fieldId` and `value` as string, number, boolean, or null.

### Fronting

Plural Star has a tiered front model:

```ts
FrontState {
  primary: FrontTier,
  coFront: FrontTier,
  coConscious: FrontTier,
  startTime: string
}
```

`FrontTier` includes:

- `memberIds`
- Optional `mood`
- Optional `note`
- Optional `location`
- Optional `energyLevel`

History entries preserve the same idea:

- `memberIds`, `startTime`, `endTime`, `note`.
- Optional mood/location/energy.
- Optional `coFrontIds`, `coConsciousIds` with separate notes/mood/energy.
- Optional change metadata: `changeType`, `changeTime`, `changeTier`.

This is more expressive than simple overlapping per-member rows because it distinguishes primary front, co-front, and co-conscious presence.

### Journal

`JournalEntry`:

- `id`, `title`, `body`.
- `authorIds`.
- `hashtags`.
- Optional per-entry password.
- `timestamp`.

### Chat

Chat is exported as two sibling top-level keys, `chatChannels` and `chatMessages`, not nested under a single `chat` object.

`ChatChannel` (in `chatChannels[]`):

- `id`, `name`, archived state/timestamps, created time.

`ChatMessage` (in `chatMessages[]`):

- `id`, `channelId`, `authorId`.
- `type`: text, image, file, reply, reaction.
- `content`, optional `replyToId`, reactions record, timestamp.

### Noteboards And Polls

`NoteboardEntry`:

- `id`, `memberId`, `authorId`, `content`, `timestamp`, optional `pinned`.

`MemberPoll`:

- `id`, `targetMemberId`, `question`, `options`, `createdBy`, `createdAt`.
- Optional `closedAt`, `hideVoterNames`.

Options contain `id`, `label`, and member-ID votes.

### Settings

`AppSettings` includes:

- Saved locations, custom moods, light mode, GPS/files flags.
- Language, notification flag, active palette, text scale.
- Sorting, front check, and noteboard preferences.

## Import/Interoperability Notes

Plural Star imports its own backup JSON, Simply Plural, and PluralKit. Its Simply Plural importer groups overlapping SP front rows into a single history entry with `memberIds`.

For OpenPlural, it argues for:

- Supporting tiered front roles (`primary`, `co_front`, `co_conscious`) rather than only a flat member list.
- Export-time asset dictionaries or a generic assets module.
- Category-selective export.
- Optional chat, boards (Plural Star's noteboards), polls, journal, moods, locations, and settings modules.
