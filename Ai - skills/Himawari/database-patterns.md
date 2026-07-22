# Database Patterns: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: high
- hub_node: false
- tags: [database, sql, coding-patterns, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.9
- hub_score: 0.4
- reuse_score: 0.8
- stability_score: 0.95
- dependency_weight: 0.7

## GRAPH VISUALIZATION
```mermaid
graph TD
    SDB[Ai - skills/Himawari/database-patterns.md] --> DB[[[05-database.md]]]
```

This document outlines SQL database patterns, RLS policies, indexing strategies, and migrations for the Himawari shared database.

## Migration Patterns

### Migration: `himawari_link_and_tickets`
```sql
-- Enable RLS
ALTER TABLE link_codes ENABLE ROW LEVEL SECURITY;
ALTER TABLE linked_accounts ENABLE ROW LEVEL SECURITY;
ALTER TABLE link_config ENABLE ROW LEVEL SECURITY;
ALTER TABLE ticket_config ENABLE ROW LEVEL SECURITY;
ALTER TABLE tickets ENABLE ROW LEVEL SECURITY;

-- Enable Public read on link codes and linked accounts
CREATE POLICY "Public Read Access for Link Codes" 
ON link_codes FOR SELECT 
TO anon, authenticated 
USING (true);

CREATE POLICY "Public Read Access for Linked Accounts" 
ON linked_accounts FOR SELECT 
TO anon, authenticated 
USING (true);

-- Restrict everything else to Service Role writes
CREATE POLICY "Service Role Full Access for Link Codes" 
ON link_codes FOR ALL 
TO service_role 
USING (true);

CREATE POLICY "Service Role Full Access for Linked Accounts" 
ON linked_accounts FOR ALL 
TO service_role 
USING (true);
```

## Indexing Strategies
* **Account Lookup:** Index the `minecraft_uuid` column in `linked_accounts` as it is searched heavily by the Minecraft mod on player join.
  ```sql
  CREATE INDEX idx_linked_accounts_uuid ON linked_accounts(minecraft_uuid);
  ```
* **Sync Polling:** Index the `nick_synced` column in `linked_accounts` to optimize periodic polling.
  ```sql
  CREATE INDEX idx_linked_accounts_unsynced ON linked_accounts(nick_synced) WHERE (nick_synced = false);
  ```
* **Expired Token Cleanup:** Index the `expires_at` column in `link_codes` to speed up pruning routines.
  ```sql
  CREATE INDEX idx_link_codes_expiry ON link_codes(expires_at);
  ```

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Move ticket data archives to secondary databases or storage buckets once ticket counts exceed 100,000 logs to optimize indexing footprints.
