---
name: spraxium-components
description: "Register button and select handlers, build rows, and carry component context between Discord interactions."
metadata:
  framework-version: "0.10.0"
---

# Buttons, selects, and component context

## Start with the action that receives the click

Import `ComponentsModule` using the same root pattern as the example. Define the button or select with the matching decorator and a stable custom ID. Register its execution handler in the feature module and inject the appropriate component service when constructing a row. A link button opens its URL and has no local callback. A select can return one or more values. Use the matching parameter decorators and validate that the value represents an action the current user may take.

## Handle stale context and mixed clients

Use the component-context helpers when a later click needs data from the original interaction. Pick an expiry and a context driver consistent with restarts and shards. Store an ID and look up the latest record when users can change it from another message. Avoid embedding private values or trusting a client-visible ID as proof of access. For layouts in V2, follow the separate V2 skill so the reply's flags and component rows fit Discord's rules.

## Verify the round trip

Test the initial message and the actual click. Cover a member without access, a second click, an expired context and a response after the bot restarts if the flow promises persistence. When a component renders but nothing handles it, compare the built custom ID against the decorated handler and module registrations. If data is wrong, inspect context lifetime and where the application writes it.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/components-bot/src/modules/buttons/components/approve-button.component.ts

~~~ts
import { Button } from '@spraxium/components';

@Button({ customId: 'btn_approve', label: 'Approve', style: 'success', emoji: '✅' })
export class ApproveButton {}
~~~

### apps/components-bot/src/modules/buttons/handlers/approve-button-command.handler.ts

~~~ts
import { Ctx } from '@spraxium/common';
import { ButtonHandler } from '@spraxium/components';
import type { ButtonInteraction } from 'discord.js';
import { ApproveButton } from '../components/approve-button.component';

@ButtonHandler(ApproveButton)
export class ApproveButtonCommandHandler {
  async handle(@Ctx() interaction: ButtonInteraction): Promise<void> {
    await interaction.reply({ content: '✅ Approve button clicked!', flags: 'Ephemeral' });
  }
}
~~~

## References

- [components-overview](https://spraxium.com/en/guide/components-overview)
- [components-buttons](https://spraxium.com/en/guide/components-buttons)
- [components-selects](https://spraxium.com/en/guide/components-selects)
- [components-context](https://spraxium.com/en/guide/components-context)
- [components-v2](https://spraxium.com/en/guide/components-v2)
- [apps/components-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/app.module.ts)
- [apps/components-bot/src/modules/buttons/buttons.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/modules/buttons/buttons.module.ts)
- [apps/components-bot/src/modules/buttons/components/approve-button.component.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/modules/buttons/components/approve-button.component.ts)
- [apps/components-bot/src/modules/buttons/handlers/approve-button-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/modules/buttons/handlers/approve-button-command.handler.ts)
