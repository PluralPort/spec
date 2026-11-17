# OpenSelves

## Source Status

Sources:

- Repository: https://github.com/FreckleQueens/OpenSelves

## Storage And Sync Shape

OpenSelves is a smaller offline-first app using:

- Browser IndexedDB on the client.
- Drizzle/PostgreSQL schema in `common/src/db/schema`.
- Server sync endpoints based on operation logs.

No general export file was found in the inspected source. The portable shape is therefore inferred from database and sync DTOs.

## Records

### Users

`users`:

- `id`, `email`, `passwordHash`.
- Created/updated timestamps.

There is no separate system profile table in the checked schema snapshot.

### Members

`members`:

- Composite primary key: `userId`, `id`.
- `name`, `pronouns`, `description`.
- Optional `image`.
- `isArchived`, optional `archivedReason`.
- `createdAt`, `updatedAt`.

IDs are generated with CUID2.

### Fronts

`fronts`:

- Composite primary key: `userId`, `id`.
- `memberId` foreign key.
- `startedAt`, optional `endedAt`.
- Optional `note`.
- `createdAt`, `updatedAt`.

This is per-member interval fronting. There is no many-to-many front record; co-fronting would be represented as overlapping rows if supported by UI/workflows.

### Logs

`logs` supports offline sync:

- Composite primary key: `userId`, `id`.
- Optional `memberId` or `frontId`.
- `operationType`: `create`, `update`, `delete`.
- `data` JSON for create/update.
- `deletedId` for deletes.
- `executedAt`, `pushedAt`.

Checks enforce that create/update logs reference either one member or one front and include data, while delete logs reference a deleted ID and no data.

### Sync DTOs

Client pushes pending logs to `/sync/push`. Create/update DTOs validate:

Members:

- `name`, `pronouns`, `description`.
- Optional image URL or data URI.
- `isArchived`, `archivedReason`.

Fronts:

- `memberId`, `startedAt`, `endedAt`, `note`.

Client pulls logs since a timestamp from `/sync/pull` and applies them to IndexedDB.

## Import/Interoperability Notes

OpenSelves is useful as a minimal target:

- Member profile fields are required and simple.
- Fronting is one member per interval.
- There are no groups, custom fields, journals, polls, chat, or system profile in the inspected schema.
- PluralPort should allow importing a rich file into a simpler app with a clear `losses` report, not require every module to be implemented.
