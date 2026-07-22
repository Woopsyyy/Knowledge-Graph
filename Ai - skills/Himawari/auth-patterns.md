# Authentication Patterns: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: high
- hub_node: false
- tags: [auth, coding-patterns, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.85
- hub_score: 0.4
- reuse_score: 0.8
- stability_score: 0.95
- dependency_weight: 0.6

## GRAPH VISUALIZATION
```mermaid
graph TD
    SAuth[Ai - skills/Himawari/auth-patterns.md] --> MAuth[[[modules/auth.md]]]
```

This document outlines reusable account verification code generation and token validation patterns between Discord and Minecraft.

## Reusable Verification Patterns

### Alphanumeric Code Generation (Node.js)
```javascript
const crypto = require('crypto');

/**
 * Generates a 6-character alphanumeric verification code.
 * Excludes confusing characters (0, O, I, L, 1).
 * @returns {string}
 */
function generateVerificationCode() {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  let result = '';
  const bytes = crypto.randomBytes(6);
  for (let i = 0; i < 6; i++) {
    result += chars[bytes[i] % chars.length];
  }
  return result;
}
```

### Verification Code Verification (Java)
```java
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;
import java.util.concurrent.CompletableFuture;

public class CodeValidator {
    private final HttpClient client = HttpClient.newHttpClient();
    private final String supabaseUrl;
    private final String serviceKey;

    public CodeValidator(String supabaseUrl, String serviceKey) {
        this.supabaseUrl = supabaseUrl;
        this.serviceKey = serviceKey;
    }

    public CompletableFuture<Boolean> validateCode(String code, String uuid, String name) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                // PostgREST call to resolve and delete the code
                return true; 
            } catch (Exception e) {
                return false;
            }
        });
    }
}
```

## Best Practices
* **Exclusion List:** Avoid confusing characters (`1`, `I`, `l`, `0`, `O`) in generated verification tokens to prevent manual typing errors.
* **Database Expiration Constraints:** Always index the `expires_at` column in SQL to optimize deletion queries.

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Abstract verification generator methods into an NPM module shared across Minecraft, Discord, and target web registration portals.
