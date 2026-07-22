# API Patterns: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: high
- hub_node: false
- tags: [api, database-client, coding-patterns, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.9
- hub_score: 0.5
- reuse_score: 0.8
- stability_score: 0.9
- dependency_weight: 0.6

## GRAPH VISUALIZATION
```mermaid
graph TD
    SAPI[Ai - skills/Himawari/api-patterns.md] --> MApi[[[modules/api.md]]]
```

This document outlines reusable API patterns for interacting with the Supabase PostgREST endpoints in both JavaScript and Java.

## JavaScript API Patterns (discord.js Bot)

### Supabase Initialization Pattern
```javascript
const { createClient } = require('@supabase/supabase-js');

class SupabaseService {
  constructor(url, serviceKey) {
    if (!url || !serviceKey) {
      throw new Error('Supabase URL and Service Key are required.');
    }
    this.client = createClient(url, serviceKey, {
      auth: {
        persistSession: false
      }
    });
  }

  async fetchGuildConfig(guildId) {
    const { data, error } = await this.client
      .from('link_config')
      .select('linked_role_id')
      .eq('guild_id', guildId)
      .maybeSingle();

    if (error) throw error;
    return data;
  }
}
```

## Java API Patterns (Fabric Mod)

### Asynchronous PostgREST Client Pattern
```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.concurrent.CompletableFuture;

public class SupabaseClient {
    private final HttpClient httpClient = HttpClient.newBuilder().build();
    private final String url;
    private final String serviceKey;

    public SupabaseClient(String url, String serviceKey) {
        this.url = url;
        this.serviceKey = serviceKey;
    }

    public CompletableFuture<String> getLinkedAccount(String uuid) {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url + "/rest/v1/linked_accounts?minecraft_uuid=eq." + uuid))
            .header("apikey", serviceKey)
            .header("Authorization", "Bearer " + serviceKey)
            .header("Accept", "application/json")
            .GET()
            .build();

        return httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString())
            .thenApply(HttpResponse::body);
    }
}
```

## Best Practices
* **Header Authorization:** For raw HTTP queries, always inject both `apikey` and `Authorization: Bearer <Key>` headers to comply with Supabase routing requirements.
* **Keep Connection Alive:** Reuse a single HTTP Client connection pool to avoid thread overhead during high concurrency.

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Transition the Java mod queries from raw `HttpClient` construction to Retrofit or OkHttp interfaces, simplifying query parameters mapping.
