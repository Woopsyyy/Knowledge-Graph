# Mythoria — Card System

## Hero types (Rarity / Card Type)
| Type | Stat Multiplier | Description |
|------|----------------|-------------|
| Common | ×1.0 | Baseline |
| Uncommon | ×1.5 | Slightly stronger |
| Rare | ×2.0 | Notable power boost |
| Epic | ×2.8 | Substantial stats |
| Legendary | ×4.0 | Extremely powerful |
| Celestial | ×6.0 | Overpowered |

## Hero dimensions
| Dimension | Range | Effect |
|-----------|-------|--------|
| Stars (★) | 1–5 | Multiplier via `STAR_MULTIPLIERS`: ★1=×0.4, ★2=×0.6, ★3=×0.8, ★4=×1.0, ★5=×1.2 |
| Quality | Dead→Perfect | Multiplier vector: Dead=×0.0, Broken=×0.5, Poor=×0.7, Normal=×1.0, Fine=×1.2, Excellent=×1.5, Masterwork=×1.8, Legendary=×2.0, Perfect=×2.2 |
| Tier | I→Ascended | Ascended=×3.0 scaler |
| Level | 1–60 | XP curve: `100 × level^1.5` |
| Health | Injured→Healthy | ×0.7→×1.15 starting stat variant |

## Stat calculation
```ts
growthRate = baseStat * 0.1  // 10% of base per level
stat = round((baseStat + (level-1) * growthRate) * qualMult * tierMult * starMult)
power = round((hp/5 + attack + defense + magic + magicDefense) * tierMult * qualMult * stars)
```

## Card image generation
`ImageService.generateCardBuffer()` fetches `template.artworkPath` from local Nginx (port 3002), resizes to 400×480, extends to 400×560 with dark padding, and composites an SVG overlay with: name, rarity badge, quality + stars, serial, level, tier, and power. Output: 400×560 PNG via Sharp.

Gallery-quality borders coloured by quality (Dead=black → Perfect=pink). Used by `/inspect` as `attachment://card.png` in the embed.

`ImageWorker.ts` no longer renders images — just copies `template.artworkPath` to `hero.imageUrl` in the database.

## Artwork bucket structure
Artwork stored at Nginx-served path `artwork/{rarity}/{id}.webp` (container port 3002):
- `rarity` folder: `common/`, `uncommon/`, `rare/`, `epic/`, `legendary/`, `celestial/`
- Same character can have multiple rarity variants (e.g., Gojo at Common, Rare, and Celestial each with different art)
- Path is set on the template's `artwork_path` field
- Fallback: templates without artwork are filtered out by `/kit` and card generation

## Hero minting
`CardService.mintHero(ownerId, templateId, rarity, quality)` (kingdom is already assigned on the template):
- Creates hero instance with random traits (10% two, 50% one, 40% none)
- Traits pool: Brave, Lucky, Resilient, Swift, Focused — applied in battle
- Auto-assigns serial number (sequential per hero)
- 6 equipment slots created empty (weapon, helmet, armor, boots, gloves, accessory, artifact)

## Kingdom
Kingdom is randomly assigned on card import (no longer derived from class). The 5 kingdoms: Eldoria, Frosthold, Silvercrest, Ashen Dominion, Celestial Empire.

## Disassembly
`CardService.disassembleHero(ownerId, heroId)`:
- Destroys hero card
- Returns coins + essence based on formula: `baseValue × qualityMult × (1 + level × 0.02)`

## Merge
`MergeService.mergeHeroes(ownerId, heroId1, heroId2)`:
- Requires both heroes: owned by same user, Excellent quality, same template ID, same star count, under 5★
- Destroys both source heroes
- Creates new hero with +1 star (★★★★ → ★★★★★)
- Cannot merge heroes that are locked (trading/auction/expedition)

## Stat formulas used
```ts
// Power rating
power = (hp/5 + atk + def + mag + mdef) * tierMult * qualMult * starCount

// Growth per level
growthRate = baseStat * 0.1

// Final stat
stat = (baseStat + (level-1) * growthRate) * qualMult * tierMult * starMult
```

## GRAPH METADATA
- cluster: Mythoria
- node_type: system
- importance_level: 0.9
- hub_node: false
- tags: #mythoria #cards #stats
