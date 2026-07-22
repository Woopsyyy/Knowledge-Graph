# Combat Status (PvP tag + boss bar)

Part of [[Himawari-SMP]].

## What it is
A PvP "combat tag": dealing or taking player-vs-player damage tags both players for
`combatTagSeconds` (Supabase `features.combatTagSeconds`, default 5s). While tagged, teleports are
refused so players can't escape a fight. Ephemeral game-time stamps, no persistence.

## Key file
`combat/CombatTracker.java`:
- `lastPvp: Map<UUID,Long>` — overworld game-time of the last PvP hit. `isTagged()`, `secondsLeft()`.
- `tag(player)` — set from the `AFTER_DAMAGE` handler in `SurvivalMod.java` (only ServerPlayer →
  ServerPlayer damage tags both).

## Boss bar (2026-06-21)
- Replaced the old once-a-second `⚔ Combat: Xs` action-bar line with a **red draining
  `ServerBossEvent`** ("⚔ In Combat — Xs"), `bars: Map<UUID,ServerBossEvent>`.
- `tag()` creates/refreshes the bar and adds the player as a viewer instantly.
- `tick(server)` (wired every server tick in `SurvivalMod.java`) drains `progress = remain/total`,
  re-adds the player after a relog, and drops the bar when the tag expires or the owner logs off.
- ⚠️ MC 26.2: `ServerBossEvent` constructor needs a **`UUID` first arg**
  (`new ServerBossEvent(UUID.randomUUID(), name, BossBarColor.RED, BossBarOverlay.PROGRESS)`).

## What the tag gates
- **RTP** (`rtp/RandomTeleport.java`) — blocked while tagged (gating otherwise unchanged this round).
- **/back, /home** (`HomeManager`) — cancelled on damage.
- **TPA, including auto-accept** (`tpa/TpaManager.java`) — see below.

## TPA combat gate + accept sound (2026-06-21)
- Manual `/tpaccept` already refused while either player was tagged. Added the same gate to
  **auto-accept** in `TpaManager.request()` (previously a combat-log escape hatch). Shared helper
  `blockedByCombat(server, a, b)` used by both `accept()` and the auto-accept branch.
- Teleport plays a destination whoosh (`ENDERMAN_TELEPORT`) **plus** an accept chime
  (`NOTE_BLOCK_PLING`) at the destination, where both players stand — so a popular target hears it on
  every warp in (intended). `ServerPlayer.playNotifySound(...)` does **not** exist in 26.2; use a
  positional `level.playSound(null, x,y,z, ...)`.

## Related
[[shop-catalog]] · [[sell-and-economy]] · [[trial-item-expiry]]
