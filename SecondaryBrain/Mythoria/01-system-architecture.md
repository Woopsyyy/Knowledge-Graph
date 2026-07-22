# Mythoria — System Architecture

## Project structure
```
Mythoria/
├── .env                          # Shared env vars (DATABASE_URL, Discord IDs, Redis)
├── database/
│   └── schema.sql                # 12 tables + 7 indexes + 1 fn + trigger
│   └── seed.sql                  # 13 items + 3 regions seed data
├── MythoriaBot/                  # Discord bot
│   ├── src/
│   │   ├── bot/                  # Discord.js handlers
│   │   │   ├── commands/         # 19 slash commands
│   │   │   ├── buttons/          # 10 button handlers (inspect pages, stats pages, trade)
│   │   │   ├── events/           # ready, messageCreate, GuildJoin, GuildLeave
│   │   │   ├── utils/            # ComponentLoader, EventLoader, RegisterCommands, statsView, ReadFolder, CheckIntents
│   │   │   └── index.ts          # Entry point — initialises client, loads everything
│   │   ├── dashboard/            # Express server (localhost:3001)
│   │   │   └── server.ts         # POST /api/import-card (multipart, Sharp compress, saves to Nginx-served assets)
│   │   ├── database/             # Client configs
│   │   │   ├── postgres.ts       # pg.Pool wrapper with query() helper
│   │   │   └── redis.ts          # ioredis client
│   │   ├── repositories/         # 7 DB query classes
│   │   │   ├── UserRepository.ts
│   │   │   ├── HeroRepository.ts
│   │   │   ├── InventoryRepository.ts
│   │   │   ├── TeamRepository.ts
│   │   │   ├── TradeRepository.ts
│   │   │   ├── AuctionRepository.ts
│   │   │   └── ExpeditionRepository.ts
│   │   ├── services/             # 12 business-logic classes
│   │   │   ├── CardService.ts    # Stat engine, mint, disassemble
│   │   │   ├── ImageService.ts   # SVG→PNG card overlay on bucket artwork (Sharp)
│   │   │   ├── UserService.ts    # CRUD, daily/weekly/monthly claims
│   │   │   ├── UpgradeService.ts # Quality ladder + celestial orbs
│   │   │   ├── MergeService.ts   # 2→1 star fusion
│   │   │   ├── BattleService.ts  # Turn-based combat engine
│   │   │   ├── ExpeditionService.ts  # Tick-based auto-dungeon crawler
│   │   │   ├── TradeService.ts   # Peer-to-peer card trades
│   │   │   ├── AuctionService.ts # Full auction house + expiry
│   │   │   ├── CraftingService.ts    # 18 recipes (6 weapons + 12 gear) + batch crafting
│   │   │   ├── InventoryService.ts   # Items + equipment with Redis cache
│   │   │   ├── TeamService.ts    # Party CRUD (5 teams × 5 slots)
│   │   │   ├── MiningService.ts  # Tool-based background resource gathering
│   │   │   ├── FishingService.ts # Tool-based background resource gathering
│   │   │   ├── DropService.ts    # Card drop system
│   │   │   ├── InviteService.ts  # Invite tracking + rewards
│   │   │   ├── NotificationService.ts  # DM notification toggles
│   │   │   └── UserSettingsService.ts   # User preferences CRUD
│   │   ├── types/game.ts         # All TypeScript interfaces + constants
│   │   ├── utils/                # logger, sendDM, lockCheck
│   │   └── workers/              # 6 BullMQ workers
│   │       ├── UpgradeWorker.ts
│   │       ├── ExpeditionTickWorker.ts
│   │       ├── SchedulerWorker.ts
│   │       ├── ImageWorker.ts
│   │       ├── MiningWorker.ts
│   │       └── FishingWorker.ts
│   ├── public/                   # Express dashboard static files
│   ├── package.json
│   └── tsconfig.json
├── MythoriaWeb/                  # Web portal
│   ├── src/
│   │   ├── components/           # 9 React components
│   │   ├── context/              # AuthContext (Discord OAuth)
│   │   ├── data/                 # Static feature + command data
│   │   ├── lib/                  # Supabase client (anon key) + Edge Function URL
│   │   ├── pages/                # 4 route pages
│   │   ├── App.tsx               # React Router v7 layout
│   │   └── main.tsx              # Entry point
│   ├── supabase/functions/import-card/  # Edge Function (Deno)
│   ├── package.json
│   ├── vite.config.ts
│   └── vercel.json
```

