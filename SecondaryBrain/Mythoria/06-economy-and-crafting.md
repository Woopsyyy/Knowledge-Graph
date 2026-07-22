# Mythoria — Economy & Crafting

## Currency
- **Coins** — primary currency, earned via expeditions, mining/fishing, disassembly, daily/weekly/monthly, invite bonuses
- **Essence** — premium currency, earned via disassembly (higher quality = more essence), monthly claims

## User claims
| Type | Cooldown | Base Reward | Invite Bonus |
|------|----------|------------|-------------|
| Daily (20h) | 24h | RNG loot | +100 coins per active invite + bonus loot tier |
| Weekly | 7d | 1,500 flat coins | — |
| Monthly | 30d | 8,000 coins + 50 essence | — |
| Starter Kit | One-time | 5 random Common heroes (1–3★) + starter items | — |

### Invite bonus on daily
Active invites (players who joined via your link and stay in the production guild) stack:
- **Per invite**: +100 coins to daily reward
- **Tiers**: 1 invite = Common, 3 = +Uncommon, 5 = +Rare, 10 = +Epic, 20 = +Legendary, 50 = +Celestial materials

## Items (80+ seeded)
### Material ladder — 8 families × 6 rarities = 48 items
| Family | Common | Uncommon | Rare | Epic | Legendary | Celestial |
|--------|--------|----------|------|------|-----------|-----------|
| Stone 🪨 | Stone | Polished Stone | Enchanted Stone | Mythril Stone | Dragon Heartstone | Star Stone |
| Ore ⛏️ | Raw Ore | Fine Ore | Enchanted Ore | Mythril Ore | Dragon Vein Ore | Starforged Ore |
| Wood 🪵 | Timber | Fine Timber | Enchanted Wood | Mythril Timber | Ancient Heartwood | Celestial Timber |
| Cloth 🧵 | Linen | Quality Cloth | Enchanted Cloth | Exquisite Silk | Dragonweave | Star Silk |
| Fish 🐟 | Minnow | Silverfin | Enchanted Fish | Mythril Carp | Ancient Leviathan | Starspawn |
| Gem 💎 | Pebble | Cut Gem | Enchanted Gem | Mythril Gem | Dragon Eye Gem | Star Gem |
| Scale 🦎 | Lizard Scale | Fine Scale | Enchanted Scale | Exquisite Scale | Dragonscale Plate | Celestial Scale |
| Relic 📜 | Old Relic | Fine Relic | Enchanted Relic | Exquisite Relic | Ancient Relic | Star Relic |

5-tier upgrade recipes per family (Common→Uncommon→Rare→Epic→Legendary→Celestial), each requiring 5× previous tier + coins/essence.

### Gear sets — 3 sets × 3 slots × 6 rarities = 18 items
| Set | Theme | Slots | Primary Materials |
|-----|-------|-------|-------------------|
| Dawn Guard | Holy Knight | Sword, Helm, Armor | Stone + Ore + Gem |
| Shadow Ward | Dark Rogue | Dagger, Hood, Cloak | Wood + Cloth + Gem |
| Frost Weaver | Ice Mage | Staff, Crown, Robe | Wood + Fish + Scale |

Each slot has 6 rarity tiers (Common→Celestial). Stats double per tier. 54 crafting recipes total (3 sets × 3 slots × 6 rarities).

### Tools — 2 types × 6 rarities = 12 items
| Tool | Slot | Use | Durability | Crafting Materials |
|------|------|-----|-----------|-------------------|
| Pickaxe ⛏️ | Weapon | Mining | Common 12 → Celestial 720 | Stone + Wood |
| Fishing Rod 🎣 | Weapon | Fishing | Common 12 → Celestial 720 | Wood + Fish + Cloth |

Tools equip on the team leader's hero weapon slot. Higher rarity = more durability (Common ~1h, Celestial ~5 days) and better loot table.

### Other items
| Item | Rarity | Type |
|------|--------|------|
| Bandage | Common | consumable |
| Upgrade Stone | Rare | material |
| Potion | Uncommon | consumable |
| Ether | Uncommon | consumable |
| Resurrection Orb | Epic | consumable |
| Celestial Orb | Celestial | upgrade_protection |

## Crafting system
80+ recipes loaded from `crafting_recipes` DB table (fallback to hardcoded constants). Categories:
- **Material upgrades**: 8 families × 5 tiers = 40 recipes
- **Gear crafting**: 3 sets × 3 slots × 6 rarities = 54 recipes
- **Tool crafting**: pickaxes (6) + rods (6) = 12 recipes
- **Consumables**: bandage, potion, upgrade stone, celestial orb

Supports batch crafting via `quantity` parameter. Crafting costs coins + optional essence.

## Inventory system
- `inventory` table: stackable items (upsert pattern, auto-delete at 0)
- `equipment` table: individual instances with stats JSON
- Equipment slots: weapon, helmet, armor, boots, gloves, accessory, artifact (7 total)
- Redis cache: inventory cached 60s, invalidated on mutations

## Economy flows
```
Expeditions ──→ coins, items, hero drops (auto-loot)
Mining ───────→ ores, gems, stones + coins (tool durability-based)
Fishing ──────→ fish, scales, cloth + coins (tool durability-based)
Disassembly ──→ coins + essence (quality-scaled)
Upgrades ─────→ consumes coins + upgrade stones + essence
Crafting ─────→ consumes materials + coins → items/equipment
Auctions ─────→ coins exchange between players (escrow pattern)
Trading ──────→ direct hero swaps (no currency)
Invites ──────→ daily reward bonus (coins + loot) per active referral
```

## GRAPH METADATA
- cluster: Mythoria
- node_type: system
- importance_level: 0.9
- hub_node: false
- tags: #mythoria #economy #crafting
