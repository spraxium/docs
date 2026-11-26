---
name: spraxium-exceptions
description: "Define domain errors and user-visible Discord replies through Spraxium's exception pipeline, layouts, and logging."
metadata:
  framework-version: "0.10.0"
---

# Exceptions and error replies

## Decide how the failure should reach the developer and user

Use a built-in exception when it already expresses the failure. For a domain problem such as a closed ticket, derive from `SpraxiumException` and give it a stable code, safe public message and explicit properties. Ordinary programming failures should still be visible to operators. Use correlation metadata and safe fields in logs. Don't include another person's private information, access tokens, or an exception stack in the Discord reply.

## Choose the scope of presentation

Read the active precedence in the exception layout strategy guide. Use a layout attached with `@WithLayout` when the exception owns its presentation. Use the configuration mapping when an application wants to customize a framework exception without changing the original class. Use the default for the rest. Use `defineExceptionLayout` for a function layout. Keep formatting out of business rules. Check whether the handler already replied or deferred and let the framework apply the corresponding reply strategy.

## Test the failure where it originates

Trigger the exception from a test handler, then try the same feature after a deferred reply. Exercise bot actions in contexts without a Discord interaction separately. The expected result for an event can be a log instead of a reply. For an RPC handler, use the RPC error contract, not a Discord exception layout. Check whether a caught error is being swallowed or responded to twice before adding another error handler.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/sandbox/src/exceptions/account-suspended.exception.ts

~~~ts
import { WithLayout } from '@spraxium/common';
import { SpraxiumException, defineExceptionLayout } from '@spraxium/core';
import { EmbedBuilder } from 'discord.js';

const SuspendedLayout = defineExceptionLayout((ex) => ({
  embeds: [
    new EmbedBuilder()
      .setColor(0xff4444)
      .setTitle('Account Suspended')
      .setDescription(
        `Your account has been suspended.\n\n**Reason:** ${ex.props.reason ?? 'No reason provided'}\n**Expires:** ${ex.props.expiresAt ?? 'Never'}`,
      )
      .setFooter({ text: `Case ID: ${ex.props.caseId}` }),
  ],
  ephemeral: true,
}));

@WithLayout(SuspendedLayout)
export class AccountSuspendedException extends SpraxiumException {
  constructor(props: { reason?: string; expiresAt?: string; caseId: string }) {
    super({
      code: 'ACCOUNT_SUSPENDED',
      message: 'This account is currently suspended.',
      props,
      shouldLog: true,
    });
  }
}
~~~

## References

- [exceptions-overview](https://spraxium.com/en/guide/exceptions-overview)
- [exception-layout-strategy](https://spraxium.com/en/guide/exception-layout-strategy)
- [exceptions-configuration](https://spraxium.com/en/guide/exceptions-configuration)
- [exceptions-best-practices](https://spraxium.com/en/guide/exceptions-best-practices)
- [apps/sandbox/src/exceptions/account-suspended.exception.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/exceptions/account-suspended.exception.ts)
- [apps/sandbox/src/exceptions/exceptions.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/exceptions/exceptions.module.ts)
- [packages/core/src/exceptions/exception.handler.ts](https://github.com/spraxium/spraxium/blob/main/packages/core/src/exceptions/exception.handler.ts)
- [packages/core/src/exceptions/discord-reply.strategy.ts](https://github.com/spraxium/spraxium/blob/main/packages/core/src/exceptions/discord-reply.strategy.ts)
