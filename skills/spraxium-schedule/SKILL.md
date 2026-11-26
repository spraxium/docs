---
name: spraxium-schedule
description: "Run startup, delayed and recurring jobs with the Spraxium schedule package. Use for cron, intervals, persistence, or shard-safe scheduling."
metadata:
  framework-version: "0.10.0"
---

# Scheduled jobs

## Choose the event that should start the job

For calendar work pick `@Cron` with an explicit timezone. For periodic work pick `@Interval`. Read the difference between `@Timeout`, `@AfterOnline` and `@RunOnce` before using a one-time job. Decide what should happen on restart. Register the configuration with `defineSchedule`, import `ScheduleModule` and make the class holding the decorated job a provider reachable from the root module. Use a unique, stable job name so logs and storage refer to the same work.

## Account for shards and jobs that overlap

Think through how many Node processes will start this provider. A memory driver is useful for one process. Where multiple processes can see the same job, use a supported shared driver and locking strategy or assign an explicit owner. Pick a lock TTL that fits the expected job time and document what happens after a failed run. A scheduler does not make its actual database or Discord action idempotent. Record the operation or check the existing state before repeating it. Split durable long-running work into a service that can resume or expose status when needed.

## Test on a short schedule before shipping

Use controllable dates or an explicit test clock for cron. Test the configured timezone, the first startup, a slow run and a restart. If the job runs twice, inspect process count, lock timing, module imports and whether the same provider is registered more than once. Prefer structured logs with the job name and duration. Don't log secrets or full payloads in a recurring loop.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/schedule-bot/config/schedule.config.ts

~~~ts
import { defineSchedule } from '@spraxium/schedule';

export const scheduleConfig = defineSchedule({
  timezone: 'UTC',
  defaultLockTtlMs: 30_000,
  disableAll: false,
});
~~~

### apps/schedule-bot/src/modules/jobs/services/scheduled-tasks.service.ts

~~~ts
import { Injectable } from '@spraxium/common';
import { Logger } from '@spraxium/logger';
import { Cron, CronExpression, Interval, IntervalExpression } from '@spraxium/schedule';
import { Client } from 'discord.js';

@Injectable()
export class ScheduledTasksService {
  private readonly logger = new Logger(ScheduledTasksService.name);

  constructor(private readonly client: Client) {}

  @Interval(IntervalExpression.EVERY_MINUTE, { name: 'heartbeat' })
  async heartbeat(): Promise<void> {
    this.logger.debug(`Heartbeat - ping: ${this.client.ws.ping} ms`);
  }

  @Interval(IntervalExpression.EVERY_30_MINUTES, { name: 'status-report' })
  async statusReport(): Promise<void> {
    const guilds = this.client.guilds.cache.size;
    this.logger.info(`Status report - guilds: ${guilds}`);
  }

  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, { name: 'daily-reset' })
  async dailyReset(): Promise<void> {
    this.logger.info('Daily reset triggered.');
  }

  @Cron(CronExpression.EVERY_DAY_AT_9AM, { name: 'morning-report' })
  async morningReport(): Promise<void> {
    this.logger.info('Good morning! Daily report running.');
  }
}
~~~

## References

- [schedule-overview](https://spraxium.com/en/guide/schedule-overview)
- [schedule-config-pattern](https://spraxium.com/en/guide/schedule-config-pattern)
- [schedule-decorators-jobs](https://spraxium.com/en/guide/schedule-decorators-jobs)
- [schedule-drivers-ops](https://spraxium.com/en/guide/schedule-drivers-ops)
- [apps/schedule-bot/config/schedule.config.ts](https://github.com/spraxium/spraxium/blob/main/apps/schedule-bot/config/schedule.config.ts)
- [apps/schedule-bot/src/app.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/schedule-bot/src/app.module.ts)
- [apps/schedule-bot/src/modules/jobs/jobs.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/schedule-bot/src/modules/jobs/jobs.module.ts)
- [apps/schedule-bot/src/modules/jobs/services/scheduled-tasks.service.ts](https://github.com/spraxium/spraxium/blob/main/apps/schedule-bot/src/modules/jobs/services/scheduled-tasks.service.ts)
- [apps/schedule-bot/src/modules/jobs/services/startup-tasks.service.ts](https://github.com/spraxium/spraxium/blob/main/apps/schedule-bot/src/modules/jobs/services/startup-tasks.service.ts)
