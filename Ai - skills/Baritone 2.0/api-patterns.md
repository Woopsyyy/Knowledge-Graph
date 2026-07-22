# API Patterns - Baritone 2.0

## 1. Provider Decoupling Pattern
Decouple interfaces from compiled implementations. Load implementation classes at runtime using reflection to support third-party integrations.

### Java Implementation Pattern
```java
package myapp.api;

public final class AppAPI {
    private static final IAppProvider provider;

    static {
        try {
            provider = (IAppProvider) Class.forName("myapp.impl.AppProvider").getDeclaredConstructor().newInstance();
        } catch (Exception ex) {
            throw new RuntimeException("Failed reflectively instantiating AppProvider implementation class", ex);
        }
    }

    public static IAppProvider getProvider() {
        return provider;
    }
}
```

---

## 2. Event-Driven Module Extension Pattern
Relay client triggers to registered modular subsystems using a centralized event bus handler.

### Bus Dispatch Template
```java
package myapp.api.event;

import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class EventBus {
    private final List<Object> listeners = new CopyOnWriteArrayList<>();

    public void register(Object listener) {
        listeners.add(listener);
    }

    public void post(Event event) {
        for (Object listener : listeners) {
            if (listener instanceof EventListener) {
                try {
                    ((EventListener) listener).onEvent(event);
                } catch (Exception ex) {
                    ex.printStackTrace(); // Prevent module crash from taking down core dispatch loop
                }
            }
        }
    }
}
```
