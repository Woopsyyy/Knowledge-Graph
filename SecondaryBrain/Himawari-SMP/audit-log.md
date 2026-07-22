# Command Audit Log (/log)

Part of [[Himawari-SMP]]. An owner-only audit trail of every command run by staff (mod/owner),
reviewed through a glowing "Command Log" book.

## Capture
- `mixin/CommandAuditMixin` — `@Inject(HEAD)` on `Commands.performPrefixedCommand(CommandSourceStack,
  String)` (void). If the source is a player with `smp.mod` (so mods **and** owners, since owner
  outranks mod), it records `(uuid, name, command)`. Failures are swallowed so logging can't break a
  command. Registered in `survivalmod.mixins.json`.
- `audit/AuditLog` — per-player ring buffer (latest 50), persisted to
  `<world>/survivalmod_audit.json` (load on start, save on stop). `record / recent / tracked / nameOf`.

## /log tool (owner only, with or without OP)
- `/log` (gated `isOwnerSource`) gives the **Command Log** book — `WRITABLE_BOOK` marked
  `CUSTOM_DATA{survivalmod_commandlog}` (`audit/CommandLogBook`).
- Right-click → `gui/CommandLogMenu`: a player head per staff member who has logged commands; clicking
  a head closes the menu and prints that player's most recent commands (timestamped) to chat. The
  right-click is gated to `smp.owner`.

## Shared staff-tool behaviour (with the [[admin-investigator]] book)
- **Destroyed on drop**: a `ServerEntityEvents.ENTITY_LOAD` handler in `SurvivalMod.java` discards any
  dropped `ItemEntity` whose stack is an Investigator or Command Log book — so a staff tool can't be
  left in a chest / picked up by others.
- **Rank-gated use**: right-clicking is checked — Investigator needs `smp.mod`, Command Log needs
  `smp.owner` — so even a stolen tool is useless.

## Related
[[admin-investigator]] · [[moderation-bans]] · [[Himawari-SMP]]
