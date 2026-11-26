---
name: spraxium-architecture
description: "Structure Spraxium modules, providers, and lifecycle hooks. Use when adding a feature, moving a service, or fixing dependency injection."
metadata:
  framework-version: "0.10.0"
---

# Organize a bot by feature

## Map the existing module graph

Begin with `app.useModule(AppModule)`. Follow the `@Module({ imports: [...] })` graph before moving a file. Put transport-facing code in the feature that already owns the behavior. A ticket command and a ticket button that close the same ticket should delegate to one ticket service. Keep registration close to the feature. If two features need the same service, give it an owner module, export the provider and import the owner in the consuming module. A plain TypeScript import only makes a name visible to the compiler. It doesn't register anything with Spraxium.

## Register by responsibility

Use `commands` for definitions, `handlers` for command and component execution, `listeners` for Discord event subscribers and `providers` for services and other injectable values. Match field names and content to the official example for the feature. Mark regular providers with `@Injectable()` when they take dependencies. Preserve the project's metadata and explicit `@Inject(...)` use when a dependency is a token or a type erased by TypeScript. Register configuration once at boot with `app.provide(AppEnv, environment)` after validating it. Keep the provider token imported by consumers identical to the token used during registration.

## Use lifecycle for resources

Startup code prepares a dependency before a handler uses it. Open databases and similar resources in their owner's lifecycle hook and close them in its shutdown hook. Put Discord-only work in the phase that runs after connection. Look up the matching lifecycle interface before choosing a hook. Treat a new native shard as its own bot process. A cached singleton in one process will not be shared with other shards.

## Verify a layering change

Check the boot entry point after moving a module. Build and run the smallest feature that injects the provider. For an unknown dependency, inspect the token, the provider's module and the export/import path before adding a duplicate registration. A cycle between two feature modules often means some shared behavior needs a clear owner. Test both the success case and one guard or service failure to make sure an entry point still delegates where you expect.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/slash-bot/src/modules/simple/simple.module.ts

~~~ts
import { Module } from '@spraxium/common';
import { ColorCommand } from './commands/color.command';
import { DigestCommand } from './commands/digest.command';
import { InspectCommand } from './commands/inspect.command';
import { PingCommand } from './commands/ping.command';
import { ReportCommand } from './commands/report.command';
import { WeatherCommand } from './commands/weather.command';
import { ColorHandler } from './handlers/color-command.handler';
import { DigestHandler } from './handlers/digest-command.handler';
import { InspectHandler } from './handlers/inspect-command.handler';
import { PingHandler } from './handlers/ping-command.handler';
import { ReportHandler } from './handlers/report-command.handler';
import { WeatherHandler } from './handlers/weather-command.handler';

@Module({
  commands: [PingCommand, InspectCommand, ColorCommand, ReportCommand, WeatherCommand, DigestCommand],
  handlers: [PingHandler, InspectHandler, ColorHandler, ReportHandler, WeatherHandler, DigestHandler],
})
export class SimpleModule {}
~~~

## References

- [modules](https://spraxium.com/en/guide/modules)
- [providers-and-lifecycle](https://spraxium.com/en/guide/providers-and-lifecycle)
- [env-project-structure](https://spraxium.com/en/guide/env-project-structure)
- [examples](https://spraxium.com/en/guide/examples)
- [templates/default/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/app.module.ts)
- [templates/default/src/main.ts](https://github.com/spraxium/spraxium/blob/main/templates/default/src/main.ts)
- [apps/slash-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/app.module.ts)
- [apps/slash-bot/src/modules/simple/simple.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/slash-bot/src/modules/simple/simple.module.ts)
