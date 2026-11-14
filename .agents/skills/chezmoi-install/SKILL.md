---
name: chezmoi-install
description: Manage the install lifecycle of software via chezmoi scripts — prerequisites by run_once, activation by run_after, cleanup by run_9_cleanup_. Use when a setup installs packages, binaries, or services (e.g. a userspace systemd unit). Triggers on 'install', 'setup', 'package', 'upgrade'.
license: UNKNOWN
---

# `/skill:chezmoi-install <app>` — Manage the Install Lifecycle

Handle one software dependency from install through upgrade and cleanup using chezmoi's script phases and lexicographic ordering. Start by reading present state, then verify how each piece is managed.

## Pre-check (read-only)

Report what exists on the machine and whether each piece follows the pattern:

| Piece | Present | Managed by chezmoi? |
|---|---|---|
| Binary / package | `command -v my-app` | — |
| Unit file | `ls ~/.config/systemd/user/app.service` | is it a `dot_file` in the source dir? |
| App config | `ls ~/.config/app/config.toml` | is it a `dot_file` in the source dir? |
| Prereqs script | `ls .chezmoiscripts/run_once_before_00_*` | matches `run_once_before_00_<app>_prereqs.sh`? |
| Activation script | `ls .chezmoiscripts/run_after_00_*` | matches `run_after_00_enable_<app>.sh`, uses `systemctl --user enable app --now`? |
| Cleanup script | `ls .chezmoiscripts/run_after_9_cleanup_*` | matches `run_after_9_cleanup_<app>.sh`? |

If a piece is missing, it's added by the pattern below. If it exists but isn't managed by chezmoi, move it (edit at target, then `chezmoi add`) before proceeding. Unit state is read-only here: `systemctl --user is-enabled my-app`, `systemctl --user status my-app`.

## Findings

Present but not managed → migrate first (edit at target, `chezmoi add`). Missing → follow the pattern below. Fully managed → only upgrades: write a new `run_once_00` script and adjust `dot_file`s, then `chezmoi apply` handles the rest.

## The pattern

```
.chezmoiscripts/
├── run_once_before_00_install_prereqs.sh     # binaries this setup needs
├── dot_<path-to-unit>                        # unit file (dot_file)
├── dot_<path-to-config>                      # app config (dot_file)
└── run_after_00_enable_unit.sh               # activation
```

The order comes from chezmoi's ordering technique: `run_before_*` runs before targets, targets are applied in ASCII order, and `run_after_*` runs last — all sorted lexicographically within their phase. A numeric prefix (`00`, `9`) encodes the sequence deterministically.

## 1. Prerequisites: `run_once_before_00_<app>_prereqs.sh`

`run_once_*` scripts run once per machine, keyed by content hash — re-run if the content changes. The `before_` attribute places them before all targets are applied. Numbered `00` so they run first within the phase.

```bash
#!/bin/bash
set -euo pipefail
if ! command -v my-app >/dev/null 2>&1; then
  sudo apt install -y my-app
fi
```

## 2. Unit file and config: `dot_file`

The `.config/systemd/user/app.service` target is a `dot_file` — managed by chezmoi itself, idempotent, no script needed for the file. Its activation lives in the phase after file updates.

## 3. Activation: `run_after_00_enable_<app>.sh`

Runs after targets are applied (the unit file exists on disk), so it can safely enable and start it. Uses `systemctl --user enable app --now` — one idempotent call for both. Within the after phase, `00` sorts before higher numbers, and this phase runs last in `chezmoi apply`.

```bash
#!/bin/bash
set -euo pipefail
systemctl --user enable my-app --now
```

## 4. Cleanup: `run_after_9_cleanup_<app>.sh`

Runs last in the after phase: chezmoi sorts within-phase scripts lexicographically, and `9` sorts after every lower digit (`00`–`89`). If cleanup ran earlier it could delete artifacts a later script creates.

```bash
#!/bin/bash
set -euo pipefail
rm -rf ~/.cache/my-app-old 2>/dev/null || true
```

## Upgrades

`run_once_*` is keyed by content hash, not filename — changing the script's contents makes it run again on every machine. An upgrade path is therefore: write the new `run_once_00` script (new version), add/adjust `dot_file`s as needed, and let the next `chezmoi apply` handle the transition. The cleanup step removes stale state from the previous version.

## Self-manage

The skill bundle itself is kept in sync by a chezmoi script in your source dir — `.chezmoiscripts/run_once_5_install_skills.sh` (the repo's git remote, `git@github.com:janesser/chezmoi-skill.git`, from `git remote -v`). It settles next to the `aitools` group, since skills are agent infrastructure too. It fetches the latest source and copies each skill into `~/.agents/skills/<name>/`, with OpenCode's symlinks in `~/.codex/skills/` re-created. Idempotent: on the next `chezmoi apply` it refreshes nothing but itself.

## Guard

- Moving or renaming a `run_once_*` script changes its state key — it runs again on every machine.
- Ordering is deterministic only within a phase; never rely on ordering across phases.
