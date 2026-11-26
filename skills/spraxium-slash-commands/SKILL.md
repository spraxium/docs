---
name: spraxium-slash-commands
description: "Define slash commands, options, subcommands, autocomplete and guarded handlers. Use for Discord chat-input commands and interaction replies."
metadata:
  framework-version: "0.10.0"
---

# Slash commands

## Trace the command from Discord to its handler

A definition decorated with `@SlashCommand` describes what Discord registers. Add options with `@SlashOption` and keep their names and required flags aligned with the handler parameters. A class marked `@SlashCommandHandler(YourCommand)` owns execution. Use `@Ctx()` for the interaction or the documented injection decorators for options. Put the command in its feature module's `commands`, the handler in `handlers`, and import the module from the root graph. For older bots, read the existing guild or global registration settings before making a change.

## Add options, routes and access

Choose a real type for each option. Preserve Discord IDs as strings. For optional values, define what the handler does when omitted. For a group or subcommand, declare the path on the command and point each handler at the corresponding path using the documented options. An autocomplete response suggests up to Discord's allowed count and is a separate handler. Mark that option for autocomplete and register the autocomplete handler. Validate the final selection in the command too. Protect moderation code with a guard and enforce Discord-side role hierarchy where the action happens.

## Handle timing and replies

Reply once. When work can take longer than the Discord acknowledgment window, use the version's `@Defer` or `@AutoDefer` pattern and finish with `editReply` after a defer. An autocomplete interaction responds with choices independently. Prefer an explicit plan for a reply, edit, or follow-up rather than guessing whether a previous helper already acknowledged the interaction. Keep network and database work in services so you can test it without logging in to Discord.

## Verify Discord behavior

Build and start in a test guild. Check the registered name and options, normal reply, missing optional argument, denied user, and autocomplete with no matches. If a command does not show up, inspect module registration, the application and installation scope, and the guild/global deployment. If it shows up but doesn't reply, check handler registration and the exact path and option names first.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/slash-bot/src/modules/simple/commands/ping.command.ts

~~~ts
import { SlashCommand } from '@spraxium/common';

@SlashCommand({ name: 'ping', description: 'Replies with pong and WebSocket latency.' })
export class PingCommand {
  build() {}
}
~~~

### apps/slash-bot/src/modules/simple/handlers/ping-command.handler.ts

~~~ts
import { Ctx, SlashCommandHandler } from '@spraxium/common';
import type { ChatInputCommandInteraction } from 'discord.js';
import { PingCommand } from '../commands/ping.command';

@SlashCommandHandler(PingCommand)
export class PingHandler {
  async handle(@Ctx() interaction: ChatInputCommandInteraction): Promise<void> {
    const ws = interaction.client.ws.ping;
    await interaction.reply(`🏓 Pong! WebSocket latency: **${ws}ms**.`);
  }
}
~~~

## References

- [slash-commands-overview](https://spraxium.com/en/guide/slash-commands-overview)
- [slash-commands-definition-options](https://spraxium.com/en/guide/slash-commands-definition-options)
- [slash-commands-handlers-injection](https://spraxium.com/en/guide/slash-commands-handlers-injection)
- [slash-commands-autocomplete](https://spraxium.com/en/guide/slash-commands-autocomplete)
- [slash-commands-defer](https://spraxium.com/en/guide/slash-commands-defer)
- [slash-commands-subcommands-groups](https://spraxium.com/en/guide/slash-commands-subcommands-groups)
- [slash-commands-guards-permissions-errors](https://spraxium.com/en/guide/slash-commands-guards-permissions-errors)
- [apps/slash-bot/src/modules/simple/commands/ping.command.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/modules/simple/commands/ping.command.ts)
- [apps/slash-bot/src/modules/simple/handlers/ping-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/modules/simple/handlers/ping-command.handler.ts)
- [apps/slash-bot/src/modules/simple/simple.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/modules/simple/simple.module.ts)
- [apps/slash-bot/src/modules/autocomplete/handlers/search-query.autocomplete.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/modules/autocomplete/handlers/search-query.autocomplete.ts)
