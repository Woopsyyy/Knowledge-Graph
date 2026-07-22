# Architecture Patterns - Baritone 2.0

## 1. Asynchronous Off-Thread Graph Search Heuristics
When computing paths or spatial designs in a real-time system, run operations asynchronously on background threads to prevent halting the main execution loop.

### Core Execution Pattern
* **Dispatch Thread Task:** Spawn graph calculations to a dedicated ThreadPoolExecutor.
* **Non-Blocking Query Interfaces:** Query static snapshots of data instead of live models to avoid concurrency errors.
* **Interruptible Calculations:** The path planning thread checks for cancellation flags (`isCancelled()`) to release resources quickly when goals change.

---

## 2. Process Manager State Delegation
Separate specific goals into state machines that subclass a common process interface. Let a central coordinator prioritize active processes.

### State Orchestrator Pattern
```java
public interface IProcess {
    /**
     * Called every tick. Returns true if this process wants to take control.
     */
    boolean isActive();
    
    /**
     * Executes process steps.
     */
    void onTick();
}
```
* **Process Priority Resolution:** The controller sorts active processes by priority, delegating control to the highest priority process.
