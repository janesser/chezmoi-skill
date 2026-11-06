---
name: chezmoi-apply
description: Apply pending changes to your dotfiles, optionally scoped to a target path. Use when you want to push pending changes — same as update but with the option to scope to one target. Triggers on 'apply', 'push changes', 'deploy dotfiles', 'chezmoi apply'.
license: UNKNOWN
---

# `/skill:chezmoi-apply [target]` — Apply Changes (optionally scoped)

Apply pending changes to your dotfiles. Unlike `update`, this can be scoped to a specific target path, e.g. `/skill:chezmoi-apply ~/.local/bin`.

## Core principles

1. **Preview before apply.** Run `chezmoi diff`/`chezmoi cat` before any `chezmoi apply`.
2. **Explicit confirmation before change.** Nothing is applied without an explicit `yes`.
3. **Source dir.** All commands operate on `~/.local/share/chezmoi` unless a target path is given.

## Pre-check (read-only)

- Is there anything pending? (`chezmoi status`) if no target given
- Is the target already managed? (`chezmoi status --<target>`) if target given

## Workflow

1. If no target: same as `update` — preview then `chezmoi apply --refresh-externals --force`.
2. If target given:
   a. Preview what's pending for that target: `chezmoi diff <target>` or `chezmoi status --<target>`.
   b. If the target isn't managed yet: suggest `chezmoi add <target>` to include it in management, then confirm.
   c. On confirmation, run `chezmoi apply --refresh-externals --force` (same flags).
3. Verify with a final `chezmoi status` and show summary of what changed.

## Guard

If `chezmoi diff` shows any file being *removed* unexpectedly (not in your intentional list), stop and show it — the user must confirm deletions explicitly.

## Invocations

```
/skill:chezmoi-apply
/skill:chezmoi-apply ~/.local/bin
/skill:chezmoi-apply --refresh-externals
```

## Gotchas

1. **Interactive prompts block execution.** Always use `--refresh-externals --force` flags so chezmoi never hangs waiting for TTY input.
2. **Dry-run first.** Run `chezmoi diff`/`chezmoi cat` before any `chezmoi apply`.
3. **Removing managed files.** Use `.chezmoiremove` — never just delete a file from your home dir, chezmoi will re-add it.
