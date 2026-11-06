---
name: chezmoi
description: "A bundle of six agent skills for managing dotfiles with chezmoi — status, init, update, add, review and apply. Each action is its own /skill: entrypoint. Use when you want to manage dotfiles with chezmoi: setup, sync, add files, or review your config."
---

# Chezmoi Dotfile Management Skill Bundle

Six focused skills that together cover the full chezmoi workflow. Each has its own entry point in Pi Agent; they share principles and reference each other's docs.

## Entry points

Each skill is a directory with its own `SKILL.md`, so Pi advertises it separately:

| Entry | What it does |
|---|---|
| `/skill:chezmoi-status` | Read-only state view — what's managed, what's pending |
| `/skill:chezmoi-init` | Guided initialization (fresh or existing) |
| `/skill:chezmoi-update` | Update dotfiles incl. externals |
| `/skill:chezmoi-add <file>` | Add a file → preview → commit → push |
| `/skill:chezmoi-review` | Review your setup with suggestions (incl. script order) |
| `/skill:chezmoi-apply [target]` | Apply pending changes, optionally scoped to a path |

## Principles shared by all six

1. **Discover before acting.** Every command starts by reading the user's current state (`chezmoi status`, `chezmoi diff`, `.chezmoi.toml`).
2. **Explicit confirmation before change.** No write operation without a preview and an explicit `yes`.
3. **Dry-run first.** Everything modifying is previewed with `chezmoi diff`/`chezmoi cat` before it runs.

## Project structure

```
~/projs/chezmoi-skill/
├── SKILL.md                    # This file — the bundle landing page
├── README.md                   # Installation & usage docs
├── references/pi-skills.md     # Pi Agent compatibility reference
└── .agents/skills/<name>/      # One directory per skill (six total)
    ├── chezmoi-status/SKILL.md
    ├── chezmoi-init/SKILL.md
    ├── chezmoi-update/SKILL.md
    ├── chezmoi-add/SKILL.md
    ├── chezmoi-review/SKILL.md
    └── chezmoi-apply/SKILL.md
```

## Installing the bundle

Each skill installs into its own directory — see [README.md](README.md) for the full install script. Then reload: `/reload`.

## Validation

Every skill lints clean against the Agent Skills spec:

```bash
skillmd lint .agents/skills/         # 6 clean · avg score 98-100/100
skillmd scan .agents/skills/         # docs_only — no scripts executed
```

## License

MIT
