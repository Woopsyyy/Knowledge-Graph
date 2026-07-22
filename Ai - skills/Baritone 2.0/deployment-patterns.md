# Deployment Patterns - Baritone 2.0

## 1. Multi-Target Modular Packaging
For projects targets distributed across different runtime loaders, separate core business logic from boot modules.

### Pattern Rules
* **Core Library Module:** Keep all graph, API, and event logic in a central module with neutral interfaces.
* **Loader-Specific Modules:** Build loader wrappers (e.g. Forge, Fabric) that depend on the core library, inject specific launch hooks, and package final platform jars.

---

## 2. ProGuard Obfuscation & Mappings
Keep API paths visible to third-party developers while obfuscating implementation directories to optimize size. Export `.map` files with releases to assist in stack trace parsing.
