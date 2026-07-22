# Mythoria — Hero Progression

## Quality ladder
9 tiers, upgraded via `/upgrade`:
```
Dead → Broken → Poor → Normal → Fine → Excellent → Masterwork → Legendary → Perfect
```

| Quality | Stat Mult | Upgrade Cost (coins) | Stones | Essence | Success Rate |
|---------|-----------|---------------------|--------|---------|-------------|
| Dead | ×0.0 | 500 | 1 | 0 | 95% |
| Broken | ×0.5 | 1,000 | 2 | 0 | 85% |
| Poor | ×0.7 | 2,500 | 3 | 10 | 75% |
| Normal | ×1.0 | 5,000 | 5 | 25 | 65% |
| Fine | ×1.2 | 10,000 | 8 | 50 | 50% |
| Excellent | ×1.5 | 25,000 | 12 | 100 | 35% |
| Masterwork | ×1.8 | 50,000 | 18 | 200 | 20% |
| Legendary | ×2.0 | 100,000 | 25 | 500 | 10% |
| Perfect | ×2.2 | — | — | — | 0% (max) |

## Upgrade flow
1. **Queue**: `/upgrade <hero-id> [use-orb]` → adds job to `upgradeQueue` (BullMQ)
2. **Worker**: `UpgradeWorker` processes asynchronously:
   - Validates ownership and quality (not at Perfect max)
   - Checks sufficient coins/stones/essence (+ celestial orb if flagged)
   - Consumes resources before the roll (gambler's commitment)
   - Rolls success: base rate + 15% if using celestial orb
   - **Success**: promotes to next quality level, regenerates hero ID
   - **Failure w/ orb**: protects hero (no penalty, no change)
   - **Failure w/o orb**: 5% destroy hero, 50% downgrade 1 level, 45% keep quality
3. **Notification**: worker sends DM with coloured embed (green=success, red=destroyed, orange=failed)

## Celestial Orb
- Craftable via `/craft` (Dragon Scale×1 + Star Dust×3 + Relic Shard×2, 500 coins, 50 essence)
- Provides: +15% success rate + full failure protection (no destroy/downgrade)

## Merge (`/merge`)
2 Excellent-quality heroes → 1 hero with +1★ (max ★★★★★)

**Requirements:**
- Same template ID (same hero type)
- Both Excellent quality
- Both same star count
- Under 5★
- Both owned by same user
- Neither locked (trading/auction/expedition)

**Result:** both source heroes destroyed, new hero created with increased star count

## Disassembly (`/disassemble`)
Salvage a hero for resources. Formula:
```
coins = baseValue × qualityMult × (1 + level × 0.02)
essence = qualityEssenceValue × qualityMult
```

## Hero leveling
- XP earned via expedition battles
- Curve: `100 × level^1.5` XP per level
- Max level: 60
- Stat growth: `+10% of base stat per level`

## GRAPH METADATA
- cluster: Mythoria
- node_type: system
- importance_level: 0.85
- hub_node: false
- tags: #mythoria #upgrades #progression
