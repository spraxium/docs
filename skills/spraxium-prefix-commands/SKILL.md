---
name: spraxium-prefix-commands
description: "Configure message prefixes, arguments, aliases and subcommands, and verify Discord intents. Use for commands such as !help or !roll."
metadata:
  framework-version: "0.10.0"
---

# Prefix commands

## Start from one message example

Pick a command and write the expected behavior for a message like `!roll 20 2`. Review the `prefix` block for server overrides, mention prefix, case sensitivity and default cooldown. In Discord, message-content access is a privileged intent. Check the Developer Portal and the bot's runtime intents for the guild message flow. If the bot cannot use the intent for your deployment, discuss a slash command before writing a parser.

## Declare, parse and dispatch

Use `@PrefixCommand` for name, aliases, usage and policy. Put typed `@PrefixArg` decorators on the definition and use `@PrefixArg('name')` at the handler boundary. When the pipeline rejects an invalid integer or missing required argument, keep that logic in the existing argument or exception layer. Register the command in `commands` and the `@PrefixCommandHandler` in `handlers`. `@Ctx()` here is a Discord `Message`. It has `reply` but no slash-interaction acknowledgment. For multiword arguments, consult the parsing page before choosing its quoting, rest or index behavior.

## Cover permission and runtime edge cases

Make guild-only assumptions explicit when the handler accesses member roles or channels. Keep slow work behind a guard and use configurable per-guild prefixes when the project already supports them. Test the default prefix, alias, optional input, missing or malformed input, a denied member and a message that should be ignored. Verify bot messages can't trigger a loop. For a command that is never picked up, check Message Content, bot login, actual configured prefix, and the root module imports before replacing any API.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/sandbox/src/modules/prefix/commands/roll.command.ts

~~~ts
import { PrefixArg, PrefixCommand } from '@spraxium/common';

@PrefixCommand({
  name: 'roll',
  aliases: ['r', 'dice'],
  description: 'Roll one or more dice.',
  category: 'Fun',
  usage: '!roll [sides] [count]',
  cooldown: 3,
})
export class RollCommand {
  @PrefixArg.Integer('sides', { required: false, min: 2, max: 1000 })
  @PrefixArg.Integer('count', { required: false, min: 1, max: 25 })
  build() {}
}
~~~

### apps/sandbox/src/modules/prefix/handlers/roll-command.handler.ts

~~~ts
import { Ctx, PrefixArg, PrefixCommandHandler } from '@spraxium/common';
import { Logger } from '@spraxium/logger';
import type { Message } from 'discord.js';
import { RollCommand } from '../commands/roll.command';

@PrefixCommandHandler(RollCommand)
export class RollHandler {
  private readonly logger = new Logger(RollHandler.name);

  async handle(
    @Ctx() message: Message,
    @PrefixArg('sides') sides: number | undefined,
    @PrefixArg('count') count: number | undefined,
  ): Promise<void> {
    const faces = sides ?? 6;
    const rolls = count ?? 1;

    const results = Array.from({ length: rolls }, () => Math.floor(Math.random() * faces) + 1);
    const total = results.reduce((a, b) => a + b, 0);

    const rollStr = results.join(', ');
    const summary =
      rolls > 1
        ? `🎲 Rolled **${rolls}d${faces}**: [${rollStr}] → Total: **${total}**`
        : `🎲 Rolled **1d${faces}**: **${results[0]}**`;

    this.logger.debug(`${message.author.tag} rolled ${rolls}d${faces}: [${rollStr}]`);
    await message.reply(summary);
  }
}
~~~

## References

- [prefix-commands-overview](https://spraxium.com/en/guide/prefix-commands-overview)
- [prefix-commands-definition](https://spraxium.com/en/guide/prefix-commands-definition)
- [prefix-commands-arguments-parsing](https://spraxium.com/en/guide/prefix-commands-arguments-parsing)
- [prefix-commands-execution-guild-prefixes](https://spraxium.com/en/guide/prefix-commands-execution-guild-prefixes)
- [prefix-commands-service-api](https://spraxium.com/en/guide/prefix-commands-service-api)
- [prefix-commands-runtime-testing](https://spraxium.com/en/guide/prefix-commands-runtime-testing)
- [apps/sandbox/spraxium.config.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/spraxium.config.ts)
- [apps/sandbox/src/modules/prefix/commands/roll.command.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/prefix/commands/roll.command.ts)
- [apps/sandbox/src/modules/prefix/handlers/roll-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/prefix/handlers/roll-command.handler.ts)
- [apps/sandbox/src/modules/prefix/prefix.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/prefix/prefix.module.ts)
