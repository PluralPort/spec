# Ampersand

## Source Status

Sources:

- Repository: https://github.com/NyaomiDEV/Ampersand

Verified against Ampersand `main` at commit `75787996` on 2026-05-26.

Ampersand describes itself as an offline-first tracking and journaling app for plural systems. It is alpha software.

## Export Shape

Ampersand is a local Tauri/Vue app. For inter-app migration, the relevant export is JSON via `exportDatabaseToJSON()` in `src/lib/db/ioutils/json.ts`. This is the path Nao designed for third-party consumers — in the UI (`views/options/ImportExport.vue`) it lives under a `// --- THIRD PARTY` section, separate from Ampersand's own self-backup.

`exportDatabaseToJSON()` takes no arguments. It always writes every table and always embeds file payloads (avatars, covers, asset blobs) as Data URIs via `toDataURI`. There is no separate "with files" / "without files" mode. The export is streamed to disk with `json-stream-es` rather than buffered.

The top-level shape is:

```json
{
  "config": {
    "appConfig": {},
    "accessibilityConfig": {},
    "securityConfig": {}
  },
  "database": {
    "systems": [],
    "members": [],
    "boardMessages": [],
    "frontingEntries": [],
    "journalPosts": [],
    "reminders": [],
    "tags": [],
    "assets": [],
    "customFields": [],
    "notes": [],
    "filterQueries": []
  }
}
```

All eleven tables in `getTables()` are emitted. The canonical type is `DatabaseJSON` in `src/lib/db/ioutils/json_types.d.ts`. There is no top-level schema version field.

`importDatabaseFromJSON()` parses the file with `json-stream-es`, **clears every table before importing**, then runs `table.migrate(0)` per table after the stream completes. A failed import that aborts after the `clear()` call can leave the user with no data.

### Note on Asset.file

`Asset.file` is typed as required on both the runtime entity and `AssetJSON`, but the JSON exporter and importer are written defensively (`file: _data.file ? await toDataURI(_data.file) : undefined`). A malformed runtime asset can therefore round-trip as a record with no `file` despite the declared type.

### Sibling format: `.ampar` archive

For completeness — Ampersand's primary user-facing backup is a separate `.ampar` msgpack archive (`exportArchive()` in `src/lib/db/ioutils/archive.ts`), with a magic-versioned 10-byte header (`AMPAR\0\0\x01\0\0` for version 1) and inline binary file blobs via a `_meta` envelope. It's a self-format, not an interop format, and isn't documented further here. Worth knowing as a data point if OpenPlural pursues a unified archive container, alongside Prism's encrypted `.prism` JSON envelope.

## Records

### Systems

`System`:

- `uuid`, `name`, optional `description`.
- Optional `nameStyle` (`family`, `weight`, `italic`) for custom name typography.
- Optional `cover`/`image` files and `imageClip` shape (e.g. `arch`, `gem`, `heart`, `pixel-circle` — 34 literal values).
- Optional `parent` for nested systems/subsystems.
- Optional `color`.
- `isPinned`, `isArchived`, `viewInLists`.

### Members And Custom Fronts

`Member`:

- `uuid`, `system`, `name`.
- Optional `nameStyle` (`family`, `weight`, `italic`).
- Optional `pronouns`, `description`, `role`, `age` (number).
- Optional `image`/`cover` files and `imageClip` shape.
- Optional `color`.
- Optional `customFields` map of custom-field UUID to string value.
- `isPinned`, `isArchived`.
- `isCustomFront`.
- `tags` array of UUIDs.
- `dateCreated`.

Custom fronts use the same `Member` type with `isCustomFront: true`.

### Fronting Entries

`FrontingEntry`:

- `uuid`, `member` (single UUID).
- `startTime`, optional `endTime`.
- `isMainFronter`.
- `isLocked`.
- Optional `customStatus`.
- Optional `influencing` member.
- Optional `presence`: map of timestamp to numeric rating (exported as ISO-8601-keyed object).
- Optional `comment` — the author's own fronting note.
- Optional `comments`: array of `Comment` records written by other members.

Co-fronting is represented by simultaneous open/overlapping entries. One entry can be marked main fronter. Entries can also distinguish influencing members and per-time presence ratings.

### Tags

`Tag`:

- `uuid`, `name`, optional `description`.
- `type`: `member`, `journal`, or `asset`.
- Optional `color`.
- `isArchived`, `viewInLists`.

Membership is stored as UUID references on members, journal posts, or assets.

### Custom Fields

`CustomField`:

