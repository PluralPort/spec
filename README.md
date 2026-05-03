# OpenPlural Research Notes

**Site**: <https://skylartaylor.github.io/openplural/>

OpenPlural is a research/proposal workspace for a portable plurality-system data format.

The initial goal is to compare existing app data shapes and identify a smallest useful common model that app developers can export/import once instead of writing pairwise converters for every other app.

## Site

The full site is at <https://skylartaylor.github.io/openplural/> (built from the `.md` files in this repo via GitHub Pages):

- [Home](https://skylartaylor.github.io/openplural/) — overview, who's onboard, links
- [Apps](https://skylartaylor.github.io/openplural/apps.html) — feature matrix and per-app summaries
- [Spec hub](https://skylartaylor.github.io/openplural/spec.html) — conventions, envelope, shared fragments
  - [Records](https://skylartaylor.github.io/openplural/spec-records.html) — field tables for 17 core records
  - [Fronting](https://skylartaylor.github.io/openplural/spec-fronting.html) — periods, events, comments, assignments
  - [Modules & contract](https://skylartaylor.github.io/openplural/spec-modules.html) — chat, boards, optional modules, importer contract
- [Adoption guide](https://skylartaylor.github.io/openplural/adopt.html) — Prism + Sheaf mapping tables, maintainer guidance
- [Proposal](https://skylartaylor.github.io/openplural/proposal.html) — the v0.1 proposal as a single page

## Reference

Source markdown for the same content (rendered on GitHub):

- [Feature matrix](docs/feature-matrix.md)
- [OpenPlural v0.1 proposal](docs/openplural-proposal.md)
- Per-page sources at the repo root: `index.md`, `apps.md`, `spec.md`, `spec-records.md`, `spec-fronting.md`, `spec-modules.md`, `adopt.md`

## App Research

- [Prism](docs/apps/prism.md)
- [Sheaf](docs/apps/sheaf.md)
- [PluralSpace](docs/apps/pluralspace.md)
- [Simply Plural](docs/apps/simply-plural.md)
- [PluralKit](docs/apps/pluralkit.md)
- [Octocon](docs/apps/octocon.md)
- [Plural Star](docs/apps/plural-star.md)
- [Lighthouse](docs/apps/lighthouse.md)
- [OpenSelves](docs/apps/openselves.md)
- [Ampersand](docs/apps/ampersand.md)

## Scope Notes

This is source-backed where possible. PluralSpace is the one exception: its server is closed-source, so its research is based on inspected GDPR exports rather than source.

This repo intentionally starts with documentation only. A JSON Schema, fixture set, conformance tests, and reference converters are natural next steps once the model is agreed.

## Contributing

Issues and PRs are welcome — especially research corrections from app maintainers, missing fields, schema inconsistencies, and shape proposals for future modules. `docs/openplural-proposal.md` is the working document for v0.1.

## License

MIT — see [LICENSE](LICENSE).
