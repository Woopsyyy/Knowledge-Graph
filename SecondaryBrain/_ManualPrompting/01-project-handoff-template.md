# Project Handoff Template

> Fill this in and paste it into a new AI agent (after `00-system-prompt.md`) to hand off a
> single project with full context. Replace every `<...>` placeholder.

---

## Project: `<project-name>`

**Cluster:** `<cluster-id, e.g. himawari>`
**Status:** `<active | paused | deprecated>`
**Last updated:** `<YYYY-MM-DD>`

### One-line summary
`<what this project is, in one sentence>`

### Current architecture (hub nodes)
- `<SecondaryBrain\<Project>\00-overview.md>` — `<role>`
- `<SecondaryBrain\<Project>\02-architecture.md>` — `<role>`
- `<add the highest-importance nodes>`

### Tech stack
`<languages, frameworks, services, hosting>`

### What works right now
- `<feature / module that is done and validated>`

### Current task / next steps
1. `<the immediate next thing to do>`
2. `<follow-up>`

### Open questions / decisions pending
- `<anything unresolved>`

### Reusable patterns already extracted
- `<Ai - skills\<Project>\<pattern>.md>` — `<what it does>`

### Constraints / gotchas
- `<deployment quirks, env config, things that broke before>`

### Where to read first
1. `GLOBAL-PROJECT-MEMORY.md` → find the `<cluster>` cluster
2. `SecondaryBrain\<Project>\00-overview.md`
3. `<any node directly related to the current task>`

---

> After completing work, update the project's nodes and `GLOBAL-PROJECT-MEMORY.md`, then
> regenerate this handoff so the next agent starts current.
