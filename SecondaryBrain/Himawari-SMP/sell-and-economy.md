# Sell Commands & Economy

Part of [[Himawari-SMP]].

## Sell commands (2026-06-21 rework)
- **`/sell`** — sells only the **held stack** (`ShopManager.sellHeldStack`): whole stack in the main
  hand, not the inventory.
- **`/sellall`** — sells **every stack of the held item's type** (`ShopManager.sellAllHeld` → loop):
  hold one slime, sell all slime. Refuses spawners (each can be a different mob/refund — use `/sell`).
- The old whole-inventory liquidation (`sellInventory`) was **removed**.
- Both use `realSellPrice(stack)`: explicit catalog/`sell_prices` value or a spawner's refund, else
  `null`. So gear, tools, trial tools, custom potions and unpriced junk are **refused** — nothing
  valuable is liquidated for a fallback. (`sellPriceOf` adds the spawner/trial/potion guards.)
- Wiring in `command/ModCommands.java` (`/sell`, `/sellall`). The in-GUI `sellOneOf`/`sellAllOf`
  sub-menu still exists.

## Price sources
- All prices live in **Supabase** (no code defaults; empty shop if unreachable). Tables: `gear`
  (buy), `utilities` (buy+sell), `trial`/`potions` (buy), `sell_prices` (sell-only), and the new
  `shop_catalog` (buy+sell, many categories — see [[shop-catalog]]).
- Flat `item_id → Price` index built by `ShopPrices.buildIndex()`; duplicates resolve to the
  last-fetched table (`shop_catalog`). `EconomyValidator` runs every 5 min to flag craft arbitrage.

## Related
[[shop-catalog]] · [[combat-status]]
