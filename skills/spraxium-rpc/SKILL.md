---
name: spraxium-rpc
description: "Add a typed Spraxium RPC operation and call it from a backend. Use for new bot-to-backend integrations, gateway settings and handler troubleshooting."
metadata:
  framework-version: "0.10.0"
---

# Connect a backend to the bot

## Separate the two entry points

Keep public URLs, user sessions and server access in the existing backend. Put `@spraxium/rpc` and the gateway module in the bot and `@spraxium/rpc-client` in a server process. Use the same release's contract on each end and validate input and output at runtime. For a project moving from legacy HTTP, use [the dedicated migration skill](https://github.com/spraxium/rpc/blob/main/SKILL.md) as well.

## Add a handler and follow the request

Pick a stable operation name and one read with useful output. Register a provider with `@RpcHandler` in a feature imported by the root module. Set named permissions and choose guild or singleton routing. A guild call carries a top-level Discord string ID, and the gateway chooses the child shard. Return a serializable object containing the fields needed by the caller. The web route validates the session and user's access before calling the bot. Use the documented configuration for TLS and separate service credentials. Keep the token in server-side config.

## Reuse the client and map failures

Create one SDK instance per process and service credential. In Nest, register it using a symbol and `useFactory`, inject it into an application service, call `rpc.call` from that service and let Nest controllers do HTTP concerns. In Express or Fastify, pass the startup client into the router or plugin factory. Close it after outstanding HTTP requests finish. Make deadlines fit the caller's route time budget, handle `RpcClientError` consistently, and keep private RPC details out of public responses.

## Verify the complete route from outside the bot

Check startup, the health call, the allowed guild read, a missing or invalid guild and insufficient permission. With sharding, test a guild on a different child. If the feature writes to Discord, store a stable operation ID and check it before retrying a timed-out write. Test the backend route as an end user separately from testing the service credential. Match all logs by safe request identifiers.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### examples/bot/src/guild-rpc.service.ts

~~~ts
import { Inject, Injectable } from '@spraxium/common';
import { Code, RpcError, RpcHandler } from '@spraxium/rpc';
import type { BotRpcContract } from '@spraxium/rpc-example-shared/bot-rpc-contract';
import { z } from 'zod';
import { GuildService } from './guild.service.ts';

const guildInput = z.object({ guildId: z.string() }).strict();
const guildStats = z
  .object({
    members: z.number().int().nonnegative(),
    premium: z.boolean(),
    shardId: z.number().int().nonnegative(),
  })
  .strict();

@Injectable()
export class GuildRpcService {
  constructor(@Inject(GuildService) private readonly guilds: GuildService) {}

  /** Exposes the guild counters to services with the guild:read permission. */
  @RpcHandler('guild.stats', {
    routing: 'guild',
    permissions: ['guild:read'],
    schema: guildInput,
    responseSchema: guildStats,
  })
  stats({ guildId }: z.infer<typeof guildInput>): BotRpcContract['guild.stats']['output'] {
    const stats = this.guilds.findStats(guildId);
    if (!stats) throw new RpcError('Guild not found.', Code.NotFound);
    return stats;
  }
}
~~~

### examples/nestjs/src/bot-rpc.service.ts

~~~ts
import {
  BadRequestException,
  Inject,
  Injectable,
  NotFoundException,
  type OnApplicationShutdown,
  ServiceUnavailableException,
} from '@nestjs/common';
import { Code, RpcClientError } from '@spraxium/rpc-client';
import type { BotRpcClient } from '@spraxium/rpc-example-shared/bot-rpc-client';
import type { BotRpcContract } from '@spraxium/rpc-example-shared/bot-rpc-contract';
import { BOT_RPC_CLIENT } from './bot-rpc-client.token.ts';

@Injectable()
export class BotRpcService implements OnApplicationShutdown {
  constructor(@Inject(BOT_RPC_CLIENT) private readonly rpc: BotRpcClient) {}

  /** Returns the guild counters used by the dashboard. */
  async getGuildStats(guildId: string): Promise<BotRpcContract['guild.stats']['output']> {
    try {
      return await this.rpc.call('guild.stats', { guildId });
    } catch (error) {
      if (!(error instanceof RpcClientError)) throw error;
      if (error.code === Code.InvalidArgument) throw new BadRequestException('Invalid guild ID');
      if (error.code === Code.NotFound) throw new NotFoundException('Guild not found');
      throw new ServiceUnavailableException('Bot unavailable');
    }
  }

  /** Releases the SDK after Nest has closed its HTTP server. */
  onApplicationShutdown(): void {
    this.rpc.close();
  }
}
~~~

## References

- [rpc-overview](https://spraxium.com/en/guide/rpc-overview)
- [rpc-quickstart](https://spraxium.com/en/guide/rpc-quickstart)
- [rpc-handlers](https://spraxium.com/en/guide/rpc-handlers)
- [rpc-authentication](https://spraxium.com/en/guide/rpc-authentication)
- [rpc-backends](https://spraxium.com/en/guide/rpc-backends)
- [rpc-nestjs](https://spraxium.com/en/guide/rpc-nestjs)
- [rpc-sharding](https://spraxium.com/en/guide/rpc-sharding)
- [rpc-configuration](https://spraxium.com/en/guide/rpc-configuration)
- [rpc-troubleshooting](https://spraxium.com/en/guide/rpc-troubleshooting)
- [examples/bot/src/guild-rpc.service.ts](https://github.com/spraxium/rpc/blob/main/examples/bot/src/guild-rpc.service.ts)
- [examples/bot/src/app.module.ts](https://github.com/spraxium/rpc/blob/main/examples/bot/src/app.module.ts)
- [examples/nestjs/src/guild.module.ts](https://github.com/spraxium/rpc/blob/main/examples/nestjs/src/guild.module.ts)
- [examples/nestjs/src/bot-rpc.service.ts](https://github.com/spraxium/rpc/blob/main/examples/nestjs/src/bot-rpc.service.ts)
- [examples/shared/src/types/bot-rpc-contract.type.ts](https://github.com/spraxium/rpc/blob/main/examples/shared/src/types/bot-rpc-contract.type.ts)