- `uuid`, `name`, `priority`.
- `default` (boolean): whether this field should be pre-populated on newly created members.

Per-member values live in `Member.customFields` as a UUID-to-string map. There is no field type enum — values are always strings.

### Journal Posts

`JournalPost`:

- `uuid`, `members`: array of member UUIDs (multi-author; may be empty).
- `date`, `title`, optional `subtitle`, `body`.
- Optional `cover` file (Data URI in export).
- `tags` array of UUIDs.
- `isPinned`, `isPrivate`.
- Optional `contentWarning`.
- Optional `comments`: array of `Comment` records.

Multi-author journal posts landed in `1d6875e3 chore: multiauthoring` (May 2026); older exports use a single optional `member` field instead.

### Board Messages And Polls

`BoardMessage`:

- `uuid`, `members`: array of member UUIDs (required; may be empty for a system-wide post).
- `title`, `body`, `date`.
- `isPinned`, `isArchived`.
- Optional `poll`.
- Optional `comments`: array of `Comment` records.

`Poll`:

- `entries`, `multipleChoice`.

`PollEntry`:

- `choice`, `votes`.

`Vote`:

- `member`, optional `reason`.

### Comments

`Comment` (used by `BoardMessage`, `JournalPost`, `FrontingEntry`):

- `member`: UUID of the author.
- `comment`: text body.
- `date`: ISO-8601 timestamp.
- Optional `replyTo`: another comment's `date`, used as a stable identifier for reply threading.

### Assets

`Asset`:

- `uuid`, `file` (required `File`, exported as Data URI), `friendlyName`, `tags` (UUID array).

### Reminders

`Reminder`:

- `uuid`, `active` (boolean).
- `title`, `message`.
- `trigger`: `"fronting"` fires when a listed member *starts* fronting. `"fronted"` fires when a listed member *stops* fronting. With `members` omitted, the trigger watches the fronting set as a whole — `"fronting"` fires when the fronter count goes up, `"fronted"` when it goes down (see `triggerReminders` in `src/lib/db/tables/reminders.ts`).
- Optional `members`: array of member UUIDs the reminder applies to.
- `delay`: milliseconds. Used as a notification schedule offset (the notification is scheduled `delay` ms after the trigger event), not as a fronting-duration threshold.

The older split between event reminders (`memberAdded`/`memberRemoved` triggers, filter queries) and periodic reminders (year/month/weekday calendar intervals) was replaced by this single shape in May 2026, at which point reminders also started being included in the JSON export.

### Notes

`Note`:

- `uuid`, `title`, `content`.
- `priority` (number).
- `isArchived`.

System-level notes. Distinct from journal posts (per-member dated entries).

### Filter Queries

`FilterQuery`:

- `uuid`, `name`, `query` (string).
- `type`: one of `members`, `systems`, `journal`, `frontHistory`, `messageBoard`, `assetManager`, `customFields`, `notes`, `tagManagement`.

Saved filter expressions for the corresponding list view.

## Imports

Ampersand has import support for:

- Its own JSON file (`importDatabaseFromJSON` in `ioutils/json.ts`) — the third-party migration path.
- Simply Plural.
- PluralKit.
- Octocon.
- Tupperbox.

The external import modules under `src/lib/db/external/` are useful references for conversion work even though this research focused on Ampersand's own export.

## Import/Interoperability Notes

Ampersand argues for:

- Complete JSON migration exports (one file, every table).
- File payloads always embedded as Data URIs (no separate "with files" mode).
- Custom fronts as member-like records.
- Front intervals with main/influence/presence metadata plus separate author-comment and other-commenter fields.
- Multi-author journal posts and board messages (member UUID arrays, not a single author).
- Tags scoped by target type.
- Per-record threaded comments (`Comment.replyTo` references a parent comment's `date` as a stable identifier).
- System and member name styling (font family, weight, italic).
- Reminders modeled as a single flat shape with a fronting-state trigger and millisecond delay.

Ampersand's JSON export has `config` and `database` keys but no top-level schema version field. OpenPlural should add an explicit version to its own format.

Polls attach to a board post, not the other way around — the boards and polls modules need a cross-reference path (post-v0.1).

> Source note: at the verified commit, `src/lib/db/ioutils/json_types.d.ts` declares `DatabaseJSON` using `Reminder`, `Tag`, `CustomField`, and `UUID` without importing them, so the canonical type doesn't fully compile in isolation. The intended shapes are still unambiguous from `entities.d.ts`. Worth knowing if you're code-genning anything off the type file directly.
