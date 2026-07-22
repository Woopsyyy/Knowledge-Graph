# Automation Skills: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: high
- hub_node: false
- tags: [automation, sync, scheduling, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.9
- hub_score: 0.4
- reuse_score: 0.8
- stability_score: 0.95
- dependency_weight: 0.5

## GRAPH VISUALIZATION
```mermaid
graph TD
    SAuto[Ai - skills/Himawari/automation-skills.md] --> MUser[[[modules/user.md]]]
```

This document outlines reusable automation scripts, polling loops, and database cleanup routines for Himawari.

## Periodic Nickname Sync Automation

```javascript
const { ActivityType } = require('discord.js');

/**
 * Periodically polls the database for un-synchronized account links and applies changes.
 * @param {Client} client - The Discord.js client.
 * @param {SupabaseService} db - The Supabase service client.
 * @param {string} guildId - The target server Guild ID.
 */
function startNicknameSyncLoop(client, db, guildId) {
  setInterval(async () => {
    try {
      const { data: unsynced, error } = await db.client
        .from('linked_accounts')
        .select('*')
        .eq('nick_synced', false);

      if (error) throw error;
      if (!unsynced || unsynced.length === 0) return;

      const guild = client.guilds.cache.get(guildId);
      if (!guild) return;

      for (const record of unsynced) {
        try {
          const member = await guild.members.fetch(record.discord_id);
          if (member) {
            await member.setNickname(`${record.minecraft_name} ⛏`);
            
            const config = await db.fetchGuildConfig(guildId);
            if (config?.linked_role_id) {
              await member.roles.add(config.linked_role_id);
            }
          }
          
          await db.client
            .from('linked_accounts')
            .update({ nick_synced: true })
            .eq('discord_id', record.discord_id);
            
        } catch (memberError) {
          console.error(`Failed to sync user ${record.discord_id}:`, memberError);
        }
      }
    } catch (err) {
      console.error('Error in nickname sync loop:', err);
    }
  }, 15000);
}
```

## Expired Token Cleanup Automation
Automate deletion of expired codes directly inside database queries:
```sql
DELETE FROM link_codes WHERE expires_at < NOW();
```

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Transition the periodic sync to serverless edge functions receiving direct PostgreSQL change notifications via logical replication, achieving near real-time updates.
