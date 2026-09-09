# Prism

## Source Status

Reference source is the Prism Flutter app. The strongest files for this research are:

- `lib/features/data_management/models/export_models.dart`
- `lib/features/data_management/services/data_export_service.dart`
- `lib/core/database/tables/*`
- `lib/features/boards/`

## Storage And Export Shape

Prism is a local-first Flutter app with structured database tables and an encrypted `.prism` export. The export model is named `V1Export`, but its comments describe the current encrypted envelope as the Prism 3/V3 export container.

Top-level export fields:

| Field | Meaning |
| --- | --- |
| `formatVersion` | Export format version, currently `1.0` in the model. |
| `version` | App/export version metadata. |
| `appName` | `Prism Plurality`. |
| `exportDate` | Export timestamp. |
| `totalRecords` | Export count summary. |
| `headmates` | Member/headmate profiles. |
| `frontSessions` | Per-member front intervals. |
| `sleepSessions` | Dedicated sleep intervals. |
| `conversations`, `messages` | Internal chat. |
| `polls`, `pollOptions` | Decision polls and embedded votes. |
| `systemSettings` | User/system app settings and profile metadata. |
| `habits`, `habitCompletions` | Habit tracker data. |
| Optional arrays | PluralKit sync state, groups, custom fields, notes, front comments, conversation categories, reminders, friends, media attachments, and legacy rescue data. |

Member board posts are a separate communication surface persisted in the database and synced via the local CRDT layer, but they aren't currently emitted in the `.prism` export envelope (no `memberBoardPosts` field on `V1Export`, and the export service doesn't inject the board-posts repository). The Simply Plural importer backfills them into the `member_board_posts` table directly.

## Records

### System Settings

Prism exports a single `systemSettings` record with system and app state:

- System name, description, sharing ID, avatar data, accent color, per-member accent colors.
- Terminology settings and custom terminology.
- Feature toggles for quick front, chat, boards, polls, habits, notes, sleep tracking, reminders, sync theme, navigation sync, and badges.
- Theme mode, brightness, style, font scale, font family.
- Front timing behavior, quick-switch threshold, fronting reminders.
- Chat log front behavior, chat badge preferences.
- PIN/biometric lock metadata and auto-lock delay.

### Headmates

`V1Headmate` fields include:

- `id`, `name`, `displayName`, `pronouns`, `emoji`, `age`, `birthday`, `notes`.
- Profile image as `profilePhotoData`.
- `isActive`, `isAdmin`, `createdAt`, `displayOrder`.
- Custom color flags and hex color.
- `parentSystemId`.
- PluralKit references: `pluralkitUuid`, `pluralkitId`, `proxyTagsJson`, `pluralkitSyncIgnored`.
- Markdown flag.

### Fronting

Prism now models fronting as per-member intervals:

- `id`, `startTime`, `endTime`, `headmateId`.
- `notes`, `confidence`, `quality`.
- `sessionType`: normal or sleep/HealthKit.
- PluralKit sync identifiers and legacy co-fronter/member ID JSON fields.
- `isHealthKitImport`.

Co-fronting is represented by overlapping interval rows, not by one canonical switch object. Legacy fields can preserve older grouped-session exports.

Front comments are separate records. Newer comments anchor to `targetTime` and optional `authorMemberId`; older comments may refer directly to a `sessionId`.

### Sleep

Sleep is also exported separately as `sleepSessions`:

- `id`, `startTime`, `endTime`, `quality`, `notes`, `isHealthKitImport`.

### Internal Communication

Prism has rich internal chat:

- `conversations`: title, emoji, direct-message flag, creator, participants, read timestamps, archive/mute state, category, description, display order.
- `messages`: content, timestamp, system-message flag, edit time, author, conversation, reactions, reply metadata.
- `mediaAttachments`: message attachment metadata including encrypted media IDs, hashes, mime type, size, dimensions, duration, blurhash, waveform, thumbnail, and deletion flag.

### Message Boards

Separate from chat. The `member_board_posts` table holds short async messages between headmates. The feature is gated by a `boards` toggle in system settings.

`MemberBoardPost` fields:

- `id`: UUID primary key.
- `targetMemberId`: recipient member; null for system-wide public posts.
- `authorId`: writer; null only for legacy SP imports where `writtenBy` was unknown.
- `audience`: exactly `'public'` or `'private'`. Public posts appear on the system-wide Public timeline and the target member's profile section. Private posts appear only in the Inbox of whoever is currently fronting as `targetMemberId`.
- `title`: optional.
- `body`: required, non-empty.
- `createdAt`: immutable creation timestamp.
- `writtenAt`: user-facing post timestamp; equals `createdAt` for native posts, equals SP `writtenAt` for SP imports.
- `editedAt`: nullable; non-null after first edit.
- `isDeleted`: soft-delete tombstone (always emitted to the CRDT engine).

No replies, threading, reactions, or attachments. Permissions: only the author can edit; the author, profile-owner, or any admin can delete.

Board posts are persisted and synced via the local CRDT adapter but are **not** included in the `.prism` export envelope at present — the export service injects no board-posts repository and `V1Export` has no corresponding field. Cross-device sharing currently relies on the live sync layer, not on file export.

### Polls

Prism exports polls with options and embedded votes:

- Polls: question, description, anonymous/multiple vote flags, closed flag, expiry, created time.
- Options: text, sort order, other-option flag, color, votes.
- Votes: member, voted-at timestamp, optional response text.

### Groups And Custom Fields

Groups are hierarchical:

- `memberGroups`: name, description, color, emoji, display order, parent group ID.
- `memberGroupEntries`: group/member join records.

Custom fields are normalized:

- `customFields`: name, field type, date precision, display order, created time.
- `customFieldValues`: field ID, member ID, value.

### Notes, Habits, Reminders, Friends

Additional optional modules:

- `notes`: title, body, color, member ID, date, created/modified times.
- `habits`: name, frequency, recurrence configuration, reminders, assigned member, privacy, counters.
- `habitCompletions`: habit ID, completed time, member, notes, fronting flag, rating.
- `reminders`: name, message, trigger, interval/delay/time fields, active flag.
- `friends`: local encrypted sharing metadata, peer keys, granted/offered scopes, verification state.

## Import/Interoperability Notes

Prism has Simply Plural and PluralKit importers. Its current shape is broader than most other tools: fronting, members, groups, custom fields, chat, polls, habits, reminders, media, and local sharing all need either PluralPort core support or extension modules.

For PluralPort, Prism argues for:

- Overlapping per-member front intervals as a first-class model.
- Front comments anchored by time, not only by session ID.
- Separate asset records instead of embedding images everywhere.
- Optional modules for chat, boards, polls, habits, reminders, and friend sharing.
- A boards module distinct from chat — the unit is a member-targeted post, not a thread message, and Prism's `member_board_posts` shape (title, audience, target, written_at) doesn't fit ChatMessage cleanly. The fact that boards are sync-only today (not in `.prism` export) also means their shape needs to be explicit at the PluralPort layer, since the source app currently has no portable serialization to map from.
