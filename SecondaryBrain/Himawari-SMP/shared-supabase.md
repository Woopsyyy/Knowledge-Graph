# Shared Supabase (the bridge)

Part of [[Himawari-System]]. Both the bot and the mod talk to **one** Supabase project —
**"Mod"**, ref `tdmzxxyctqnxxkdulvar` (region ap-southeast-1, Postgres 17). It is the bridge between a
Minecraft account and a Discord account, and the mod's live-config + backup store.

## How each side connects
- **Mod (Java)**: REST (PostgREST) via `supabase/SupabaseClient.java`; config seam
  `supabase/RemoteConfig.java` (`loadShop/loadRtp/loadModConfig`, code-default fallback on failure);
  live pushes via `supabase/SupabaseRealtime.java` (Phoenix WS → debounced `refreshAll()`). Anon key
  for reads; **service key** only for backups + linking writes. Settings in
  `config/survivalmod/supabase.json`.
- **Bot (Node)**: `@supabase/supabase-js` as `client.db` (`utils/Supabase.js`), **service key**
  (bypasses RLS). Reads/writes link, ticket, embed, and event-message tables.
- RLS: shop/config tables have an `anon read` policy (mod reads with anon key); backup + linking
  tables have **no anon policy** — service-key only.

## Table catalog (who reads / writes)
**Linking** (both sides): `link_codes` (pending codes, origin mc/discord, 10-min expiry),
`linked_accounts` (`discord_id` PK ↔ `minecraft_uuid`/`name`, `nick_synced`), `link_config`
(per-guild linked role).
**Shop / economy** (mod reads): `gear`, `utilities`, `sell_prices`, `trial`, `potions`, and
`shop_catalog` (the survival tabs — see [[shop-catalog]]). Legacy `shop_prices` is empty/unused.
**Mod config** (mod reads, anon): `features` (TPA/RTP/combat/homes toggles & tunables), `rtp_config`
(per-dimension ranges), `spawners` + `spawner_config`, `mod_config` (misc JSON rows: ranks, scoreboard,
trial, economy, discord, …), `market`.
**Backups** (mod writes, service key): `auction_listings` + `auction_meta`, `order_listings`,
`map_arts` (+ storage bucket for map art).
**Moderation mirror** (mod local-primary, service-key writes, anon read at boot): `banned_players`,
`muted_players`, `investigations` — see [[moderation-bans]] / [[admin-investigator]] and
`docs/supabase/schema_moderation.sql`.
**Bot-owned** (bot writes, service key): `tickets` + `ticket_config`, `embeds`,
`guild_event_messages` (greet/leave/boost).

## Account-linking flow
| Start in Discord | Start in-game |
|---|---|
| `/link start` → code | `/link` → code (`link/LinkManager.java`) |
| in-game `/link <code>` | Discord `/link enter <code>` |
| ✅ nickname → `Name ⛏` (+ linked role) | ✅ same |

Whoever **enters** a code writes `linked_accounts` with `nick_synced=false`; the bot applies the
nickname/role immediately (if it processed the code) or within ~15s via the `events/ready.js` poll
(`utils/NickSync.js`). `/unlink` reverses it. Mod side: `link/LinkManager.java` + `SupabaseClient`
link methods. Cross-platform doc: `docs/linking.md`.

## ⚠️ Rule
A schema change touches **both** projects — update `docs/` (the in-repo source of truth) and
coordinate the mod + bot in the same change. Shop/config edits propagate live to the mod through
`SupabaseRealtime` (subscribed tables include `shop_catalog`).

## Related
[[Himawari-System]] · [[Himawari-Bot]] · [[Himawari-SMP]] · [[shop-catalog]]
