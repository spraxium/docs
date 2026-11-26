---
name: spraxium-guards
description: "Apply built-in or custom guards to Spraxium handlers. Use for roles, permissions, guild-only checks, and guard options."
metadata:
  framework-version: "0.10.0"
---

# Guards and access checks

## Pick a policy for each layer

Decide who can perform the operation and whether that rule depends on the server, role, permission, or user. Try the matching built-in first. Attach `@UseGuards` to the relevant handler and confirm whether a global rule or a module already applies. A command's Discord registration permissions affect discoverability and availability in Discord. A handler guard decides whether it proceeds. Keep the service responsible for invariants such as ownership and the actual Discord permission when it performs the change.

## Implement a custom guard against its runtime context

Use `@Guard` and implement `SpraxiumGuard.canActivate`. Use `ExecutionContext` to obtain the guild, user, and member. Handle direct messages and uncached guild members. The member roles in discord.js can come as a cached manager or an array, as the VIP example shows. Use `@GuardOption` and `withOptions` when callers supply policy settings. Spraxium 0.10.0 constructs guards without constructor arguments. Use `ExecutionContext` for request data and `@GuardOption` for fixed configuration. If a policy requires an injected service, call it from the handler before acting.

## Exercise a denial and report it well

Cover an allowed member, rejected member, direct message and missing option. Test the same policy on the individual handler surfaces that need it, since component clicks and commands are distinct inputs. Return or throw using the documented pattern for guard failure so the exception system can decide the user-facing response. Avoid sending a reply in multiple layers or exposing details of a private role rule in logs or a denial.

## Read the example in context

The following files come from the Spraxium examples. Follow the source links for their imports and surrounding module. Preserve your application’s names and paths when adapting them.

### apps/guard-bot/src/guards/vip-role.guard.ts

~~~ts
import { Guard, GuardOption } from '@spraxium/common';
import type { ExecutionContext, SpraxiumGuard } from '@spraxium/common';

/**
 * Blocks the handler unless the invoking user has a specific role.
 *
 * The role ID is injected via @UseGuards(withOptions(VipRoleGuard, { roleId: '...' })).
 *
 * This guard works identically on slash commands AND component handlers
 * (buttons, selects, modals) because it only uses ctx.getMember(), which is
 * available on every Discord interaction regardless of type.
 */
@Guard()
export class VipRoleGuard implements SpraxiumGuard {
  @GuardOption({ required: true })
  public roleId!: string;

  public canActivate(ctx: ExecutionContext): boolean {
    // getGuildId() returns null in DMs — block them up front
    if (ctx.getGuildId() === null) return false;

    const member = ctx.getMember();
    if (!member) return false;

    // member.roles is a GuildMemberRoleManager (cached) or an array of snowflakes (API)
    const roles = 'cache' in member.roles ? member.roles.cache : member.roles;
    return Array.isArray(roles) ? roles.includes(this.roleId) : roles.has(this.roleId);
  }
}
~~~

## References

- [guards-overview](https://spraxium.com/en/guide/guards-overview)
- [guards-registration](https://spraxium.com/en/guide/guards-registration)
- [guards-options-built-ins](https://spraxium.com/en/guide/guards-options-built-ins)
- [guards-failures-testing](https://spraxium.com/en/guide/guards-failures-testing)
- [apps/guard-bot/src/guards/vip-role.guard.ts](https://github.com/spraxium/spraxium/blob/main/apps/guard-bot/src/guards/vip-role.guard.ts)
- [apps/guard-bot/src/modules/vip/vip.module.ts](https://github.com/spraxium/spraxium/blob/main/apps/guard-bot/src/modules/vip/vip.module.ts)
- [apps/guard-bot/src/modules/vip/handlers/vip-claim-button.handler.ts](https://github.com/spraxium/spraxium/blob/main/apps/guard-bot/src/modules/vip/handlers/vip-claim-button.handler.ts)
