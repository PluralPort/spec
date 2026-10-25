# Ampersand

## Source Status

- Repository: https://github.com/NyaomiDEV/Ampersand
- Ampersand source snapshot inspected: `38101d87e500dd21905f4deb2496f1b1b0ca9a48` (2026-08-11).
- PluralPort converter source: https://github.com/PluralSpace/PluralPort, snapshot `7df6357b7f4ee027e50431cf67540ff5715f4a41` (2026-07-16).

Ampersand describes itself as an offline-first tracking and journaling app for plural systems. It is alpha software.

## Export Shape

Ampersand now exposes two user-facing export paths:

- An `.ampar` msgpack archive for native self-backup and restore.
- A JSON export named `ampersand-export-YYYY-MM-DD.json`, with top-level `revision`, `config`, and streamed `database` collections.

The JSON is still Ampersand's native shape rather than a direct OpenPlural export, but it is now an interoperable source in practice: PluralPort ships an in-browser Ampersand JSON to OpenPlural v0.1 converter.

The JSON exporter serializes profile and attachment blobs as data URIs, including system/member images and covers, journal covers, and asset files. It also includes app, accessibility, and security configuration; PluralPort deliberately excludes `config` because it contains device settings and the app-lock password hash rather than portable system data.

## Conceptual Surface

The current source and PluralPort converter expose these concepts:

- Systems, with nesting (subsystems via a parent pointer).
- Members, with custom fronts represented as member-like records.
- Per-member fronting intervals, with co-fronting via overlapping intervals, a main-fronter flag, optional influencing member, optional per-entry custom status, and per-time presence ratings.
- Typed tags scoped to members, journals, or assets.
- Custom fields with per-member string values (no type enum).
- Journal posts.
- Board messages with optional polls.
- Assets with friendly names and tags.

PluralPort currently maps systems, members and custom fronts, front history, tags, custom fields, journals and sticky notes, board posts and replies, polls, and inline assets. Reminders and saved filter queries are retained under `extensions.ampersand`; app/device configuration is omitted intentionally.

## Imports

Ampersand's import surface is the more relevant interop story today — it's a migration destination from several apps:

- Simply Plural.
- PluralKit.
- Octocon.
- Tupperbox.

## Notes For OpenPlural

- Don't treat the `.ampar` archive as an interop contract; use the JSON export when building a converter.
- Inline Ampersand images belong in `Asset.data_uri`. PluralPort initially placed them in `uri`, then corrected the converter so self-contained bytes are not mistaken for fragile external references.
- PluralPort is a converter implementation, not proof that Ampersand itself natively imports or exports OpenPlural.
- The import-from list above is the real touchpoint: anything OpenPlural ships that can produce a Simply Plural / Octocon / PluralKit / Tupperbox export becomes importable into Ampersand by extension.
