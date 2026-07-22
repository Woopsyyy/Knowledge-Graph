# Trial Item Expiry & Destruction

Part of [[Himawari-SMP]].

## What it is
Trial tools are pre-enchanted netherite tools sold whole in `/shop` (`ShopManager.buildSpecial`).
Each carries a trial type + absolute expiry timestamp in `CUSTOM_DATA` (see `TrialBooks`). The tool
*is* the 24h purchase, so on expiry the **entire item** is destroyed, not just its effect.

## Key files
- `trial/TrialBooks.java` — type/expiry accessors; `isActive()`, `isExpired()` (expiry `0` /
  `Long.MAX_VALUE` = permanent/legacy, never destroyed), countdown lore.
- `trial/TrialManager.java` — the break effects + the expiry sweeps.
- `mixin/ChunkMapAccessor.java` — `@Accessor("visibleChunkMap")` to enumerate loaded chunks
  (no public API for this in MC 26.2). Registered in `survivalmod.mixins.json`.
- `SurvivalMod.java` — tick wiring (`END_SERVER_TICK`).

## Behaviour (2026-06-20 change)
- **Every 1s** (`tickExpiry`): sweep each online player's inventory + ender chest. Expired tools are
  removed outright and the owner is notified (red chat line per tool + `ITEM_BREAK` sound). Survivors
  get their countdown lore refreshed (`refreshLore`).
- **Every 30s** (`tickWorldContainers`): walk every `ServerLevel`'s loaded chunks, sweep each
  `Container` block entity (chests, barrels, hoppers, droppers, dispensers, placed shulkers). Silent.

## 2026-06-21 fix — chests in non-ticking chunks
- The original `tickWorldContainers` used `ChunkHolder.getTickingChunk()`, which is **null for loaded
  but non-ticking chunks** (within view distance, beyond simulation distance) — chests there were
  never swept ("expired tool won't die in a chest"). Now uses
  `level.getChunkSource().getChunkNow(holder.getPos().x(), holder.getPos().z())` → any FULL-status
  loaded chunk, no forced generation. (`ChunkHolder.getFullChunk()` does **not** exist in MC 26.2;
  `ChunkPos` exposes record-style `x()`/`z()`, fields are private.)
- `refreshLore` was generalised from `Inventory` to `Container` and is now also called on the
  player's ender chest, so ender-chest trial tools show their countdown too.
- **Nested**: `sweepNested` recurses into shulker-box items (`DataComponents.CONTAINER` /
  `ItemContainerContents.fromItems`) and bundles (`BUNDLE_CONTENTS` / `new BundleContents(...)` with
  `ItemStackTemplate.fromStack`). One reusable `sweepContainer` is used everywhere.

## Fortune ↔ Silk Touch toggle (2026-06-22)
Three special tools are **toggleable** — right-click to swap their enchant set in place:
**3×3 Pickaxe** (`pick_33`), **Ore Miner** (`ore_miner`), **3×3 Shovel** (`shovel_33`). The Tree Feller
axe, plain Silk pickaxe/shovel, and Auto-Replant hoe are **not** toggleable.

- **Flag**: stamped at purchase in `ShopManager.buildSpecial` via `TrialBooks.setToggleable(tool, FORTUNE)`.
  Stored in `CUSTOM_DATA`: `survivalmod_trial_toggle` (bool) + `survivalmod_trial_mode` (`"fortune"`/`"silk"`).
  **Can't be retrofitted** onto a tool already in someone's inventory — only fresh purchases carry it.
- **Effect** (`TrialManager.toggleEnchant`): `flipMode` flips + persists the mode, then `Enchants.apply`
  **replaces** the whole enchant set (eff5/unbr3/mend1 + `fortune3` *or* `silk_touch1` — never both, since
  `apply` rebuilds from `EMPTY`). Renames the item `… (Fortune)`/`… (Silk Touch)`, sends an aqua "Switched
  to …" chat line, plays `AMETHYST_BLOCK_CHIME`.
- **Wiring** (`SurvivalMod`): `UseItemCallback` (in-air — Java sends the packet, Bedrock/Geyser often does
  not) **and** `UseBlockCallback` (right-click a block; a non-sneak click on a *container* block passes
  through so chests/furnaces still open — sneak to toggle on those). Per-player 5-tick debounce
  (`lastToggleTick`) absorbs the Geyser double right-click packet that would otherwise flip twice.

## Gotchas
- ⚠️ **Toggle silently won't fire while you hold food (any `CONSUMABLE`) in your OFF-hand.** The
  `eatingOffhand(sp, hand)` guard in `SurvivalMod` returns true for a MAIN_HAND interaction whenever the
  offhand item is consumable, so both interaction callbacks `PASS` (to let you eat). By design, but a real
  UX trap — confirmed 2026-06-22 via `[trial-diag]` logging (the toggle code itself was correct all along).
- `SoundEvents.ITEM_BREAK` is a `Holder.Reference<SoundEvent>` → call `.value()` for `playSound`
  (unlike `AMETHYST_BLOCK_CHIME` which is a raw `SoundEvent`).
- Enumerate loaded chunks via `getChunkNow` (FULL status), **not** `getTickingChunk()` — the latter
  silently skips loaded-but-non-ticking chunks (see the 2026-06-21 fix above).
- Component "container" streams (`nonEmptyItemCopyStream`, `itemCopyStream`) return **copies** —
  rebuild the component from the filtered list to persist removals.

## Spec
`SMP/docs/superpowers/specs/2026-06-20-trial-item-destruction-design.md`
