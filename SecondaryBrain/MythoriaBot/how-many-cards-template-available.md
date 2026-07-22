---
asked: 2026-07-22
source: codebase
tags: [cards, templates, hero_templates, database]
project: MythoriaBot
---

## Question

How many cards template available?

## Answer

There are **36** card templates currently in the `hero_templates` table.

Templates are stored in the `hero_templates` database table (defined in `database/schema.sql:25`), mapped to the `HeroTemplate` TypeScript interface (`src/types/game.ts:37`). There are no pre-seeded templates — all 36 were created dynamically via the dashboard import API (`POST /api/import-card` in `src/dashboard/server.ts`).

Each template has a 7-char alphanumeric ID, name, anime, kingdom, 10 base stats, artwork path, and description. Rarity distribution follows round-robin across 6 rarities (common → celestial), max 10 per rarity, 60 max per character.

## Source Files

- `database/schema.sql`
- `src/types/game.ts`
- `src/dashboard/server.ts`
- `src/services/CardService.ts`
- `src/repositories/HeroRepository.ts`
