---
name: spraxium-i18n
description: "Translate commands and messages with @spraxium/i18n. Use for locale files, interpolation, plural rules, component labels, and preference storage."
metadata:
  framework-version: "0.10.0"
---

# Translations and locale preferences

## Keep stable keys across locales

Before adding text, inspect the configured references and look for an existing key. Add the same key to each applicable locale, preserving interpolation variables and plural forms. Use locale IDs that the framework and Discord understand, such as `en-US` and `pt-BR` in the examples. Map the directory through `defineI18n`, keep it available in build and deployment, and import `I18nModule` in the bot. Changing a published key can affect commands, dynamic components and any stored references.

## Resolve the language at the boundary

For user-facing replies use the installed service's `tUser(userId, key, vars, guildId)` or the matching interaction helper. Use `tpUser` when rendering a count that needs pluralization. Check the active user, guild and default resolution path before making it part of application behavior. For component translation, match the documented decorators rather than translating the custom ID. Stores persist locale choices when needed. Memory is suitable for temporary preferences. A shared deployment needs a store that survives process restarts and can read guild and user preferences consistently.

## Test missing and dynamic values

Run a command in the default locale and a non-default locale. Check fallback when the translation, user preference, or guild setting is missing. Include one and several counts, names with punctuation, and a later click that needs to use the same locale. If a raw key leaks through, verify the reference prefix, path, file name and actual locale used at lookup before adding duplicate labels.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/i18n-bot/config/i18n.config.ts

~~~ts
import { defineI18n } from '@spraxium/i18n';

export const i18nConfig = defineI18n({
  defaultLocale: 'en-US',
  references: [
    {
      name: 'commands',
      path: './locales',
    },
  ],
});
~~~

### apps/i18n-bot/src/modules/greet/handlers/greet-me-command.handler.ts

~~~ts
import { Ctx, SlashCommandHandler } from '@spraxium/common';
import { I18nService } from '@spraxium/i18n';
import type { ChatInputCommandInteraction } from 'discord.js';
import { GreetCommand } from '../commands/greet.command';

@SlashCommandHandler(GreetCommand, { sub: 'me' })
export class GreetMeHandler {
  constructor(private readonly i18n: I18nService) {}

  async handle(@Ctx() interaction: ChatInputCommandInteraction): Promise<void> {
    const userId = interaction.user.id;
    const username = interaction.user.username;
    const locale = await this.i18n.getUserLocale(userId);

    const reply = await this.i18n.tUser(userId, 'commands.greet.me.reply', {
      user: username,
      locale,
    });

    const tip = await this.i18n.tUser(userId, 'commands.greet.me.tip');

    await interaction.reply({ content: `${reply}\n${tip}`, flags: 'Ephemeral' });
  }
}
~~~

## References

- [i18n-overview](https://spraxium.com/en/guide/i18n-overview)
- [i18n-references](https://spraxium.com/en/guide/i18n-references)
- [i18n-components](https://spraxium.com/en/guide/i18n-components)
- [i18n-pluralization](https://spraxium.com/en/guide/i18n-pluralization)
- [i18n-store](https://spraxium.com/en/guide/i18n-store)
- [apps/i18n-bot/config/i18n.config.ts](https://github.com/spraxium/spraxium/blob/main/apps/i18n-bot/config/i18n.config.ts)
- [apps/i18n-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/i18n-bot/src/app.module.ts)
- [apps/i18n-bot/src/modules/greet/handlers/greet-me-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/i18n-bot/src/modules/greet/handlers/greet-me-command.handler.ts)
- [apps/i18n-bot/locales/pt-BR.json](https://github.com/spraxium/spraxium/blob/main/apps/i18n-bot/locales/pt-BR.json)
- [packages/i18n/src/service/i18n.service.ts](https://github.com/spraxium/spraxium/blob/main/packages/i18n/src/service/i18n.service.ts)
