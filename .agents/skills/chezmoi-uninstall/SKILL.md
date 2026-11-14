---
name: chezmoi-uninstall
description: Uninstall software that was managed with chezmoi — discover present state, deploy uninstallers hooked into cleanup scripts. Use when you want to remove an app and its dotfiles from the machine. Triggers on 'uninstall', 'remove', 'decommission'.
license: UNKNOWN
---

# `/skill:chezmoi-uninstall <app>` — Uninstall Managed Software

The reverse of `chezmoi-install`: discover what's installed, then deploy uninstaller scripts that run during cleanup and remove the software idempotently.

## Pre-check (read-only)

Report what exists on the machine and whether it is still managed by chezmoi:

| Piece | Present | Managed by chezmoi? |
|---|---|---|
| Binary / package | `command -v my-app` | — |
| Unit file | `ls ~/.config/systemd/user/app.service` | enabled? running? |
| dot_files | `chezmoi find <app>` | in the source dir? |
| Uninstaller | `ls .chezmoiscripts/run_9_cleanup_<app>.sh` or `uninstall/<app>_uninstall.sh` | already deployed? |

## Uninstaller deployment

An uninstaller is deployed to `uninstaller/<app>_uninstall.sh`, then hooked into cleanup so it runs on every `chezmoi apply`:

```bash
#!/bin/bash
# uninstaller/my-app_uninstall.sh — idempotent, safe on every apply
set -euo pipefail

# stop + disable the unit (mirror of `systemctl --user enable app --now`)
if systemctl --user is-enabled my-app >/dev/null 2>&1; then
  systemctl --user disable my-app --now
fi
rm -f ~/.config/systemd/user/my-app.service 2>/dev/null || true

# remove installed packages and leftovers
sudo apt remove -y my-app 2>/dev/null || true
rm -rf ~/.cache/my-app-old 2>/dev/null || true
```

## Hook into cleanup

Two options, mirroring what's already in the source dir:

1. **Cleanup script** (per app) — `run_9_cleanup_<app>.sh` in `.chezmoiscripts/`, runs last in the cleanup phase like the other `run_9_cleanup_*` scripts:
```bash
#!/bin/bash
bash "$CHEZMOI_SOURCE_DIR/uninstaller/my-app_uninstall.sh"
```

2. **Uninstaller dispatch** — add to `run_once_9_uninstallers.sh` alongside the existing ones:
```bash
bash "$CHEZMOI_SOURCE_DIR/uninstaller/my-app_uninstall.sh"
```

## Guard

- Unit state is read-only during pre-check; disable/stop happens in the uninstaller.
- `dot_file`s are not removed by this skill — manage their removal in your source dir with `chezmoi remove`.
- Uninstallers are idempotent: they can stay hooked into cleanup forever without doing harm.
