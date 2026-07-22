# Shop Catalog (survival buy/sell tabs)

Part of [[Himawari-SMP]].

## What it is
The creative-menu-style survival shop tabs added 2026-06-21: **Building Blocks, Mob Drops, Farms,
Resources, Other**. Every catalog item is both buyable and sellable, priced by rarity × demand.
These sit alongside the curated tabs (gear, utilities, trial, potions, spawners).

## Data model — one generic table
- New Supabase table **`public.shop_catalog`**: `category, item_id, buy_price, sell_price, sort_order`,
  PK `(category, item_id)`. One table → any number of categories with no per-category mod code.
- Fetched in `SupabaseClient.fetchShopPrices()` via one loop
  (`/shop_catalog?...&order=category.asc,sort_order.asc`), tagged with the row's own `category`.
  The fetch is **tolerant**: a missing table (HTTP error) is caught so the rest of the shop still loads.
- `RemoteConfig.assemble()` + `ShopPrices.buildIndex()` already bucket by category and build the flat
  `item_id → Price` index — no change needed. `ShopManager.buyEntries(category)` returns buyable rows
  for any category; `ShopMenu.display()` adds the `Sell:` lore line when `sell_price != null`.

## GUI (`gui/ShopMenu.java`)
- Five new categories added to the `TABS` set; top-choice menu re-laid as **two rows of five** icons
  (slots 11–15 specials, 29–33 catalog). Each `select(category)` on click.
- Command shortcuts `/building /mobdrops /farms /resources /other` (the `tabCommands` loop in
  `command/ModCommands.java`).

## Pricing & the override trap
- Rarity tiers for Resources: coal/copper < iron < gold/redstone/lapis < quartz/amethyst < emerald <
  diamond. Mineral block ≈ 9× the unit. `sell ≈ 30–35%` of buy (anti-arbitrage; `EconomyValidator`
  also guards craft arbitrage).
- ⚠️ **The flat index resolves duplicate `item_id`s to whatever category is fetched LAST — and
  `shop_catalog` is fetched last.** So a catalog row silently overrides the buy/sell of the same item
  in `gear`/`utilities`/`sell_prices`. Rules followed when seeding:
  - Excluded items already in curated buy tabs (gear/tools/elytra, netherite ingots/scrap/debris,
    totems, shulkers, end crystals, golden apples, potions, spawners) + anything
    `UnobtainableItems.notSurvivalObtainable()` rejects.
  - For items that overlap `sell_prices`, the catalog `sell_price` was set **equal** to the curated
    value (continuity), bumping `buy_price` to ≥ 3× sell to keep buy > sell.
  - Post-seed audit must read `0` for: utilities/gear overlap, sell mismatch, and `buy_price <= sell_price`.

## Key files
- `supabase/SupabaseClient.java` (fetch loop), `supabase/SupabaseRealtime.java` (subscribes
  `shop_catalog` so live price edits push), `gui/ShopMenu.java`, `command/ModCommands.java`.
- Seed + DDL: `SMP/docs/supabase/seed_shop_catalog.sql` (idempotent `create table if not exists` +
  RLS anon-read + realtime + `insert ... on conflict do update`). Seeded live via the Supabase MCP
  to project `tdmzxxyctqnxxkdulvar` ("Mod"), 360 rows (incl. concrete-powder ×16, `powder_snow_bucket`,
  and the netherite raws ancient_debris/scrap/ingot/block in Resources — armour/tools stay in /gear).

## Related
[[combat-status]] · [[sell-and-economy]] · [[trial-item-expiry]]
