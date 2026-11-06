---
name: chezmoi-update
description: Update dotfiles, including external resources such as git repos, archives and secrets. Use when you want to pull latest externals and sync your machine state with the source dir. Triggers on 'update', 'sync dotfiles', 'refresh externals', 'chezmoi update'.
license: UNKNOWN
---

# `/skill:chezmoi-update` — Update Dotfiles

Updates your dotfiles, including external resources (git repos, archives, secrets).

## Core principle

Preview every change before it runs. Nothing is applied without `yes`. All commands operate on `~/.local/share/chezmoi` unless a target path is given.

## Pre-check (read-only)

- `chezmoi status` — what's out of sync?
- `chezmoi diff` — what would change?
- Are externals configured? (`.chezmoiexternal.toml`, in the source dir)

## Workflow

1. Preview the changes: `chezmoi diff --refresh-externals --force`.
2. Show the preview to the user and ask for confirmation.
3. On confirmation, run: `chezmoi apply --refresh-externals --force`.
4. Verify with a final `chezmoi status` and show summary of what changed.

## Guard

If `chezmoi diff` shows any file being *removed* unexpectedly (not in your intentional list), stop and show it — the user must confirm deletions explicitly.

## Invocations

```
/skill:chezmoi-update
/skill:chezmoi-update --refresh-externals
```

## Gotchas

1. **Interactive prompts block execution.** Always use `--refresh-externals --force` flags so chezmoi never hangs waiting for TTY input.
2. **Dry-run first.** Run `chezmoi diff`/`chezmoi cat` before any `chezmoi apply`.
3. **Removing managed files.** Use `.chezmoiremove` — never just delete a file from your home dir, chezmoi will re-add it.
