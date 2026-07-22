# Mythoria — Expeditions & Battle System

## Overview
Automated dungeon-crawling system inspired by SAO's Aincrad. Players send teams on tick-based expeditions that auto-battle through floors (1–600), earning XP, coins, items, and hero drops.

## Expedition lifecycle
1. **Start** — `/mexplore start <team>`
   - Anti-spam cooldown (3s Redis key)
   - Validates: no active expedition, team exists, heroes fit (not resting/injured)
   - Calculates starting floor: `Math.floor(teamPower / 10)`, clamped 1–600
   - Shows tower image from `mythoria-assets/Environment/Tower/tower.png` in embed
   - Selects a random region from the 5 available in the current floor tier (queries DB by `floor_min`/`floor_max`)
   - Creates DB record + Redis cache, queues first BullMQ tick (300s delay)
2. **Tick** — `ExpeditionService.processExpeditionTick()` (every 5 min)
   - Checks expedition still active, filters injured heroes
   - Generates floor-appropriate enemies (scaling stats: base × (1 + (floor-1) × 0.04))
   - 12 enemy types (Slime→Celestial Warden, cycling every 50 floors), 6 boss types (Troll King→Celestial Tyrant, cycling every 100 floors)
   - 15% boss tick chance; Red Gate chance scales with floor
   - Delegates combat to `BattleService.resolveBattleRound()`
   - Applies XP/leveling (curve: `100 × level^1.5`, max 60), stamina/HP updates
   - Accumulates loot; mints dropped hero cards
   - Updates DB + Redis, queues next tick if heroes survive
3. **Stop** — `/mexplore stop`
   - Sets expedition status, applies 2h resting cooldown to heroes (Redis TTL)
   - Clears expedition cache

## Battle engine (`BattleService.ts`)
Turn-based combat (max 10 rounds per tick):
1. Calculate active stats via `CardService.calculateStats()`
2. Combat loop:
   - All heroes attack using hybrid damage formula: `attackPower + magicPower` (Trait modifiers: Brave +15% dmg, Lucky 15% chance +50% crit)
   - Victory check → enemy attacks random target
3. Post-battle: stamina decay (6 normal / 15 Red Gate; Resilient reduces 40%), health status calc
4. Loot generation: coins, XP, items, hero cards (scales with floor/gate, capped 15%)
5. Boss ticks: 2.5× reward; Red Gate: 3.0× reward
6. **Floor 200+ gear drops**: equipment drops unlock at higher floors with scaling rarity

## Contribution tracking
Each tick tracks per-hero contribution by system:
- **Expeditions**: damage dealt per battle logged per hero; `/mexplore status` and `/mexplore stop` display sorted highest→lowest with 🥇🥈🥉 rankings
- **Mining**: items mined per hero tracked; `/mmine status`/`stop` shows ranked leaderboard
- **Fishing**: fish caught per hero tracked; `/mfish status`/`stop` shows ranked leaderboard

## Red Gates
Random events that dramatically increase difficulty and rewards:
- **Floors 1–99**: 0% chance (locked)
- **Floors 100–399**: 0.5% chance (super rare)
- **Floors 400–600**: 5–30% chance (scales `0.05 + (floor-400) × 0.0025`)
- Enemies have 2.5× stats, reward multiplier 3.0×, stamina decay 15/tick
- Visual: embed turns red, flavor messages change to hellish/demonic themes

## Floor system
| Floor Range | Enemy Name | Boss Name | Enemy Mult | Diff Mod |
|-------------|-----------|-----------|-----------|----------|
| 1–50 | Slime | Troll King | 1× | 1.0 |
| 51–100 | Goblin | Troll King | 3× | 2.0 |
| 101–150 | Skeleton | Lich Lord | 5× | 3.0 |
| ... | ... | ... | ... | ... |
| 551–600 | Celestial Warden | Celestial Tyrant | 25× | 13.0 |

## Regions (30 total, 5 per floor tier)
| Tier | Floors | Difficulty | Regions |
|------|--------|-----------|---------|
| 1 | 1–100 | Easy | Whispering Woods, Verdant Meadows, Sandy Coast, Crystal Cavern, Old Ruins |
| 2 | 101–200 | Moderate | Ashen Caverns, Misty Marshes, Thornwood Forest, Lava Tunnels, Sunken Catacombs |
| 3 | 201–300 | Hard | Frosthold Peaks, Storm Coast, Ancient Library, Shadow Vale, Golden Plains |
| 4 | 301–400 | Very Hard | Ashen Wastes, Abyssal Depths, Sky Citadel, Bone Desert, Clockwork Tower |
| 5 | 401–500 | Deadly | Frostspire Peaks, Crimson Forest, Void Rift, Dragon Graveyard, Obsidian Fortress |
| 6 | 501–600 | Legendary | Celestial Sanctum, Infernal Abyss, Eternal Labyrinth, World Tree, Godslayer Plateau |

Each region has defined enemies, boss, loot table, difficulty, and recommended power. The game picks a random region from the current tier on expedition start (DB query by floor range).

## Consumables on expeditions
- **Resurrection Orb**: revives Dead hero → Normal quality
- **Potion**: full heal + clear cooldown
- **Bandage**: halves remaining cooldown or restores 30 stamina

## GRAPH METADATA
- cluster: Mythoria
- node_type: system
- importance_level: 0.9
- hub_node: false
- tags: #mythoria #expeditions #battle
