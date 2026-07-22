# Moderation: Ranks, Bans & Mutes

Part of [[Himawari-SMP]].

## Two-rank model (2026-06-22)
- Assignable ranks are now **owner + mod** only (player = default, no tag). `rank/RankConfig` defaults
  dropped the `admin` tier; `rank/PermissionManager` still resolves `smp.owner > smp.mod > smp.player`
  (the `smp.admin` node remains for back-compat but isn't a tier).
- **Owner-without-OP**: new `ModCommands.isOwnerSource` = OP **or** `smp.owner`. Owner-tier commands use
  it so the owner rank works without OP: `/rank`, `/revenue`, `/economy`, `/spawner give`,
  `/mapart delete`. (`/version` **and** `/cash` stay owner-**with-OP** only.) `/revenue` moved from
  `smp.admin` → owner.
- Staff commands regated to `smp.mod` (mods + owner): `/mute`, `/unmute`, `/ban`, `/unban`, `/admin`,
  `/balance <other>`.

## Bans (mod-enforced, new)
- `moderation/BanManager` — in-memory `uuid→name`, **local `<world>/survivalmod_bans.json` primary**,
  mirrored to Supabase `banned_players`. Enforced by `handler.disconnect(...)` in the
  `ServerPlayConnectionEvents.JOIN` handler (`SurvivalMod.java`).
- `/ban <player> [reason]` (kicks if online), `/unban <name>` (by name → works offline).

## Mutes
- `moderation/MuteManager` moved from a local UUID list to `uuid→name` map, **local JSON primary +
  Supabase `muted_players` mirror** (boot-merge). Chat still blocked via `ALLOW_CHAT_MESSAGE`.
  `/mute <player>`, `/unmute <player>`.

## Tables (Supabase, service-key writes / anon read)
`banned_players`, `muted_players`, `investigations` — see `docs/supabase/schema_moderation.sql` and
[[shared-supabase]]. The mod is the live primary; the tables are a shared/manageable mirror.

## Related
[[admin-investigator]] · [[Himawari-SMP]] · [[shared-supabase]]
