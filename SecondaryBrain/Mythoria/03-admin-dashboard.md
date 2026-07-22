# Mythoria — Admin Dashboard

Three tools for bot owner card management:

## 1. `/mimportcard` (slash command)
Owner-only (`export const owner = true`). Creates a card template record.

**Options:** `name` (string), `rarity` (6 rarities)

**Logic:**
- Slugifies name + random 3 chars → template ID (e.g. `shadow_mage_x7f`)
- Randomly assigns kingdom from the 5 available
- Inserts into `hero_templates` with default base stats (HP:100, ATK:10, DEF:5, MAG:10, MDEF:5, crit:5%/1.5x)
- Responds ephemerally with template ID, kingdom, name, rarity

## 2. Express Dashboard (localhost:3001)
Inline Express server started alongside the bot via `startDashboard()` in `server.ts`.

**`POST /api/import-card`** (multipart/form-data):
- Fields: `name`, `rarity`, `artwork` (optional file)
 - Compresses artwork: resizes to 400×480, converts to WebP (80% quality) via Sharp
 - Saves to Nginx-served directory `assets/artwork/{rarity}/{id}.webp`
 - Rarity folder allows the same character to have different artwork at each rarity tier (same card, multiple rarity variants)
 - Creates asset folders on first use
 - Inserts hero_template with all base stats + artwork URL pointing to local Nginx
- Returns JSON `{ success, id, kingdom }`

**Template ID format:** `{slug}_{random3}` (e.g., `gojo_saturo_5dr`). Slug is derived from the card name (lowercased, stripped of special chars). Random suffix prevents ID collisions.

**Serves** static files from `public/` on all other routes.

## 3. Supabase Edge Function
`supabase/functions/import-card/index.ts` (Deno runtime):

**`POST /functions/v1/import-card`:**
- Validates Bearer token via `supabaseAdmin.auth.getUser(token)`
- Inserts row into `hero_templates` using service role key (bypasses RLS)
- Used by the web portal's `CardImporter` component

## Owner identification
- `OWNER_ID` from `.env` — compared at runtime in `bot/index.ts` `interactionHandler`
- Commands with `owner: true` in their export are gated
- Web portal reads `VITE_OWNER_DISCORD_ID` env var for `CardImporter` visibility

## Cleanup
`/cleanup <user_id>` — Owner-only. Deletes all records for a Discord user across: heroes, inventory, equipment, teams + members, expeditions, trades, auctions, and the user row itself. Reports deleted counts per table.

## GRAPH METADATA
- cluster: Mythoria
- node_type: feature
- importance_level: 0.85
- hub_node: false
- tags: #mythoria #admin
