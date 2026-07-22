# Mythoria — Web Portal

## Purpose
Public-facing React SPA with landing page, docs, patch notes, and authenticated dashboard.

## Tech stack
- **Framework:** Vite 8 + React 18
- **Language:** TypeScript
- **Styling:** Tailwind CSS v4 (custom `myth-*` colour palette)
- **Routing:** React Router v7
- **Auth:** Supabase Discord OAuth
- **Edge Functions:** Deno (card import endpoint)
- **Hosting:** Vercel

## Routes
| Route | Component | Auth | Description |
|-------|-----------|------|-------------|
| `/` | Landing | Public | Hero section, features grid, commands list, invite/login CTAs |
| `/dashboard` | Dashboard | Required | 6-section user dashboard with sidebar |
| `/docs` | Docs | Public | Command reference, kingdom lore, rarity system |
| `/changes` | Changes | Public | Patch notes (v1.0.0) |

## Components (9 total)
| Component | Location | Purpose |
|-----------|----------|---------|
| Navbar | `Navbar.tsx` | Fixed top bar with auth state, avatar, login/logout, nav links |
| Sidebar | `Sidebar.tsx` | Dashboard navigation (6 sections), mobile overlay, owner badge |
| HeroSection | `HeroSection.tsx` | Landing hero with animated gradient, floating SVG icons, CTAs |
| FeaturesGrid | `FeaturesGrid.tsx` | 8-feature card grid with hover effects and stagger animation |
| CommandsList | `CommandsList.tsx` | 14-command reference list |
| CardRarities | `CardRarities.tsx` | Fetches live rarities from Supabase `card_rarities` table |
| CardImporter | `CardImporter.tsx` | Admin form: name/kingdom/class/rarity/stats grid → Edge Function POST |
| ServiceStatus | `ServiceStatus.tsx` | 4-service indicator (Discord Bot, PostgreSQL, Redis, BullMQ) |
| Footer | `Footer.tsx` | Copyright footer |

## Dashboard sections (6)
| Section | ID | Description |
|---------|-----|-------------|
| Home | `home` | 4 stat cards (coins, essence, hero count, account age) + ServiceStatus |
| Inventory | `inventory` | Item list with quantities from `inventory` table |
| Cards | `cards` | Hero collection ordered by level, star rating rendering |
| Exploration | `exploration` | Expedition history with status badges |
| Analytics | `analytics` | 6 aggregate stats (total users, heroes, inventory items, expeditions) |
| Import Card | `import-card` | CardImporter component (owner-only visibility) |

## Auth flow
1. User clicks "Login with Discord" → `supabase.auth.signInWithOAuth({ provider: 'discord' })`
2. Discord OAuth redirect back → `/auth/callback` → redirect to `/dashboard`
3. `AuthContext` wraps entire app, subscribes to `onAuthStateChange`
4. Owner detection via `user.user_metadata.provider_id` compared to `VITE_OWNER_DISCORD_ID`
5. `ProtectedRoute` redirects unauthenticated users to `/`

## API access
- `supabase.ts`: anon key client (`@supabase/supabase-js`) — RLS-gated, users see only own data
- `EDGE_FUNCTION_URL`: derived from `VITE_SUPABASE_URL` + `/functions/v1`
- CardImporter POSTs to Edge Function with Bearer auth token

## Environment variables
| Variable | Purpose |
|----------|---------|
| `VITE_SUPABASE_URL` | Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | Anonymous API key |
| `VITE_OWNER_DISCORD_ID` | Bot owner Discord ID (admin detection) |
| `VITE_DISCORD_BOT_INVITE_URL` | OAuth invite link |

## GRAPH METADATA
- cluster: Mythoria
- node_type: subsystem
- importance_level: 0.8
- hub_node: false
- tags: #mythoria #web