## Tech stack
| Component | Technology |
|-----------|-----------|
| Database  | PostgreSQL 16 (local Docker, direct `pg` connection) |
| Cache     | Redis 7 (local Docker, cooldowns, expedition state, inventory cache, BullMQ) |
| Bot       | Node.js 22 Alpine, TypeScript, discord.js v14, BullMQ v5, Sharp |
| Dashboard | Express v5, Multer, Sharp (in-process) |
| Web       | Vite 8, React 18, TypeScript, Tailwind CSS v4, React Router v7 |
| Auth      | Supabase Discord OAuth (web), Discord bot token + OWNER_ID (bot) |
| Assets    | Nginx (local Docker, serves artwork + environments on port 3002) |
| Edge Functions | Supabase Deno runtime (import-card) |
| Hosting   | Docker Compose (bot + DB + cache + assets), Vercel (web) |

## Data flow
- **Bot** connects to PostgreSQL 16 directly via `pg.Pool` (DATABASE_URL), no cloud DB
- **Web** uses `SUPABASE_ANON_KEY` (RLS-gated — users see only their own heroes/inventory)
- **Express Dashboard** uses `pg.Pool` (same as bot) for admin template insertion, uploads artwork to local Nginx
- **Card images** generated on-demand via Sharp (Nginx-served artwork + SVG overlay → PNG), attached to Discord reply as `attachment://card.png`
- **Assets** served by local Nginx container on port 3002 (replaces Supabase Storage buckets)
- **Edge Function** uses `SUPABASE_SERVICE_ROLE_KEY` (admin bypass of RLS)

## Database schema (12 tables)
| Table | PK | Purpose |
|-------|-----|---------|
| `users` | discord_id | Coins, essence, claim timers, starter flag |
| `hero_templates` | id | Card type definitions (name, class, base stats, rarity) |
| `heroes` | id | Owned hero instances (quality, tier, stars, stats, equipment, traits) |
| `items` | id | Item type definitions |
| `inventory` | (owner, item) | Per-user item quantities |
| `equipment` | id | Equipment instances with stats JSON |
| `teams` | id | Expedition parties (5 per user limit) |
| `team_members` | (team, position) | Hero-to-team mapping (5 slots) |
| `regions` | id | Expedition zone definitions (enemies, boss, loot tables) |
| `expeditions` | team_id | Active/completed expeditions (red gate, floor, loot) |
| `trades` | id | P2P trade proposals |
| `auctions` | id | Auction listings with bids |

## Key design decisions
- Card images rendered in-memory via Sharp (overlay on bucket artwork) — no persistent card image storage
- Admin artwork stored in Nginx-served directory at `artwork/{rarity}/{class}/{id}.webp` (WebP compressed)
- Only one `.env` at project root for all secrets
- BullMQ for async job queuing (upgrades, expedition ticks, image gen, scheduler)
- Express dashboard runs alongside bot for local card importing
- Fully local Docker stack — no cloud dependencies (no Supabase, no Upstash, no PostgREST)

## Evolution timeline
- **2026-07-15** — Initial structure. Bot moved to `MythoriaBot/`, web created in `MythoriaWeb/`.
- **2026-07-15** — Star rating system (1-5★), admin card creation, web OAuth login.
- **2026-07-16** — Full architecture documentation: 12 services, 7 repos, 4 workers, 19 commands mapped.
- **2026-07-16** — `/inspect` rewritten with 3-button page navigation (Basic/Stats/Gear). Card image generation reworked: overlays name/quality/stars/tier/level on bucket artwork via Sharp. Crafting expanded to 18 recipes (6 weapons + 12 gear). ImageWorker simplified to just copy `template.artworkPath`. Floor 200+ gear drops added to expedition loot. Tower image added to explore embeds.
- **2026-07-16** — **Supabase → Local Docker.** Replaced Supabase PostgreSQL + Upstash Redis with local PostgreSQL 16 + Redis 7. Added Nginx for asset serving. Removed PostgREST. Bot now queries DB directly via `pg.Pool` (`postgres.ts`). 6→8 workers (added MiningWorker, FishingWorker). 12→18 services (added Mining, Fishing, Drop, Invite, Notification, UserSettings).

## GRAPH METADATA
- cluster: Mythoria
- node_type: architecture
- importance_level: 0.95
- hub_node: true
- tags: #mythoria #architecture
