---
name: spraxium-plugins
description: "Install and configure first-party Spraxium plugins. Use when wiring package factories and modules for components, translations, scheduling, signals, webhooks, or RPC."
metadata:
  framework-version: "0.10.0"
---

# First-party plugins

## Begin with the owning process

Identify whether this code is the bot, a worker, or a web service. Install the package into the process that uses it. Match the bot's framework release and the plugin's peer requirements. Read the installed package's export file and the relevant example before copying a factory. Core options such as presence and logger configuration have their own API and need not be treated as plugin entries.

## Add configuration and registration

Keep the existing `plugins` entries when adding a new factory. Examples include `defineComponents` with `ComponentsModule`, `defineSchedule` with `ScheduleModule`, `defineI18n` with `I18nModule`, and `defineRpc` with `RpcModule`. Verify the matching module, exports, and provider registration for Signal or Webhook in the specific example. Configure secrets through an environment schema or deployment secret store. Keep plugin-specific configuration in a small config file when it has several fields. Avoid creating more than one owner of the same runtime resource.

## Check the lifecycle and boundary

Boot the application and inspect whether the feature registers. For a background or network feature, exercise shutdown as well as startup. Confirm that the chosen plugin actually handles the job. For example, first-party Signal delivers an event, whereas RPC handles a server-to-server request expecting a reply. Older example apps that use `@spraxium/http` are for legacy maintenance. New web APIs belong in the user's web framework. When adding an official plugin, preserve the rest of the application's config and run the existing checks.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/components-bot/spraxium.config.ts

~~~ts
import { defineConfig } from '@spraxium/core';
import { componentsConfig } from './config/components.config';

export default defineConfig((env) => ({
  debug: env.isNeutral,
  plugins: [componentsConfig],
}));
~~~

### apps/components-bot/src/app.module.ts

~~~ts
import { Module } from '@spraxium/common';
import { ComponentsModule } from '@spraxium/components';
import { ButtonsModule } from './modules/buttons/buttons.module';
import { FlowModule } from './modules/flow/components/flow.module';
import { ModalModule } from './modules/modal/modal.module';
import { SelectsModule } from './modules/selects/selects.module';
import { V2Module } from './modules/v2/v2.module';

@Module({
  imports: [ComponentsModule, ButtonsModule, SelectsModule, ModalModule, FlowModule, V2Module],
})
export class AppModule {}
~~~

## References

- [config-plugins](https://spraxium.com/en/guide/config-plugins)
- [configuration-layers](https://spraxium.com/en/guide/configuration-layers)
- [config-core-fields](https://spraxium.com/en/guide/config-core-fields)
- [config-creating-plugins](https://spraxium.com/en/guide/config-creating-plugins)
- [signal-overview](https://spraxium.com/en/guide/signal-overview)
- [webhook-overview](https://spraxium.com/en/guide/webhook-overview)
- [rpc-overview](https://spraxium.com/en/guide/rpc-overview)
- [packages/core/src/config/define-plugin.helper.ts](https://github.com/spraxium/spraxium/blob/main/packages/core/src/config/define-plugin.helper.ts)
- [apps/components-bot/spraxium.config.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/spraxium.config.ts)
- [apps/components-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/components-bot/src/app.module.ts)
- [apps/schedule-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/schedule-bot/src/app.module.ts)
- [apps/i18n-bot/spraxium.config.ts](https://github.com/spraxium/spraxium/blob/main/apps/i18n-bot/spraxium.config.ts)
- [examples/bot/src/app.module.ts](https://github.com/spraxium/rpc/blob/main/examples/bot/src/app.module.ts)
- [examples/bot/spraxium.config.ts](https://github.com/spraxium/rpc/blob/main/examples/bot/spraxium.config.ts)
