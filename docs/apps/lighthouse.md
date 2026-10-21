# Lighthouse

## Source Status

Sources:

- Repository: https://github.com/team-crystalline/Lighthouse
- Website: https://www.writelighthouse.com/

## Storage And API Shape

Lighthouse is a server-backed Node/Express app using PostgreSQL. Sensitive text is generally stored base64-encoded or AES-encrypted depending on table/field.

The app has:

- Browser routes for systems, alters, journals, forums, worksheets, safety plans, imports, and settings.
- A token API for reading systems, members, and journals.
- A user export route that creates a ZIP of CSV files.

## Export Shape

`GET /api/user/export/:id` builds a ZIP with CSV files (see [`routes/api.js`](https://github.com/team-crystalline/Lighthouse/blob/main/routes/api.js)):

- `userInfo.csv`
- `systems.csv`
- `alters.csv`
- `journals.csv`
- `posts.csv`
- `bdaPlan.csv`
- `innerWorlds.csv`
- `rules.csv`
- `wishlist.csv`
- `communalJournals.csv`
- `categories.csv`
- `threads.csv`
- `threadPosts.csv`

The exporter decodes base64 profile fields and decrypts AES fields before writing CSV.

The export covers more than the alter/journal core: `bdaPlan` (before/during/after plans), `innerWorlds`, `rules`, `wishlist`, communal journals, and the forum surface (`categories`, `threads`, `threadPosts`) all ship in the same ZIP. Most of these have no PluralPort module today and would land in `extensions` for a converter built against this export. The `safetyplans` table appears in the schema but isn't part of the export route.

## Public Token API

Token API endpoints include:

- `GET /api/members/:id`
- `GET /api/systems/:id`
- `GET /api/journals/:id`

Tokens can grant read/write and per-area flags for alters, systems, and journals.

## Records

### User Settings

`users` includes:

- User ID, username, email/auth fields.
- Terminology: alter/system/subsystem/inner world/plural terms.
- Skin/theme, language, text size, font.
- Feature toggles for inner worlds, worksheets, glossary.
- Simply Plural and PluralKit IDs/tokens.

### Systems

`systems`:

- `sys_id`, `user_id`, `sys_alias`, `icon`, `subsys_id`, `description`.

Subsystems are represented by systems with a `subsys_id` parent link.

### Alters

`alters` has a very rich profile table:

- `alt_id`, `sys_id`, `name`.
- Triggers, age text, likes, dislikes.
- Job/role, safe place, wants, accommodations, notes.
- Image URL or image blob and MIME type.
- Type, pronouns, birthday, first noted.
- Gender, sexuality, source.
- Front tells, relationships.
- Archived flag.
- Hobbies, appearance.
- Color, outline settings, signature, species, nickname.
- Simply Plural ID and PluralKit ID.
- Up to five subsystem IDs in the checked-in class.
- Header image blob/MIME type.

### Journals

`journals`:

- `j_id`, `alt_id`, `sys_id`, privacy, password, skin, image/skin metadata.

`posts`:

- `p_id`, `j_id`, `created_on`, `title`, `body`, pinned flag, feeling.

Lighthouse also has `comm_posts` for communal journal posts:

- `id`, `u_id`, `created_on`, `title`, `body`, pinned flag, `system_id`, feeling.

### Safety, Inner Worlds, Rules

Additional exported tables:

- `safetyplans`: symptoms, safe people, distractions, keep-safe plans, get-help plans, grounding.
- `bda_plans`: before/during/after plan fields, alias, timestamp, active flag.
- `inner_worlds`: key/value pairs.
- `sys_rules`: rule and created timestamp.
- `wishlist`.

### Forums / Threads

Lighthouse includes community/forum data:

- `categories`, `forums`, `threads`, `thread_posts`.

Thread records can be authored by an alter.

### Fronting

There's a `classes/frontEntry.js` file, but it's a stub and no active fronting table was found in the checked-in SQL schema. Fronting is mentioned as an intended feature, not a reliable current export surface in this source snapshot.

## Imports

Lighthouse can import member/alters from:

- Simply Plural token API: imports member name, pronouns, avatar URL, color, description.
- PluralKit API: imports display/name, pronouns, avatar URL, birthday, PluralKit ID.

These importers are narrow compared to Lighthouse's internal alter profile.

## Import/Interoperability Notes

Lighthouse argues for:

- CSV import/export support in PluralPort tooling, even if the canonical spec is JSON.
- Rich member profile extension fields for triggers, roles, accommodations, source, front tells, relationships, etc.
- Journal import/export support.
- A clear way to mark unsupported or not-yet-shipped fronting models.
