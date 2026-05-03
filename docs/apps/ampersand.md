# Ampersand

## Source Status

Sources:

- Repository: https://github.com/NyaomiDEV/Ampersand

Ampersand describes itself as an offline-first tracking and journaling app for plural systems. It is alpha software.

## Storage And Export Shape

Ampersand is a local Tauri/Vue app. It stores data in local tables implemented by its `ShittyTable` abstraction and exports/imports a JSON backup.

The export route is `exportDatabaseToJSON(withFiles: boolean)`. It writes:

```json
{
  "config": {
    "appConfig": {},
    "accessibilityConfig": {},
    "securityConfig": {}
  },
  "database": {
    "boardMessages": [],
    "frontingEntries": [],
    "journalPosts": [],
    "members": [],
    "systems": [],
    "tags": [],
    "assets": [],
    "customFields": []
  }
}
```

Reminders exist in the model but are intentionally skipped by the JSON exporter at the time of research. File data is included only when `withFiles` is true, encoded as Data URIs.

## Records

### Systems

`System`:

- `uuid`, `name`, optional `description`.
- Optional cover/image files and image clipping shape.
- Optional `parent` for nested systems/subsystems.
- Optional `color`.
- `isPinned`, `isArchived`, `viewInLists`.

### Members And Custom Fronts

`Member`:

- `uuid`, `system`, `name`.
- Optional `pronouns`, `description`, `role`.
- Optional image/cover files and image clipping shape.
- Optional `color`.
- Optional `customFields` map of custom field UUID to string.
- `isPinned`, `isArchived`.
- `isCustomFront`.
- `tags` array.
- `dateCreated`.

Custom fronts use the same `Member` type with `isCustomFront: true`.

### Fronting Entries

`FrontingEntry`:

- `uuid`, `member`.
- `startTime`, optional `endTime`.
- `isMainFronter`.
- `isLocked`.
- Optional `customStatus`.
- Optional `influencing` member.
- Optional `presence`: map of timestamp to numeric rating.
- Optional `comment`.

Co-fronting is represented by simultaneous open/overlapping entries. One entry can be marked main fronter. Entries can distinguish influencing members and per-time presence ratings.

### Tags

`Tag`:

- `uuid`, `name`, optional `description`.
- `type`: `member`, `journal`, or `asset`.
- Optional `color`.
- `isArchived`, `viewInLists`.

Membership is stored by references on members, journal posts, or assets.

### Custom Fields

`CustomField`:

- `uuid`, `name`, `priority`, `default`.

Member values are stored on `Member.customFields` as a map. The inspected type doesn't include a field type enum.

### Journal Posts

`JournalPost`:

- `uuid`, optional `member`.
- `date`, `title`, optional `subtitle`, `body`.
- Optional cover file.
- `tags`.
- `isPinned`, `isPrivate`.
- Optional `contentWarning`.

### Board Messages And Polls

`BoardMessage`:

- `uuid`, optional member.
- `title`, `body`, `date`.
- `isPinned`, `isArchived`.
- Optional `poll`.

`Poll`:

- `entries`, `multipleChoice`.

`PollEntry`:

- `choice`, `votes`.

`Vote`:

- `member`, optional reason.

### Assets

`Asset`:

- `uuid`, file, friendly name, tags.

Export encodes file content as a Data URI when exporting with files.

### Reminders

`Reminder` exists but isn't included in current JSON export.

Two reminder shapes exist:

- Event reminder: triggering event (`memberAdded` or `memberRemoved`), optional filter query, delay hours/minutes.
- Periodic reminder: calendar interval fields such as year, month, day, weekday, hour, minute.

## Imports

Ampersand has import support for:

- Its own JSON backup.
- Older backup formats.
- Simply Plural.
- PluralKit.
- Octocon.
- Tupperbox.

The external import modules are useful references for conversion work even though this research focused on Ampersand's own export.

## Import/Interoperability Notes

Ampersand argues for:

- Offline-first complete JSON backups.
- File payloads as optional Data URI fields.
- Custom fronts as member-like records.
- Front intervals with main/front influence/presence metadata.
- Tags scoped by target type.
- Explicitly versioned export metadata. Ampersand's current backup has config and database keys but no obvious top-level schema version, which OpenPlural should add.
- Polls that can attach to a board post, not just stand alone — the boards and polls modules need a cross-reference path (post-v0.1).
