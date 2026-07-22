# Security Patterns - Baritone 2.0

## 1. Input Validation and Sandbox Safety
Restrict local automation agents from executing system commands or modifying filesystem paths outside of defined data folders.

### Implementation Checklist
* **Sanitize Inputs:** Check coordinates, block identities, and filenames against allowlists before expanding path goals.
* **Directory Bounds Enforcement:** Restrict all file IO actions using `Path.normalize()` checks to prevent path traversal vulnerability attacks.

---

## 2. Server Integration and Anti-Cheat Mitigation
When simulating interactions, match normal client behavior signatures to avoid triggering server-side heuristic detections.

### Best Practices
* **Look Interpolation:** Smooth camera yaw and pitch transitions over multiple ticks instead of snapping angles instantly.
* **Input Tick Clamping:** Limit action packet rates to normal client frequencies (e.g. 20 Hz).
