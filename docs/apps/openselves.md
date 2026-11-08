# OpenSelves

OpenSelves Version: ^0.7.0

## Source Status

Sources:

- Repository: https://codeberg.org/FreckleQueens/OpenSelves/

## Storage And Sync Shape

OpenSelves is a smaller offline-first app using:

- [The Willow data model](https://willowprotocol.org/specs/data-model/#data_model).
- Browser IndexedDB on the client.
- Drizzle/PostgreSQL schema in `server/src/db/schema`.
- Server sync endpoints for pushing and pulling Willow entries.
- CUID2 to generate IDs.

Each datapoint (i.e. a member's name, a member's description...) has its own entry. Entry paths follow the format `/{collection}/{record_id}/{field_name}`, for example `/members/w8emnbh431o3hvkeew4vbp01/name`.

Paths and payloads are end-to-end encrypted.

## Records

### Systems

Currently, only a single system is supported per-profile. There can be many profiles but there is no interaction between profiles.Profile names are local-only; they are not synced accross clients.

### Members

`members`:

- `id`.
- `name`.
- Optional `pronouns`, optional `description`, optional `color`.
- Optional `image`.
- Optional `isArchived` (default: `false`), optional `archivedReason`.
- `createdAt`.

### Fronts

`fronts`:

- `id`.
- Optional `memberId` foreign key (default: `null`).
- `startedAt`, optional `endedAt` (default: `null`).
- Optional `note`.
- `createdAt`.

This is per-member interval fronting. There is no many-to-many front record; co-fronting would be represented as overlapping rows if supported by UI/workflows.

### Sync

Client pushes pending entries to `/sync/push` and saves current timestamp on success.

Client pulls new entries resuming from a server-provided timestamp from `/sync/pull` and saves the new entries to IndexedDB as well as the new timestamp.

In both directions, the data format is a binary octet stream that is the encoding of the entries in the [Drop format](https://willowprotocol.org/specs/drop-format/#willow_drop_format) (see "Storage And Sync Shape").

## Import/Interoperability Notes

OpenSelves is useful as a minimal target:

- Member profile fields are simple, only `id`, `name` and `createdAt` are required.
- Fronting is one member per interval.
- There are no groups, custom fields, journals, polls, chat, or system profile (besides profile names).
- OpenPlural should allow importing a rich file into a simpler app with a clear `losses` report, not require every module to be implemented.
