# Pi Agent Skills Reference

Key facts for Pi Agent compatibility, from [pi.dev/docs/latest/skills](https://pi.dev/docs/latest/skills):

- Pi implements the [Agent Skills specification](https://agentskills.io/specification) — `SKILL.md` with YAML frontmatter + Markdown body.
- Names: lowercase letters, numbers, hyphens, ≤ 64 chars, no leading/trailing/consecutive hyphens.
- Descriptions: ≤ 1024 chars, must describe what the skill does **and** when it applies.
- Skills are discovered in `~/.agents/skills/`, `.agents/skills/` (project-local), and from working directory ancestors.
- Loaded via `/skill:name` — arguments after the name are appended as a user request.
- `allowed-tools`: experimental pre-approved tool list (Pi gates Bash commands with this).
- Reload skills during a session: `/reload`.
