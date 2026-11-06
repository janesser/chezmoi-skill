---
name: chezmoi-status
description: "Read-only check of your chezmoi dotfile setup: what's managed, what's pending, and whether any changes are out of sync. Use when you want a status view before acting — it never modifies anything. Triggers on 'status', 'check', 'diff', 'sync', 'what changed', or 'chezmoi status'."
license: UNKNOWN
---

# `/skill:chezmoi-status` — Check Current State

Read-only view of your dotfile setup: what's managed, what's out of sync, what's pending.

## Core principle

This skill never modifies anything. If you want to act on findings, they become their own `/skill:chezmoi-add`, `/skill:chezmoi-update`, or `/skill:chezmoi-review` task. All commands presume the source dir `~/.local/share/chezmoi` unless a target path is given.

## Pre-check (read-only)

- `chezmoi status` — full inventory of managed and unmanaged files.
- `chezmoi diff` — what would change if applied.
- Are there pending externals? (`chezmoi diff --refresh-externals`)

## Output

1. Inventory summary: how many files/dirs are managed, how many are templates, how many are scripts.
2. Sync status: what's pending vs. committed in the source dir.
3. Any errors or warnings (template rendering failures, permission issues, missing externals).

## Invocation

```
/skill:chezmoi-status
/skill:chezmoi-status ~/.config/nvim
/skill:chezmoi-status --refresh-externals
```

Arguments after `/skill:name` are appended to the loaded instructions as a user request. If no target is given, `status`/`diff` run for the whole source dir.

## Gotchas

1. **Interactive prompts block execution.** Always use `--refresh-externals --force` flags so chezmoi never hangs waiting for TTY input.
2. **Read-only by design.** Anything that modifies, adds, or removes files waits for confirmation in `/skill:chezmoi-update` or `/skill:chezmoi-review`.

## References

- See `.agents/skills/chezmoi-init` for the setup flow.
- See `references/pi-skills.md` for Pi Agent skill mechanics.
