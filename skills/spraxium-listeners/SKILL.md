---
name: spraxium-listeners
description: "Subscribe to Discord events with Spraxium listeners. Use for ready, guild/member events, or delegating event work to a provider."
metadata:
  framework-version: "0.10.0"
---

# Discord event listeners

## Follow the Discord event contract

Choose the event from the documented `Events` API and confirm its argument tuple with the installed discord.js types. A class decorated with `@Listener` names the event. Mark the method with `@Once` for a one-shot hook or the mapped event decorator for repeated events. Register the listener in the owning module's `listeners` field, then import that module through the root graph. Confirm the gateway intents and partials needed by that event.

## Keep the listener small

Use the handler to interpret the event and pass the minimum required data to an injected service. Ignore events from other bots when reacting to messages so the bot cannot answer itself. Guild/member data can be incomplete depending on cache and intents. Protect side effects with an application-level check when events can be replayed on reconnect. For initialization that owns a resource, prefer the matching application lifecycle hook instead of an event listener that can register competing instances.

## Check timing and errors

Confirm one startup event, a regular repeating event, and an event that should be ignored. A listener without a Discord interaction usually reports failures to logging, not to an ephemeral message. If a listener never runs, check module reachability, event name and intents before changing the handler signature. If it runs twice, look for duplicate registration or a separate native Discord subscription.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/sandbox/src/events/listeners/ready.listener.ts

~~~ts
import { Events, Listener, Once } from '@spraxium/common';
import { Logger } from '@spraxium/logger';
import type { Client } from 'discord.js';

@Listener(Events.ClientReady)
export class ReadyListener {
  private logger: Logger = new Logger(ReadyListener.name);

  @Once()
  onReady(client: Client<true>): void {
    this.logger.info(`Client is ready as ${client.user.tag}`);
  }
}
~~~

### apps/sandbox/src/events/events.module.ts

~~~ts
import { Module } from '@spraxium/common';
import { MemberJoinListener, MemberLeaveListener } from './listeners/member.listener';
import { MessageCreateListener } from './listeners/message-create.listener';
import { ReadyListener } from './listeners/ready.listener';

@Module({
  listeners: [ReadyListener, MessageCreateListener, MemberJoinListener, MemberLeaveListener],
})
export class EventsModule {}
~~~

## References

- [listeners-overview](https://spraxium.com/en/guide/listeners-overview)
- [listeners-decorators-lifecycle](https://spraxium.com/en/guide/listeners-decorators-lifecycle)
- [listeners-architecture-patterns](https://spraxium.com/en/guide/listeners-architecture-patterns)
- [listeners-exceptions-integration](https://spraxium.com/en/guide/listeners-exceptions-integration)
- [apps/sandbox/src/events/listeners/ready.listener.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/events/listeners/ready.listener.ts)
- [apps/sandbox/src/events/listeners/member.listener.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/events/listeners/member.listener.ts)
- [apps/sandbox/src/events/events.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/events/events.module.ts)
- [packages/core/src/listeners/listener.invoker.ts](https://github.com/spraxium/spraxium/blob/main/packages/core/src/listeners/listener.invoker.ts)
