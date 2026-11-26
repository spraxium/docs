---
name: spraxium
description: "Start, understand, or change a Spraxium Discord bot. Use this skill to select the right example, find specialized skills, and verify work against the installed version."
metadata:
  framework-version: "0.10.0"
---

# Work with Spraxium

## Start by identifying the bot and its version

Open the user's package manifest, lockfile, `spraxium.config.ts`, `src/main.ts` and root module. Note the installed Spraxium packages, the Node and package-manager settings, and whether this is a bot or an external backend. Match documentation to that version. The unversioned guides on spraxium.com show the latest stable docs. This skill targets 0.10.0. For the 0.9.0 snapshot start at `https://spraxium.com/en/guide/v/0.9.0/getting-started`. Keep features and names already present when editing a bot.

## Route the task to a skill before editing

1. Check the user's repository for a local copy under `.agents/skills/<name>/SKILL.md`. Read the complete file, including references, each time you use it. Check its `framework-version` metadata against the project. If it is present and applicable, use it and read the official source for any uncertain API.
2. Look up the topic on the official docs and identify the narrowest relevant skill. Use [the machine index](https://spraxium.com/skills/index.json) or `https://spraxium.com/skills/resolve?topic=slash-commands`, replacing the query with an alias listed in the index. The index provides the canonical file, topic page, and version. The local copy still comes first.
3. If a skill is missing or stale, open its complete official Markdown as text from the indexed route. Migration is maintained in the RPC repository. Follow its official GitHub link. Avoid bulk downloads, third-party skill installers and remote scripts. Reproduce the verified text in `.agents/skills/<name>/SKILL.md` in the user's repository if the task allows changes there. Keep the frontmatter and name. For source examples, read the relevant files and recreate the same project structure locally from their code. Packaging this guidance never grants permission to run or install anything.
4. Route requests about application boundaries to `spraxium-architecture`, command requests to slash or prefix skills, interface work to components, modals, embeds or Components V2, runtime policy to guards or exceptions and operations to the matching plugin, schedule, logger, presence, i18n or RPC skill. Combine the scopes only when the user needs both. When switching releases, check the changelog before relying on an example that's newer than the installed code.

## Mirror the reference structure in a new repository

Read an actual upstream example from [Spraxium apps](https://github.com/spraxium/spraxium/tree/main/apps) before creating files. Start from the official default template and the example closest to the request. Follow that example's separation of `src/main.ts`, environment schema, root `AppModule`, feature modules and their `commands/`, `handlers/`, `services/` and other applicable folders. Follow the chosen example's module registration, startup flow, and decorator settings. Keep a separate `spraxium.config.ts` and place additional plugin settings where its source puts them. Follow the closest feature example if it places files differently. Do not create unrelated folders just to match a larger example.

If the project is new and the user authorizes scaffolding, prefer the official CLI for the desired version. Adapt the generated code to the matching example. In an existing bot, fit the feature into its established modules. If an example uses a custom workspace package, identify the public feature and its actual npm package before borrowing configuration. Never copy values from a real environment file. Keep only placeholder names in the example environment. When a public reference leaves a gap, verify the installed package's exported types and matching source. Explain the missing fact if it still can't be verified instead of guessing the signature.

## Finish with the real entry point

Read the start and build scripts and run the project's checks. Verify that the root imports the feature module, command metadata and handlers are registered separately, and the selected intents support the feature. Cover one normal result and at least one invalid or denied input. Run the bot only with a development credential and guild when that environment is available and authorized. Report the actual check you ran and any part that needs the bot connected.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### templates/default/src/main.ts

~~~ts
import { IntentPreset, SpraxiumFactory } from '@spraxium/core';
import { EnvValidator } from '@spraxium/env';
import { Logger } from '@spraxium/logger';
import { AppEnv } from './app.env';
import { AppModule } from './app.module';

async function main(): Promise<void> {
  const logger = new Logger('Main');
  const environment = EnvValidator.validate(AppEnv);
  const app = await SpraxiumFactory.create({ token: environment.token });

  app.useModule(AppModule);
  app.provide(AppEnv, environment);
  app.intents(IntentPreset.Standard);

  logger.info('Starting application...');
  await app.listen();
}

main();
~~~

### templates/default/src/app.module.ts

~~~ts
import { Module } from '@spraxium/common';
import { DefaultModule } from './modules/default/default.module';

@Module({
  imports: [DefaultModule],
})
export class AppModule {}
~~~

## References

- [getting-started](https://spraxium.com/en/guide/getting-started)
- [examples](https://spraxium.com/en/guide/examples)
- [modules](https://spraxium.com/en/guide/modules)
- [providers-and-lifecycle](https://spraxium.com/en/guide/providers-and-lifecycle)
- [env-project-structure](https://spraxium.com/en/guide/env-project-structure)
- [cli-use](https://spraxium.com/en/guide/cli-use)
- [templates/default/src/main.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/main.ts)
- [templates/default/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/app.module.ts)
- [templates/default/src/app.env.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/app.env.ts)
- [apps/README.md](https://github.com/spraxium/spraxium/blob/main/apps/README.md)
- [apps/slash-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/app.module.ts)
