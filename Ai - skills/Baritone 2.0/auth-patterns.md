# Auth Patterns - Baritone 2.0

## Context-Delegated Authentication
When operating as a client-side agent inside a host app wrapper, delegate authentication to the host instead of storing user credentials.

### Java Implementation Pattern
```java
public interface ISessionProvider {
    /**
     * Retrieves the authentication identity from the host environment context.
     */
    String getSessionUsername();
    
    /**
     * Retrieves the host verified unique identifier.
     */
    java.util.UUID getSessionUUID();
}
```

### Best Practices
* **No Cache Policy:** Never cache access tokens or credentials locally. Read values directly from volatile memory fields of host classes on demand.
* **Trace Isolation:** Strip authentication contexts and UUID logs before writing to files.
