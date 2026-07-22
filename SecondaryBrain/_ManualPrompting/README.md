# _ManualPrompting

Portable prompts for moving between AI agents without losing the SecondaryBrain context.

## When switching to a new / different AI agent

1. Paste **`00-system-prompt.md`** first. This installs the full "Neural Predictive Architect"
   behavior and tells the agent where the vault, global memory, and skills store live.
2. Paste a filled-in copy of **`01-project-handoff-template.md`** for the specific project you
   want to continue. This gives the agent the current state and the next task.
3. Tell it to proceed with the next task.

## Files

| File | Purpose |
|------|---------|
| `00-system-prompt.md` | Full system prompt — reproduces the whole behavior in any agent. |
| `01-project-handoff-template.md` | Per-project context handoff. Copy, fill in, paste. |

## Notes

- For **Claude Code**, the everyday lightweight version of this behavior is already wired into
  `~/.claude/CLAUDE.md` (the `SecondaryBrain protocol` section), so you don't need to paste
  anything — it reads/updates the vault automatically. Use this folder mainly for *other*
  agents (ChatGPT, Gemini, Cursor, etc.).
- The full heavy ruleset (scoring, pruning, self-healing, prediction) is mirrored in
  `..\_System\reference-architecture-protocol.md`.
