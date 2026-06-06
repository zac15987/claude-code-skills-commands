# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is an **authoring repository** for the user's custom Claude Code skills and commands — not an application. There is no build, test, or lint step. "Working" on this repo means writing and editing Markdown definition files that get *copied* into a Claude Code config directory to take effect.

The files here are inert until installed. To activate a skill or command, it must be copied to:

| Scope | Commands | Skills |
|-------|----------|--------|
| Personal (all projects) | `~/.claude/commands/<name>.md` | `~/.claude/skills/<name>/SKILL.md` |
| Project (one project) | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |

Editing a file in this repo does **not** change live behavior — the installed copy must be updated. Keep this in mind when the user reports a skill "isn't working": check whether they edited the source here or the installed copy.

## Layout conventions

- **Skills** live at `skills/<name>/SKILL.md`. The directory name must match the `name:` in the frontmatter.
- **Commands** live at `commands/<name>.md` (one file, no subdirectory). The `commands/` directory currently exists but is empty.
- When a command and a skill share a name, the **skill wins**.

## SKILL.md structure

Every skill is a single Markdown file with YAML frontmatter:

```yaml
---
name: <kebab-case, must match directory name>
description: >
  When to trigger this skill, phrased so Claude can auto-invoke it during
  conversation. Include concrete trigger phrases (including Chinese ones the
  user actually says) and disambiguating SKIP conditions.
---
```

The `description` is load-bearing: it is the *only* thing Claude sees when deciding whether to auto-trigger the skill (the body is read only after invocation). When editing a skill's triggering behavior, edit the `description` — not the body. Good descriptions enumerate positive triggers, exact user phrases, and explicit negative/SKIP cases (see `csharp-deep-dive` for the pattern).

The body below the frontmatter is the actual instruction set Claude follows once the skill is invoked. Favor decision tables and concrete workflows over prose.

## Environment note (Windows)

The user develops on Windows with PowerShell as the default shell. This repo already encodes a hard-won lesson about it in `skills/git-push/SKILL.md`: **never use PowerShell here-string syntax (`@'...'@`) inside the Bash tool** — Bash treats `@` literally, leaks stray characters into commit messages, and still exits 0 (silent corruption). Prefer repeated `-m` flags for multi-line commits. Apply the same care anywhere this repo's skills or commands generate shell input.
