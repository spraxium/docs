---
name: spraxium-logger
description: "Use @spraxium/logger to record bot events and configure levels, masking, console output, and extra transports."
metadata:
  framework-version: "0.10.0"
---

# Logging and transports

## Name the owner of the event

Use a contextual `Logger` in a service rather than a shared string prefix copied into each message. Pick a level that says what a reader should do with the event. Record a job, guild, or request ID only when it helps correlate the operation, and avoid messages with private Discord content. Follow the method signatures in the installed `@spraxium/logger`. Turn on debug only where the volume is manageable. Use regular exception handling for failures; logging an error does not recover the operation.

~~~ts
import { Logger } from '@spraxium/logger';

const logger = new Logger('TicketCleanup');
logger.info('Cleanup started');
logger.debug('Looking for stale ticket channels');
~~~

## Configure the outputs in one place

Keep logger options in the root config or its imported config file. Check `maskTokens` and the list of levels before enabling an external transport. If output goes to Discord, provide the chosen webhook or channel configuration through deployment settings, limit levels and control who can read the channel. Known-token masking is a fallback. Avoid passing headers, webhook URLs, browser sessions or full request bodies to any logger. If you register a custom transport, use the exported interface and keep existing transports and shutdown behavior in mind.

## Verify log delivery

Check that normal boot and one controlled failure appear in the expected sinks. Confirm debug is filtered when disabled, values you intend to keep private are absent, and command logging isn't duplicated. In a sharded setup correlate by shard and operation. If a transport fails, make sure losing that sink does not hide application failures from all other output.

## References

- [logger-overview](https://spraxium.com/en/guide/logger-overview)
- [logger-transports](https://spraxium.com/en/guide/logger-transports)
- [config-core-fields](https://spraxium.com/en/guide/config-core-fields)
- [packages/logger/src/services/logger.service.ts](https://github.com/spraxium/spraxium/blob/main/packages/logger/src/services/logger.service.ts)
- [packages/logger/src/index.ts](https://github.com/spraxium/spraxium/blob/main/packages/logger/src/index.ts)
- [apps/sandbox/config/logger.config.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/config/logger.config.ts)
- [templates/default/src/main.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/main.ts)
