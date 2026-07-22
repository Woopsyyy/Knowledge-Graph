# Automation Skills - Baritone 2.0

## 1. Segmented Pathfinding splice mechanics
Use segmented pathfinding when navigating unknown graph systems to generate paths incrementally.

### Pattern Rules
* **Goal Evaluation:** Define target boundaries (`GoalBlock`, `GoalXZ`).
* **Segment Selection:** When calculations terminate, select the best partial path using weighted cost evaluations.
* **Pre-computation Splicing:** Pre-calculate the next path segment on a background thread while the active segment is executing, splicing them together seamlessly.

---

## 2. Input Simulation
Inject simulated input states directly into the host app loop to mock player actions.

### Override Pattern
```java
public class InputOverrideHandler {
    private boolean forward;
    private boolean sprint;

    public void tick() {
        if (shouldOverride()) {
            game.input.forward = this.forward;
            game.input.sprint = this.sprint;
        }
    }
}
```
* **Dynamic Cost Tuning:** Adjust block action costs dynamically using tool attributes and configurable placement penalties.
