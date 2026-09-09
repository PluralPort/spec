# Tupperbox

## Source Status

Tupperbox is a closed-source Discord proxy bot. The shape documented
here is reverse-engineered from real `tul!export` output, not from
source.

There is no public API for retrieving the export outside Discord; the
file must be exported from the bot, which delivers it as a JSON
attachment to the invoking user.

## Storage And API Shape

The export is a single JSON object with two top-level arrays:

```json
{
  "tuppers": [],
  "groups": []
}
```

There is no system metadata, no fronting log, no custom fields, no
privacy model, no journals, no chat. Tupperbox is a proxy bot, not a
system tracker.

## Records

### Tuppers (member-equivalents)

```json
{
  "id": 189220014,
  "name": "Alex",
  "nick": "Alex (they/them)",
  "description": "Bio text.",
  "avatar_url": "https://cdn.tupperbox.app/pfp/<user_id>/<file>.webp",
  "avatar": "<file>",
  "banner": null,
  "brackets": ["a:", ""],
  "show_brackets": false,
  "tag": null,
  "birthday": "2026-05-11T00:00:00.000Z",
  "posts": 1,
  "last_used": "2026-05-11T03:28:47.414Z",
  "created_at": "2026-05-11T00:00:00.000Z",
  "group_id": 2574074
}
```

Notable shape details:

- `id` is an integer, not a UUID or short string.
- `brackets` is a two-element array of prefix/suffix; either can be the
  empty string. This is Tupperbox's proxy-match shape.
- `birthday` is a full ISO 8601 timestamp with a year component even
  when the user only specified month/day. Year-less storage isn't
  preserved.
- `tag` is a per-tupper system-tag suffix used at proxy time, not a
  taxonomy tag.
- `posts` and `last_used` are runtime usage metadata.
- `avatar` is the CDN filename without the URL; `avatar_url` includes
  the host. They're redundant for portability purposes.
- `group_id` is a single integer; tuppers belong to at most one group.

### Groups

```json
{
  "id": 2574074,
  "name": "Core members",
  "description": "Optional group description.",
  "avatar": null,
  "tag": null
}
```

Groups are flat (no nesting) and do not list their members; the edge
lives on the tupper's `group_id` field. This is the inverse of PluralKit
groups, which carry a list of member IDs.

## Import/Interoperability Notes

PluralPort mapping is straightforward:

- `tuppers[].id` → `SourceRef(app: "tupperbox", collection: "tuppers")`,
  stringified.
- `tuppers[].name` → `Member.name`.
- `tuppers[].nick` → `Member.display_name`.
- `tuppers[].description` → `Member.description`.
- `tuppers[].avatar_url` → `Asset(kind: "avatar")` +
  `Member.avatar_asset_id`.
- `tuppers[].brackets` → `Member.proxy_tags[]`. The two-element array
  maps cleanly: `{prefix: brackets[0] or null, suffix: brackets[1] or
  null}`.
- `tuppers[].show_brackets` → `extensions.tupperbox.show_brackets`.
- `tuppers[].birthday` → `Member.birthday` with `precision: "day"`,
  `year_visible: true`. Tupperbox doesn't preserve year-less, so
  inferring `year_visible: false` requires guessing from a sentinel
  year, probably not worth doing.
- `tuppers[].tag` → `extensions.tupperbox.tag` (it's a proxy-time
  formatting concern, not a taxonomy tag).
- `tuppers[].banner` → `Asset(kind: "banner")` + `Member.banner_asset_id`
  when present.
- `tuppers[].posts`, `tuppers[].last_used` →
  `extensions.tupperbox.usage_stats` (or drop on export; they're not
  portable identity).
- `tuppers[].created_at` → `Member.created_at`.
- `tuppers[].group_id` → `GroupMembership[]` with one row.
- `groups[].id`, `name`, `description` → `Group` direct.
- `groups[].avatar`, `groups[].tag` → `extensions.tupperbox.*`.

The PluralKit-style `proxy_tags` mapping is the most useful piece. Any
PluralPort importer that already supports PluralKit proxy tags will
correctly render Tupperbox-origin members' proxies with no extra work.

## What Tupperbox Doesn't Have

For an exporter or importer that's targeting the full PluralPort shape:

- **No fronting.** Tupperbox doesn't model who's fronting; it's a proxy
  bot. Front history must come from elsewhere (PluralKit, Sheaf, etc.).
- **No system profile.** Producers should populate `System.name`/etc.
  with sensible defaults or prompt the user.
- **No pronouns.** Some users encode them in `nick` or `description`.
  Heuristic extraction is lossy and probably not worth attempting at
  the converter layer.
- **No privacy model.** Importers should default Tupperbox-origin
  records to the strictest privacy bucket the target app supports, on
  the theory that Tupperbox users haven't expressed a preference.
- **No custom fields, journals, or chat.** Nothing to map.

This makes Tupperbox the easiest researched producer to support: every
field is either covered by core records or fits in extensions, with no
shape ambiguity.
