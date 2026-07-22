# Mythoria — Trading & Auctions

## Peer-to-peer trading (`/trade`)
**Gate:** `dev: true`

### Propose
`/trade propose <player> <offered-hero> <requested-hero>`
- Validates: both heroes owned by respective parties, not locked (trade/auction/expedition)
- Creates `trades` row with status `pending`
- DMs recipient with Accept/Reject buttons (`trade_accept` / `trade_reject`)

### Accept
`trade_accept` button handler:
- Re-validates locks and ownership (hero may have been traded away in the meantime)
- Removes both heroes from any teams (`team_members` cleanup)
- Swaps `ownerId` on both heroes
- Updates trade status → `accepted`

### Cancel/Reject
- Proposer cancels via `/trade cancel <trade-id>`
- Receiver clicks `trade_reject`
- Status set to `cancelled` / `rejected`

### Lock checking
`isHeroLocked()` utility checks: pending trades, active auctions, active expeditions. Prevents trading/auctioning/merging a hero that's in use.

## Auction house (`/auction`)
**Gate:** `dev: true`

### Create listing
`/auction create <hero-id> <starting-bid> [min-increment]`
- 30-day duration
- Validates hero ownership, lock status, no duplicate listing
- Status: `active`

### Bid
`/auction bid <auction-id> <amount>`
- Minimum bid: `currentBid + minIncrement` (or `startingBid` if no bids)
- Deducts bid amount from bidder immediately (escrow pattern)
- Refunds previous bidder the full amount
- Updates `currentBid` and `currentBidderId`

### List
`/auction list [hero-name] [quality] [page]`
- Paginated with ILIKE name filter and quality filter
- Shows: hero name, quality, level, tier, power, current bid, seller
- JOINs heroes → templates for display data

### Cancel
`/auction cancel <auction-id>`
- Only allowed if no bids placed yet (protects bidders)

### Expiry processing
`SchedulerWorker.ts` calls `AuctionService.processExpired()` hourly:
- Scans `auctions` WHERE `expiresAt < now AND status = 'active'`
- **Winning bid**: transfers hero to bidder, coins to seller, sends DMs to both
- **No bids**: unlocks hero for seller, sends DM
- Sets status → `completed` or `expired`

## GRAPH METADATA
- cluster: Mythoria
- node_type: system
- importance_level: 0.8
- hub_node: false
- tags: #mythoria #trading #auctions
