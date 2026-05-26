# Ampersand

## Source Status

- Repository: https://github.com/NyaomiDEV/Ampersand

Ampersand describes itself as an offline-first tracking and journaling app for plural systems. It is alpha software.

## Export Shape

**Ampersand has no production interoperable export today.**

The only export the shipped UI offers is an `.ampar` msgpack archive — a self-backup for round-tripping data back into Ampersand, not an inter-app interchange format. When Ampersand ships a portable export aimed at other apps, we'll document its shape here.

## Conceptual Surface

Even without a portable export to consume, the concepts Ampersand models inform spec design. At a high level Ampersand has:

- Systems, with nesting (subsystems via a parent pointer).
- Members, with custom fronts represented as member-like records.
- Per-member fronting intervals, with co-fronting via overlapping intervals, a main-fronter flag, optional influencing member, optional per-entry custom status, and per-time presence ratings.
- Typed tags scoped to members, journals, or assets.
- Custom fields with per-member string values (no type enum).
- Journal posts.
- Board messages with optional polls.
- Assets with friendly names and tags.

Per-record field shapes are out of scope here until a portable export ships.

## Imports

Ampersand's import surface is the more relevant interop story today — it's a migration destination from several apps:

- Simply Plural.
- PluralKit.
- Octocon.
- Tupperbox.

## Notes For OpenPlural

- Don't treat the `.ampar` archive as an interop contract — it's a self-format.
- The import-from list above is the real touchpoint: anything OpenPlural ships that can produce a Simply Plural / Octocon / PluralKit / Tupperbox export becomes importable into Ampersand by extension.
