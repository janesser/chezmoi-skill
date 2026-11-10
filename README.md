# chezmoi — Dotfile Management Skill Bundle

Seven focused agent skills that together cover the full workflow for managing dotfiles with [chezmoi](https://www.chezmoi.io). Each has its own entry point in Pi Agent; they share principles and reference each other's docs.

## Entry points

| Entry | What it does |
|---|---|
| `/skill:chezmoi-status` | Read-only state view — what's managed, what's pending |
| `/skill:chezmoi-init` | Guided initialization (fresh or existing) |
| `/skill:chezmoi-update` | Update dotfiles incl. externals |
| `/skill:chezmoi-add <file>` | Add a file → preview → commit → push |
| `/skill:chezmoi-review` | Review your setup with suggestions (incl. script order) |
| `/skill:chezmoi-apply [target]` | Apply pending changes, optionally scoped to a path |
| `/skill:chezmoi-edit <file>` | Edit at target location → `chezmoi add` (encryption preserved) → `git diff HEAD` review |

The skill bundle discovers your current state at runtime — it knows nothing about your setup until it reads it. Nothing is written without your explicit confirmation.

## Installation

The bundle follows the [Agent Skills specification](https://agentskills.io/specification) (`SKILL.md` with YAML frontmatter + Markdown body), so it works across multiple agents. Each skill installs into its own directory:

```bash
for s in status init update add review apply edit; do
  mkdir -p ~/.agents/skills/chezmoi-$s
  cp ~/projs/chezmoi-skill/.agents/skills/chezmoi-$s/SKILL.md ~/.agents/skills/chezmoi-$s/SKILL.md
done
```

### Pi Agent (this environment)

Pi Agent discovers skills in `~/.agents/skills/` and `.agents/skills/` (project-local). Reload after installing: `/reload`.

### Claude Code

Install into the agent's own skills directory. The `SKILL.md` follows the open standard — any compatible agent will load it. Each skill is a separate entry in your agent's skill list, so no slash-command prefixes are needed.

## Safety principles

1. **Discover before acting** — every command starts by reading your current state
2. **Explicit confirmation before change** — no write operation without preview and a `yes`
3. **Dry-run first** — every modifying command is previewed with `chezmoi diff`/`chezmoi cat` before it runs
4. **One bounded workflow per command** — each command covers a complete end-to-end path

## Prerequisites

- `chezmoi` must be installed (`~/.local/share/chezmoi` or custom source dir)
- `git` configured for committing dotfile changes
- SSH authentication available (via `ssh-agent` or `SSH_AUTH_SOCK`)

## Project structure

```
~/projs/chezmoi-skill/
├── SKILL.md                    # This file — the bundle landing page
├── README.md                   # This file — installation & usage docs
├── references/pi-skills.md     # Pi Agent compatibility reference
└── .agents/skills/<name>/      # One directory per skill (seven total)
    ├── chezmoi-status/SKILL.md
    ├── chezmoi-init/SKILL.md
    ├── chezmoi-update/SKILL.md
    ├── chezmoi-add/SKILL.md
    ├── chezmoi-edit/SKILL.md
    ├── chezmoi-review/SKILL.md
    └── chezmoi-apply/SKILL.md
```

## Validation

Every skill lints clean against the Agent Skills spec:

```bash
skillmd lint .agents/skills/         # 7 clean · avg score 100/100
skillmd scan .agents/skills/         # docs_only — no scripts executed
```

## License

MIT
