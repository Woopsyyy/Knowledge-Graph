# Security Patterns: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: high
- hub_node: false
- tags: [security, permissions, coding-patterns, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.85
- hub_score: 0.4
- reuse_score: 0.8
- stability_score: 0.95
- dependency_weight: 0.6

## GRAPH VISUALIZATION
```mermaid
graph TD
    SSec[Ai - skills/Himawari/security-patterns.md] --> Sec[[[04-security.md]]]
```

This document outlines reusable security patterns, permission validation routines, and row-level security parameters for Himawari.

## Credentials Protection

### Git Ignore Patterns
Always add credentials and sensitive configurations to `.gitignore`:
```gitignore
# Discord secrets
config.json

# Minecraft server secrets
config/survivalmod/supabase.json
```

## Discord Command Permission Verification

### REST Slash Command Restriction Template
```javascript
const { SlashCommandBuilder, PermissionFlagsBits } = require('discord.js');

module.exports = {
  data: new SlashCommandBuilder()
    .setName('ticket')
    .setDescription('Configure ticket settings')
    .addSubcommand(sub =>
      sub.setName('setup')
         .setDescription('Create a support ticket panel')
    )
    .setDefaultMemberPermissions(PermissionFlagsBits.Administrator), // Gate command to admins only
  async execute(interaction) {
     // Core logic
  }
};
```

## Discord Hierarchy Verification
Before assigning roles, verify that the bot's highest role is positioned above the role it intends to assign:
```javascript
function canManageRole(guild, roleId) {
  const botMember = guild.members.me;
  const targetRole = guild.roles.cache.get(roleId);
  if (!targetRole) return false;
  return botMember.roles.highest.position > targetRole.position;
}
```

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Restrict slash command definitions to only bind in client interfaces via Discord's native Command Permissions UI configurations, fully removing hardcoded role validation wrappers.
