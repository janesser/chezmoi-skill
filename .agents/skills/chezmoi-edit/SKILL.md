---
name: chezmoi-edit
description: Edit a file at its target location, then register or update it in chezmoi management — preserving encryption for sensitive files, and always showing git diff HEAD for review before committing. Use when you want to change a file (managed or not) via an editor and add the result to your dotfiles. Triggers on 'edit', 'edit this file', 'chezmoi edit', 'change config', 'edit a dotfile'.
license: UNKNOWN
---

# `/skill:chezmoi-edit <target>` — Edit at Target, Add to Dotfiles

Edit a file where it lives (its target location), then commit the result into chezmoi's source state — with encryption preserved and `git diff HEAD` always shown for review.

## Core principles

1. **Edit at target.** The editor opens the real file (`~/.config/...`, not the source copy).
2. **Add after edit.** Once the editor closes, `chezmoi add` updates the source state from what you just wrote.
3. **Review before commit.** `git diff HEAD` is always shown; nothing is committed without your confirmation.

## Workflow

1. **Discover.** Check whether the target is managed:
   - `chezmoi status --<target>` — if the output contains "not managed", the target will be added fresh.
   - Otherwise it is managed (and, if the file does not exist yet at the target, continue to step 2 anyway).

2. **Edit.** Open the target in your editor:
   - `$EDITOR <target>`, falling back to `$VISUAL` or `vim`.
   - If the target does not exist yet, create an empty file first (`touch <target>`).

3. **Update source state** (in `~/.local/share/chezmoi`, or wherever the source dir is):
   - Managed + encrypted source entry → `chezmoi add --encrypt --force --no-tty <target>`. This keeps the `.age` file instead of replacing it with a plain copy.
   - Managed + plain source entry → `chezmoi add --force --no-tty <target>`.
   - Unmanaged → `chezmoi add --secrets error --force --no-tty <target>`; if secrets are flagged, re-run with `--encrypt`.

   Detection of the encrypted case (run before step 3):
   - `chezmoi add --dry-run --no-tty <target> 2>&1 | grep -q "remove encrypted attribute"` — if it matches, the target is managed by an encrypted source entry.

4. **Review.** Always show the diff to the user, then ask for confirmation before committing:
   - `git diff HEAD` (in the source dir) — displayed every time, even when empty ("no changes").
   - New files show up in `git status --porcelain | grep '??'` but not in the diff — display them too.
   - On explicit `yes`, commit + push: `git add -A && git commit -m "edit <target>" && git push`.

## Guard

- Stop before committing if `git diff HEAD` shows a deletion of a source file you didn't intend, or if the user says no.
- Never auto-commit without displaying `git diff HEAD` first.

## Gotchas

1. **Interactive prompts block execution.** Always use `--force --no-tty` so `chezmoi add` never hangs waiting for TTY input.
2. **Templates.** If the source entry is a template (`.toml`/`.tmpl`), editing at the target and re-adding replaces it with a plain file — use `chezmoi edit <target>` instead for templates.
3. **Symlinks.** `chezmoi add` on a symlinked target copies the link content by default; pass `-f --follow` to follow it.
4. **Unmanaged + private.** For unmanaged targets that look sensitive (`~/.ssh/*`, `.gnupg/*`, `.age`), `--secrets error` flags them — the review step shows the intent before you confirm.

## Invocations

```
/skill:chezmoi-edit ~/.config/hypr/hyprland.conf
/skill:chezmoi-edit ~/.gitconfig
/skill:chezmoi-edit ~/.ssh/id_rsa
```
