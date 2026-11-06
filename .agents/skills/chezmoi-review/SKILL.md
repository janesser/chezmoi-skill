---
name: chezmoi-review
description: Review your existing dotfile setup with specific suggestions — naming consistency, grouping, machine-specific value, security, secret scanning, encrypted data hygiene, template hygiene, externals, and script execution order. Use when you want feedback on your chezmoi setup. Triggers on 'review', 'suggestions', 'audit my dotfiles', 'what should I improve'.
license: UNKNOWN
---

# `/skill:chezmoi-review` — Review Existing Dotfiles

Reviews your current dotfile setup and suggests improvements. This is where the skill has "grip": it knows your state and gives specific feedback.

## Script ordering analysis

In addition to reviewing files, inspect `.chezmoiscripts/` execution order. First present a **sequential list of what chezmoi would execute** (numbered by phase then name), then determine why each script runs in its position (by `run_*` numbering convention) and question whether sequential order makes sense:

- Are dependencies satisfied by the sequence? (e.g., a cleanup script that removes what an install script created)
- Does the ordering follow the convention (`run_0_*` → `run_once_1_*` … `run_9_*`)?
- Do scripts at a given position actually perform work consistent with that phase?

Report findings as: `[ORDER] <finding>` — e.g., "[ORDER] run_9_cleanup_uv.sh runs after install but uv is installed in run_once_0_python_tools_install.sh"

## Pre-check (read-only)

All paths below are in the source dir `~/.local/share/chezmoi`, which is presumed.

- `chezmoi status` — full inventory of managed/unmanaged files.
- Iterate the source dir `~/.local/share/chezmoi` to understand naming conventions.
- Read `.chezmoi.toml.tmpl` for configuration patterns.
- **Read `.chezmoiignore`** — every path it lists is excluded from chezmoi's view. Do not report findings on ignored paths; they are not part of the dotfiles.
- Check for a secret-scanner in the git repo (pre-commit hook with `trufflehog`, `gitleaks`, `detect-secrets`, …); if none exists, flag it as a repo-wide disclosure risk.
- Count encrypted files (`*.age` and any other opaque encryption); if present, prepare to flag the AI-model-provider disclosure risk.

## Ignored paths

Any path listed in `.chezmoiignore` (or matched by a glob there) is **not managed by chezmoi**, so review findings must exclude it: files, directories and their contents under an ignored path. In practice this means skipping:

- `.pre-commit-config.yaml` — a tool config, not a dotfile
- `etc`, `usr` — package-managed paths
- `commons`, `discoveries`, `devices`, `uninstaller` — shared/doc/anchor trees
- `LICENSE`, `COMPILE.md`, `README.md`, `AGENTS.md` — repo documentation

An ignored path must never produce a naming, grouping, security or template finding.

## Review categories

1. **Naming consistency:** "Why is this a script?" — point out files that should be `executable_` prefixed, or files that should use `dot_` prefix.
2. **Grouping:** "These scripts belong together" — group related files into logical units (e.g., all shell configs under one `home_dir`).
3. **Machine-specific value:** "This file is identical on all your machines — consider making it static instead of a template."
4. **Security:** "This private key is stored without `private_` prefix" or "this config exposes a path that should be secret".
5. **Secret scanning (git repo):** if no secret-scanner pre-commit hook (e.g. `trufflehog`, `gitleaks`, `detect-secrets`) exists in the dotfile git repo, suggest adding one — an accidental commit leaks secrets into a public or shared git history. Report as `[SECRET-SCANNING] <finding>`. This check is an exception to the exclusion rule: it must detect whether such a hook exists (reading `.pre-commit-config.yaml` for hooks), but reports only on repo posture and never scans the file's contents for dotfile issues.
6. **Encrypted data hygiene:** if chezmoi manages encrypted files (`*.age` or other opaque encryption), flag that the ciphertext still leaks metadata to whatever decrypts it (e.g. an AI model provider) and recommend verifying the provider's privacy terms. Report as `[ENCRYPTED] <finding>`.
7. **Template hygiene:** "This template uses `{{ }}` but doesn't need to" or "escaping issues in nested templates".
8. **Externals:** "You have 3 external git repos — one hasn't been refreshed in 30 days."
9. **Execution order:** "Script X runs before Y, but Y installs the dependency X needs" or "cleanup script at run_9_* should run earlier".

## Skip rule (ignored paths)

Report nothing for paths that `.chezmoiignore` excludes — they are not dotfiles, so a naming/grouping/security finding on them is noise. If an ignored path is *also* part of your real setup (e.g. `devices/<hostname>/` holds machine config), note it as context only, never as a fix to apply.

## Exclusion rule

Ignored paths are out of scope: never scan their contents for secrets, secret-scan coverage or any other review finding, and never mention them in the output. The `[SECRET-SCANNING]` check reads `.pre-commit-config.yaml` only to detect whether a hook exists — it reports repo posture, not the file's contents.

## Output format

One suggestion per line, with the file path and a short reason. The user can then decide which to act on — each becomes its own `/skill:chezmoi-add` or edit task.

## Gotchas

1. **Read-only by design.** Review never modifies anything.
2. **Interactive prompts block execution.** Always use `--refresh-externals --force` flags so chezmoi never hangs waiting for TTY input.
