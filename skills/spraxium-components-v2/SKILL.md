---
name: spraxium-components-v2
description: "Build Discord Components V2 containers, text and actions with Spraxium. Use when changing a V2 layout or a reply carrying the V2 flag."
metadata:
  framework-version: "0.10.0"
---

# Components V2

## Choose the message format at the entry point

V2 is for layouts built from Discord components. Choose the schema, fluent or dynamic pattern from the version's V2 guide and keep that pattern inside the owning feature. Start with the example schema and handler so you can see how they are registered. Read the root module for the component plugin and the feature module for its handler and service. V2 belongs in a new or edited message where you can use the right payload. Older embed-based messages can stay on their existing flow.

## Keep generation and interaction separate

A stat card can take typed data like a guild name and counts. Load the values before calling `V2Service.buildReply` and await the result. Preserve the flags the service returns, including `IsComponentsV2`. To make the reply ephemeral, combine the returned flags with `MessageFlags.Ephemeral`. Keep legacy `content` and `embeds` out of a V2 message. For interactive components use documented button and select handlers, stable custom IDs, and explicit ownership checks. Store flow state through the configured context driver if needed.

## Verify the payload before expanding the design

Send or edit the message in a test guild. Verify that the component layout fits with short and long data, that a button reaches the registered handler, and that another user cannot act on a private flow. If Discord rejects the payload, inspect the reply flags and included fields first. If actions fail, trace the custom ID, handler and async service build before changing the layout.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/sandbox/src/modules/v2-demo/handlers/v2-class-command.handler.ts

~~~ts
import { Ctx, SlashCommandHandler } from '@spraxium/common';
import { V2Service } from '@spraxium/components';
import { MessageFlags, type ChatInputCommandInteraction } from 'discord.js';
import { V2DemoCommand } from '../commands/v2-demo.command';
import {
  type ServerInsightsData,
  ServerInsightsContainer,
} from '../schemas/server-insights.container';

@SlashCommandHandler(V2DemoCommand, { sub: 'class' })
export class V2ClassHandler {
  constructor(private readonly v2: V2Service) {}

  async handle(@Ctx() interaction: ChatInputCommandInteraction): Promise<void> {
    if (!interaction.guild) {
      return;
    }

    const data: ServerInsightsData = {
      guildName: interaction.guild.name,
      memberCount: interaction.guild.memberCount,
      onlineCount:
        interaction.guild.presences?.cache.filter((presence) => presence.status === 'online').size ??
        0,
      boostLevel: interaction.guild.premiumTier,
      iconUrl:
        interaction.guild.iconURL({ size: 128 }) ??
        'https://placehold.co/128x128/5865f2/ffffff.png?text=G',
    };

    const payload = await this.v2.buildReply(ServerInsightsContainer, data);

    await interaction.reply({
      ...payload,
      flags: payload.flags | MessageFlags.Ephemeral,
    });
  }
}
~~~

## References

- [components-v2](https://spraxium.com/en/guide/components-v2)
- [components-buttons](https://spraxium.com/en/guide/components-buttons)
- [components-context](https://spraxium.com/en/guide/components-context)
- [apps/components-bot/src/modules/v2/v2.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/modules/v2/v2.module.ts)
- [apps/components-bot/src/modules/v2/schemas/server-stats.container.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/modules/v2/schemas/server-stats.container.ts)
- [apps/sandbox/src/modules/v2-demo/handlers/v2-class-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/modules/v2-demo/handlers/v2-class-command.handler.ts)
