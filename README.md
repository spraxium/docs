# spraxium/docs

The [Spraxium site](https://spraxium.com/en/guide) builds its guides, release notes, roadmap and coding-agent skills from this repository.

- `versions.json` selects the current version and lists the archived versions.
- `versions/0.10.0/{en,pt,es}/` contains the active guides.
- `versions/0.9.0/{en,pt,es}/` preserves the older docs.
- `changelogs/` contains release notes. Place translations with the same filename under `changelogs/pt/` or `changelogs/es/`.
- `roadmap/data.json` contains the project status.
- `skills/manifest.json` defines the skills catalog. Keep each served file under `skills/<slug>/SKILL.md` and match its frontmatter name to the slug. The [HTTP-to-RPC migration skill](https://github.com/spraxium/rpc/blob/main/SKILL.md) lives in the RPC repository.

A merge to the docs main branch triggers the website content sync. The site build publishes the catalog, the Markdown and [the skills index](https://spraxium.com/skills/index.json).

See [CONTRIBUTING.md](CONTRIBUTING.md) for writing and review guidelines.
