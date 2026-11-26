---
name: spraxium-cli
description: "Scaffold a Spraxium project, generate feature files, and use dev, build and version-switch commands for the right release."
metadata:
  framework-version: "0.10.0"
---

# Spraxium CLI

## Check the project and command before running it

Inspect packageManager, manifest, lockfile and the installed CLI release. Run its documented help to confirm templates, flags and schematics. In a new approved directory start with the official TypeScript template and read the closest example. Keep the user's package manager. Reuse the expected `src/main.ts` and module layout and use placeholders in `.env.example`. In an existing repository, avoid creating a second scaffolding over its tracked files.

~~~bash
pnpm dlx @spraxium/cli@0.10.0 new my-bot
pnpm exec spraxium generate --help
~~~

## Work in the actual bot package

Run `generate` from the bot package, inspect the changed files and matching module metadata, and check imports before continuing. A generator gives you the outline of a feature. Implement the small part you need using an example and the exports of the installed version. Use the existing dev command for rapid iteration and run the production build script before starting its output. Check the database helper and generated files before using it on an existing project with its own migrations.

## Switch versions without losing a reproducible install

When the CLI release supports `use`, simulate the switch first. A channel target such as `next` or `stable` resolves to a specific release. Use an exact version to reproduce a report. The switch updates the framework dependency group in the selected project and verifies the resulting install. RPC packages belong to a separate release and are moved explicitly. Keep a working code revision and lockfile before trying a preview, and verify working Discord features afterwards.

~~~bash
pnpm dlx @spraxium/cli@next use next --dry-run
~~~


## References

- [cli-overview](https://spraxium.com/en/guide/cli-overview)
- [cli-new](https://spraxium.com/en/guide/cli-new)
- [cli-generate](https://spraxium.com/en/guide/cli-generate)
- [cli-dev](https://spraxium.com/en/guide/cli-dev)
- [cli-database](https://spraxium.com/en/guide/cli-database)
- [cli-use](https://spraxium.com/en/guide/cli-use)
- [packages/cli/src/commands/new.command.ts](https://github.com/spraxium/spraxium/blob/main/packages/cli/src/commands/new.command.ts)
- [packages/cli/src/commands/generate.command.ts](https://github.com/spraxium/spraxium/blob/main/packages/cli/src/commands/generate.command.ts)
- [packages/cli/src/commands/use.command.ts](https://github.com/spraxium/spraxium/blob/main/packages/cli/src/commands/use.command.ts)
- [templates/default/src/main.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/main.ts)
