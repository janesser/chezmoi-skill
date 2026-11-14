---
name: chezmoi-refactor
description: Bring unmanaged setup under chezmoi management — edit a file at its target location, or set up a userspace systemd service with prerequisites and activation. Use when you want to manage an existing config or set up a new user-level service. Triggers on 'edit', 'systemd', 'userspace service', 'enable service', 'manage', 'refactor'.
license: UNKNOWN
---

# `/skill:chezmoi-refactor <target>` — Bring Setup Under chezmoi Management

Take something not managed by chezmoi and make it managed, at file level or service level. Two patterns, one goal: `edit → manage → verify`.

## Dispatch

- A **path** (`~/.config/nvim`, a dotfile) → **File** pattern below.
- An **app name** (a service you want to run) → **Service** pattern below.

## 1. File: edit at target, then manage it

Edit the file at its actual location on disk, then add it to the source dir. For an existing dot_file this is an update; for an unmanaged file this is a migration.

1. Edit at target (e.g. `$EDITOR ~/.config/nvim/init.vim`).
2. `chezmoi add <path>` — use the `private_` prefix for 600 perms, `dot_` for `.files`, or `symlink_`/`executable_` as needed. Encryption is preserved by default.
3. Review the managed state: `git diff HEAD` in the source dir should show the file added (or its new version).

## 2. Service: unit file and config as dot_file, activation by script

Set up a userspace systemd unit following the install lifecycle — prerequisites before targets, activation after targets. The ordering rationale is in `chezmoi-review`'s script-ordering analysis.

```
.chezmoiscripts/
├── run_once_before_00_install_prereqs.sh     # binaries, fonts, or packages the unit needs
├── dot_config_systemd_user_app_service       # unit file — managed as dot_file
├── dot_config_app_config.toml                # app config
└── run_after_00_enable_unit.sh               # activation
```

1. **Prerequisites** — install what the unit depends on in `run_once_before_00_<app>_prereqs.sh`. It runs once per machine, before any targets are applied.
2. **Unit file and config** — write both as `dot_file`s. chezmoi applies them between the before and after phases, so they're on disk when activation runs. The unit file is the declarative state; don't touch it with a script.
3. **Activation** — `run_after_00_enable_<app>.sh` runs after all targets are applied and uses `systemctl --user enable app --now`:

```bash
#!/bin/bash
set -euo pipefail
systemctl --user enable my-app --now
```

Enabling creates the `.wants` symlink; `--now` starts the unit in one call, so no separate daemon-reload or start step. The command is idempotent — already-enabled units are left alone.

4. **Cleanup** — `run_after_9_cleanup_<app>.sh` runs last within the after phase (chezmoi sorts lexicographically: `9` > `00`–`89`), removing stale artifacts from a previous version without touching state created by earlier scripts.

## Guard

- A `run_once_*` script's path is part of chezmoi's state key — renaming it makes it run again on every machine.
- Activation only works when the unit file exists on disk, which the dot_file management guarantees before the after phase.
- The ordering is deterministic only within a phase; never rely on ordering across phases.
