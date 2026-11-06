---
name: chezmoi-add
description: Add a file to chezmoi management, covering everything until the git commit — preview, naming conventions, template options, and push. Use when you want to manage a new dotfile or config file. Triggers on 'add', 'manage this file', 'chezmoi add', 'new file'.
license: UNKNOWN
---

# `/skill:chezmoi-add <file>` — Add a File to Dotfiles

Adds a new file to chezmoi management, covering everything until the git commit.

## Core principles

1. **Discover before acting.** Is the file already managed? What's the target path and permissions?
2. **Explicit confirmation before change.** No write operation happens without a preview and an explicit `yes`.
3. **Source dir.** All commands run in or commit to `~/.local/share/chezmoi` unless a target path is given.

## Pre-check (read-only)

- Is the file already managed? (`chezmoi status --<file>`)
- What's the target path and permissions?

## Workflow

1. Preview first: run `chezmoi status --<file>` (if managed) or `chezmoi add --dry-run <file>` to show what would happen, then `chezmoi cat <file>` to show the rendered result.
2. Ask: "manage this file?" — explicit confirmation required before any write.
3. If it's machine-specific, suggest a template version: rename to `<name>.tmpl`, add conditionals by OS/hostname/env.
4. Suggest naming conventions (`dot_` for `.` files, `private_` for 600 perms, `executable_` for 755, `symlink_` for symlinks).
5. Commit, in the source dir `~/.local/share/chezmoi`: `cd ~/.local/share/chezmoi && git add -A && git commit -m "add <file>" && git push`.
6. Verify the remote state matches with a final `chezmoi diff` (should be empty).

## Guard

If the file is already managed, show the current source and ask whether to overwrite or skip.

## Gotchas

1. **Interactive prompts block execution.** Always use `--refresh-externals --force` flags so chezmoi never hangs waiting for TTY input.
2. **Template errors.** Use `chezmoi execute-template '{{ .chezmoi.os }}'` to test template snippets in isolation.
3. **File not applying.** Check `chezmoi status` and `chezmoi diff` to see what changes are pending.
