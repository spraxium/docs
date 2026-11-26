---
name: spraxium-presence
description: "Configure startup presence and rotate activities on a running Discord bot. Use for status, placeholders, and shard overrides."
metadata:
  framework-version: "0.10.0"
---

# Presence and activities

## Set a boot default and update it after readiness

For a static presence, define `PresenceOptions` and pass it to `app.initPresence` before `listen`. At runtime, call `PresenceManager.setStatus` to change only availability, or `setActivity` or `setPresence` after the client becomes ready when you want to change the displayed activity. Choose an `ActivityType` from the installed discord.js. Avoid setting presence in a provider constructor where the client may not be ready.

~~~ts
import { Events, Listener, Once } from '@spraxium/common';
import { PresenceManager } from '@spraxium/core';
import { ActivityType } from 'discord.js';

@Listener(Events.ClientReady)
export class ReadyPresence {
  @Once()
  onReady() {
    PresenceManager.setActivity({ name: 'support requests', type: ActivityType.Watching });
  }
}
~~~

## Decide who owns rotating or shard-specific presence

Pick a modest rotation interval based on platform limits and what the user needs to see. `startRotation` begins a new sequence and `stopRotation` ends it. `reset` restores boot settings. Check documented placeholders and use a literal fallback when a shard cannot know a total. For per-shard presence, read how `setShardPresence` and boot overrides work in the installed version. Avoid another manual loop in each provider setting the same activity.

## Verify on the client

Test boot presence, runtime status, first two rotation items and reset. With multiple shards, verify the global and local values from the Discord side. If an update silently appears to do nothing, check client readiness and look for competing ready handlers setting presence. Preserve the project's existing intent and shard configuration when making a visual update.

## References

- [presence-overview](https://spraxium.com/en/guide/presence-overview)
- [presence-rotation-shards](https://spraxium.com/en/guide/presence-rotation-shards)
- [listeners-overview](https://spraxium.com/en/guide/listeners-overview)
- [packages/core/src/client/presence.manager.ts](https://github.com/spraxium/spraxium/blob/main/packages/core/src/client/presence.manager.ts)
- [packages/core/src/client/interfaces/presence-options.interface.ts](https://github.com/spraxium/spraxium/blob/main/packages/core/src/client/interfaces/presence-options.interface.ts)
- [apps/sandbox/src/events/listeners/ready.listener.ts](https://github.com/spraxium/spraxium/blob/main/apps/sandbox/src/events/listeners/ready.listener.ts)
