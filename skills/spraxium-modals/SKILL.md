---
name: spraxium-modals
description: "Open and handle Discord modals with typed fields, validation, cached values and dynamic context. Use for forms and modal submit handling."
metadata:
  framework-version: "0.10.0"
---

# Discord modals

## Follow both interactions

The initial command or component opens the form. Modal submit is a second interaction with data to validate. Keep those boundaries visible in the module. Use `@ModalComponent` to name the form, register the submission `@ModalHandler` and inject `ModalService` into the handler that opens it. Follow the source example for `ComponentsModule` and local service registration. Showing the modal acknowledges the first interaction. Avoid deferring or replying to that interaction before calling the service to open it.

## Give each field a stable ID and rules

Choose each field's type and field name from the docs, then use the matching field decorator at submit time. Add meaningful constraints with `@ModalValidate` and handle a rejected submission before doing any writes. The pieces consumers depend on are custom ID, field IDs and reply behavior. Keep those stable when you only want to edit the label. For dynamic forms use `@ModalWhen` and the documented dynamic field or parameter helpers. Conditional fields can be absent. Treat modal parameters as context and recheck authorization, current ticket state and ownership when submitting.

## Pick appropriate persistence

`@ModalCache` helps prefill values in forms that are reopened. Choose a TTL from what users expect and decide whether the specific field belongs in cached form state. For an edit form, load the authoritative record from the service before overwriting anything. For a multiprocess or sharded bot, read the actual component context strategy before assuming that a form opened by one process can be resolved anywhere. Keep secrets and private user entries out of client-visible IDs and logs.

## Check the whole flow in Discord

Test opening the form, a normal submission, an invalid answer, and submissions with a stale or missing context. For dynamic forms test each branch and whether omitted fields reach the handler gracefully. If the first click fails, inspect the acknowledgment. If submitting fails, verify matching descriptor, handler registration, field names and the context store before adjusting validation.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/modal-bot/src/modules/ticket/components/ticket-modal.component.ts

~~~ts
import {
  ModalCache,
  ModalComponent,
  ModalInput,
  ModalTextDisplay,
  ModalValidate,
  ModalValidationConfig,
  emailFormat,
  minWords,
} from '@spraxium/components';

@ModalValidationConfig({
  ephemeral: true,
  embed: (errors) => ({
    title: '❌ Validation failed',
    description: errors.map((e) => `• **${e.label}**: ${e.message}`).join('\n'),
    color: 0xed4245,
  }),
})
@ModalCache({ ttl: 600 })
@ModalTextDisplay({ content: '🎫 Fill in your issue details. Your answers are cached for 10 minutes.' })
@ModalComponent({ id: 'ticket', title: 'Open a Support Ticket' })
export class TicketModal {
  @ModalValidate([minWords(3, 'Please summarize your issue in at least 3 words.')])
  @ModalInput({
    label: 'Subject',
    placeholder: 'Brief summary of your issue',
    required: true,
    maxLength: 100,
  })
  subject!: string;

  @ModalValidate([minWords(10, 'Please describe the issue in at least 10 words.')])
  @ModalInput({
    label: 'Description',
    style: 'paragraph',
    placeholder: 'Describe the issue in detail...',
    required: true,
  })
  description!: string;

  @ModalValidate([emailFormat()])
  @ModalInput({ label: 'Contact email (optional)', placeholder: 'you@example.com' })
  email!: string;
}
~~~

### apps/modal-bot/src/modules/ticket/handlers/ticket-open-command.handler.ts

~~~ts
import { Ctx, SlashCommandHandler } from '@spraxium/common';
import { ModalService } from '@spraxium/components';
import type { ChatInputCommandInteraction } from 'discord.js';
import { TicketCommand } from '../commands/ticket.command';
import { TicketModal } from '../components/ticket-modal.component';

@SlashCommandHandler(TicketCommand, { sub: 'open' })
export class TicketOpenCommandHandler {
  constructor(private readonly modals: ModalService) {}

  async handle(@Ctx() interaction: ChatInputCommandInteraction): Promise<void> {
    const modal = this.modals.build(TicketModal);
    await interaction.showModal(modal);
  }
}
~~~

### apps/modal-bot/src/modules/ticket/handlers/ticket-modal-submit-command.handler.ts

~~~ts
import { Ctx } from '@spraxium/common';
import { type ModalContext, ModalHandler, ModalTextField } from '@spraxium/components';
import { TicketModal } from '../components/ticket-modal.component';

@ModalHandler(TicketModal)
export class TicketModalSubmitCommandHandler {
  async handle(
    @ModalTextField('subject') subject: string,
    @ModalTextField('description') description: string,
    @ModalTextField('email') email: string,
    @Ctx() ctx: ModalContext,
  ): Promise<void> {
    const ticketId = `TICKET-${Date.now().toString(36).toUpperCase()}`;

    const lines = [
      `✅ **Ticket created!** ID: \`${ticketId}\``,
      `**Subject:** ${subject}`,
      `**Description:** ${description}`,
    ];

    if (email) {
      lines.push(`**Contact:** ${email}`);
    }

    await ctx.reply({ content: lines.join('\n'), flags: 'Ephemeral' });
  }
}
~~~

## References

- [modals-overview](https://spraxium.com/en/guide/modals-overview)
- [modals-fields](https://spraxium.com/en/guide/modals-fields)
- [modals-handlers-injection](https://spraxium.com/en/guide/modals-handlers-injection)
- [modals-validation](https://spraxium.com/en/guide/modals-validation)
- [modals-dynamics](https://spraxium.com/en/guide/modals-dynamics)
- [modals-cache](https://spraxium.com/en/guide/modals-cache)
- [apps/modal-bot/src/modules/ticket/components/ticket-modal.component.ts](https://github.com/spraxium/spraxium/blob/main/apps/modal-bot/src/modules/ticket/components/ticket-modal.component.ts)
- [apps/modal-bot/src/modules/ticket/handlers/ticket-open-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/modal-bot/src/modules/ticket/handlers/ticket-open-command.handler.ts)
- [apps/modal-bot/src/modules/ticket/handlers/ticket-modal-submit-command.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/modal-bot/src/modules/ticket/handlers/ticket-modal-submit-command.handler.ts)
- [apps/modal-params-bot/src/modules/ticket/handlers/ticket-inline-modal.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/modal-params-bot/src/modules/ticket/handlers/ticket-inline-modal.handler.ts)
- [apps/modal-params-bot/src/modules/ticket/ticket.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/modal-params-bot/src/modules/ticket/ticket.module.ts)
