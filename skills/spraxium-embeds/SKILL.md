---
name: spraxium-embeds
description: "Build readable Discord embeds from typed schemas and service data. Use for conditional fields, dynamic formatting and pagination."
metadata:
  framework-version: "0.10.0"
---

# Discord embeds

## Put the message layout in one place

Use an embed schema for output that several interactions share or that depends on structured data. Follow the version's `@Embed`, `@EmbedTitle`, `@EmbedDescription` and `@EmbedField` API. Gather raw values in the handler or a service and pass a typed object to `EmbedService.build`. Keep schema functions focused on formatting and avoid accessing a database from a decorator callback. Use `@EmbedWhen` to handle fields that are specific to a guild or have no usable value.

## Send it with the correct Discord payload

Build the embed and add it to the message's `embeds`. Keep traditional embeds separate from a Components V2 payload that forbids legacy content. If text comes from another user, handle mentions and escape content as appropriate. Pay attention to Discord's combined content constraints when generating many fields. For a growing list, use a paginator with an interaction bound to the right user and confirm the state or context lifetime. If the data should change between pages, inspect `paginateData` or the project's existing paging approach rather than building all pages from stale data.

## Verify the visible message

Test a guild and a direct message if both are supported, an empty list, populated data, and a string near a Discord display limit. For navigation, check back/next at both ends, timeout behavior, and a different user's click. If Discord rejects an otherwise valid schema, inspect the built payload, the flags and response size instead of duplicating the schema in the handler.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/sandbox/src/modules/embed-demo/handlers/embed-demo-build-command.handler.ts

~~~ts
import { Ctx, SlashCommandHandler } from '@spraxium/common';
import { EmbedService } from '@spraxium/components';
import type { ChatInputCommandInteraction } from 'discord.js';
import { EmbedDemoCommand } from '../commands/embed-demo.command';
import { type ServerStatsData, ServerStatsEmbed } from '../schemas/server-stats.embed';

@SlashCommandHandler(EmbedDemoCommand, { sub: 'build' })
export class EmbedDemoBuildHandler {
  constructor(private readonly embeds: EmbedService) {}

  async handle(@Ctx() interaction: ChatInputCommandInteraction): Promise<void> {
    const data: ServerStatsData = {
      ws: interaction.client.ws.ping,
      guildCount: interaction.client.guilds.cache.size,
      userCount: interaction.client.users.cache.size,
      guildName: interaction.guild?.name ?? null,
      channelCount: interaction.guild?.channels.cache.size ?? null,
    };

    const embed = this.embeds.build(ServerStatsEmbed, data);
    await interaction.reply({ embeds: [embed] });
  }
}
~~~

## References

- [components-embeds](https://spraxium.com/en/guide/components-embeds)
- [components-overview](https://spraxium.com/en/guide/components-overview)
- [components-context](https://spraxium.com/en/guide/components-context)
- [apps/sandbox/src/modules/embed-demo/schemas/server-stats.embed.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/embed-demo/schemas/server-stats.embed.ts)
- [apps/sandbox/src/modules/embed-demo/handlers/embed-demo-build-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/embed-demo/handlers/embed-demo-build-command.handler.ts)
- [apps/sandbox/src/modules/embed-demo/handlers/embed-demo-paginator-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/embed-demo/handlers/embed-demo-paginator-command.handler.ts)
- [apps/sandbox/src/modules/embed-demo/embed-demo.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/embed-demo/embed-demo.module.ts)
