# Bounties

Part of [[Himawari-SMP]]. Pooled PvP bounties: anyone stakes cash on a target; whoever kills them
collects the whole pool.

- `bounty/BountyStore` — `uuid → {name, amount}`, persisted to `<world>/survivalmod_bounties.json`
  (local-only, no Supabase). `add(target, name, amount)` grows the pool; `claim(target)` returns +
  clears it; `top()` is sorted desc; `amountOf(target)`.
- `/bounty <player> <amount>` (`smp.player`) — debits the poster (`economy().tryDebit`), adds to the
  pool, broadcasts. `/bounty` with no args opens `gui/BountyMenu`.
- `gui/BountyMenu` — player heads, highest bounty first, name = pooled amount. Copies the
  [[Himawari-SMP|BalTopMenu]] head pattern (`PLAYER_HEAD` + `ResolvableProfile`). Display-only.
- **Payout**: the `ServerLivingEntityEvents.AFTER_DEATH` handler (`SurvivalMod.java`) — if a PvP
  victim has a bounty, the killer gets `bounties().claim(victim)` and it's broadcast.
- Amounts accept k/m/b/t suffixes via `economy/Amounts.parse` (shared with `/pay` and `/cash`).

## Related
[[sell-and-economy]] · [[Himawari-SMP]]
