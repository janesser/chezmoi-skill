---
name: chezmoi-init
description: Guided initialization of a chezmoi dotfile setup — from scratch or adapting an existing one. Use when bootstrapping a new machine, initializing `~/.local/share/chezmoi`, or migrating another dotfile manager to chezmoi. Triggers on 'init', 'set up dotfiles', 'bootstrap', 'new machine', 'chezmoi init'.
license: UNKNOWN
---

# `/skill:chezmoi-init` — Guided Initialization

Walks through setting up chezmoi from scratch, or adapting an existing setup.

## Core principles

1. **Discover before acting.** Every command starts by reading the user's current state (`chezmoi status`, `chezmoi diff`, `.chezmoi.toml`). All commands presume the source dir is `~/.local/share/chezmoi` unless told otherwise.
2. **Explicit confirmation before change.** No write operation happens without a preview and an explicit `yes`.

## Pre-check (read-only)

- Is `chezmoi` installed? (`which chezmoi`) — if not, offer installation instructions (`brew install chezmoi`, `sudo apt install chezmoi`, etc.) before proceeding.
- Does the source directory exist? (`ls ~/.local/share/chezmoi`)
- Does `~/.gitconfig` or another git setup exist for committing dotfile changes?
- Is there a `.chezmoi.toml.tmpl` already?
- What does `chezmoi status` show?

## Fresh setup (no prior state)

1. The source directory is `~/.local/share/chezmoi`. Create it if missing: `mkdir -p ~/.local/share/chezmoi`.
2. `chezmoi init --source ~/.local/share/chezmoi` (the default) or `cd ~/.local/share/chezmoi && git init`.
3. Guide through adding first dotfiles: for each file you want managed, run `chezmoi add <file>` and show the preview.
4. Run `chezmoi apply --refresh-externals --force` to verify nothing is broken.

## Existing setup (prior chezmoi present)

1. Show current state: `chezmoi status`, `chezmoi diff` — what's already managed, what's pending.
2. Identify gaps: files not in the source dir, templates that don't render.
3. Add missing pieces with explicit confirmation per file.
4. Run a final `chezmoi apply --refresh-externals --force` to confirm everything renders.

## Gotchas

1. **Interactive prompts block execution.** Always use `--refresh-externals --force` flags so chezmoi never hangs waiting for TTY input.
2. **Dry-run first.** Every modifying command is previewed with `chezmoi diff`/`chezmoi cat` before it runs.
