# Architecture Patterns: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: high
- hub_node: true
- tags: [architecture, patterns, sync, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.95
- hub_score: 0.8
- reuse_score: 0.9
- stability_score: 0.9
- dependency_weight: 0.7

## GRAPH VISUALIZATION
```mermaid
graph TD
    SArch[Ai - skills/Himawari/architecture-patterns.md] --> Arch[[[02-architecture.md]]]
```

This document outlines the dual-system architecture synchronization patterns used to bridge Node.js applications (Discord Bot) and Java Fabric Mods (Minecraft) via a shared Supabase database.

## Shared Database Bridge Pattern

To decouple services operating in completely separate execution runtimes (V8 Engine vs. JVM Host), state changes are synced through PostgreSQL rows rather than peer-to-peer network bindings.

```mermaid
graph LR
    JVM[Java Fabric Mod] -->|1. Write Code / Request Link| PostgreSQL[(PostgreSQL Database)]
    PostgreSQL -.->|2. Persist State Changes| PostgreSQL
    PostgreSQL <--|3. Read Unsynced / Apply Nicknames| Node[Node.js Discord Bot]
```

### Key Principles

1. **Write Once, Poll/Sync Second:**
   - Platform A writes the intent (e.g. `/link` code).
   - Platform B validates and updates the database, clearing the intent.
   - Platform A or a polling task monitors DB updates to apply local effects (such as Nickname formatting or role distribution).

2. **Database Schema Coordination:**
   - Coordinate modifications across both repositories simultaneously.
   - Store SQL migrations inside a shared folder structure or document changes in `docs/` before implementing changes in code.

3. **Rate Limits & Failovers:**
   - Rate limit database insertions to avoid PostgreSQL exhaustion.
   - Fall back on local configurations if the Supabase project goes offline.

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Move to gRPC-based event streams where the Fabric Mod and Discord Bot connect to a unified streaming bridge, enabling sub-millisecond event responses.
