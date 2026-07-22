# Auction House & Buy Orders

Part of [[Himawari-SMP]]. Two player-to-player markets that share one design language and the same
Supabase-backup model (world NBT is the live primary; an off-thread mirror restores on a wiped world).

## Auction house (`auction/`)
Sellers list a real item for a set price; buyers pay and receive it. 30-day expiry, then the seller
reclaims from "Your Items". % listing fee up front → a revenue pool the owner claims.
- `AuctionStore` — `browse/bySeller`, `buy`, `reclaim`, `list` (legacy held-item path), persistence +
  Supabase mirror (`survivalmod_auction.dat`).
- `AuctionListing` — `{id, seller, sellerName, item (ItemStack, count = qty), price (total), listedAt}`.
- GUI: `gui/AuctionMenu` (browse/sort/search/buy, "Your Items" to reclaim).

## Create-listing wizard (2026-06-21, mirrors the orders wizard)
Before this, listing was command-only (hold item → `/auction sell <price>`); the GUI had no create
button. Added a **Create Listing** button (slot 46, writable book) → a 4-step wizard, matching the
buy-order wizard's look/flow exactly:
1. `gui/AuctionItemPickerMenu` — pick from **your inventory** (distinct item+component stacks,
   aggregated, showing how many you hold). *This is the one deviation from orders, which picks from the
   full catalog — you can only auction what you own.*
2. `gui/AuctionAmountMenu` — +/- amount, capped at `min(available, maxStackSize)` (a listing is one
   stack, like the held-item path).
3. `gui/AuctionPriceMenu` — price **per item** (+/- buttons); total = amount × price.
4. `gui/AuctionReviewMenu` — amount / price-each / total / fee / balance; change-item/amount/price,
   cancel, or **List Item**.
- Backend: `AuctionStore.Draft {item, quantity, price}` + `newDraft/draft/clearDraft` (same pattern as
  `OrderStore`); `available(seller, type)` counts matching `isSameItemSameComponents` in slots 0–35;
  `createFromDraft(...)` re-validates stock, charges the % fee on the total, `removeMatching` pulls the
  items, and stores a listing with `item.count = quantity`, `price = total`. Sounds on success/fail.

## Buy orders (`order/`) — the design that was mirrored
"I want N of an item at price-each", escrowing the total up front; sellers fill from inventory, buyer
claims. Wizard: `OrderMenu` → `OrderItemPickerMenu` → `OrderAmountMenu` → `OrderPriceMenu` →
`OrderReviewMenu` (+ `MyOrdersMenu`), draft in `OrderStore`. Orders never expire.

## Gotchas
- Auction listing is **one stack max** — keeps `AuctionListing.item` a single valid stack so `buy()`'s
  `giveOrDrop` never has to split a >maxStack count.
- Picker/remove match by `isSameItemSameComponents` (component-aware) so an enchanted/renamed item
  lists exactly itself, not a plain variant.

## Related
[[Himawari-SMP]] · [[sell-and-economy]] · [[shared-supabase]]
