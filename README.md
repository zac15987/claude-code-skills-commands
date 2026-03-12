# claude-code-skills-commands

My custom Claude Code skills and commands.

## Contents

### Commands

- **git-push** (`commands/git-push.md`) — Stage, commit with conventional commit prefix, and push to remote. Supports specifying commit message language.

### Skills

- **csharp-deep-dive** (`skills/csharp-deep-dive/SKILL.md`) — Deep C# code intelligence using LSP to trace call chains, type relationships, and symbol definitions. Escalates to .NET reference source when needed.

## How to Use

Claude Code supports two types of custom extensions: **Commands** and **Skills**. Both can be invoked by typing `/name` in Claude Code interactive mode.

> Official docs: https://code.claude.com/docs/en/skills

### Installation

Copy the files to the corresponding paths to enable them:

| Scope | Commands | Skills |
|-------|----------|--------|
| **Personal** (all projects) | `~/.claude/commands/<name>.md` | `~/.claude/skills/<name>/SKILL.md` |
| **Project** (current project only) | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |

For example, to install everything from this repo at the user level:

```bash
# Commands
cp commands/git-push.md ~/.claude/commands/git-push.md

# Skills
mkdir -p ~/.claude/skills/csharp-deep-dive
cp skills/csharp-deep-dive/SKILL.md ~/.claude/skills/csharp-deep-dive/SKILL.md
```

### Invoking

In Claude Code interactive mode:

```
/git-push zh-tw          # Commit in Traditional Chinese and push
/csharp-deep-dive        # Start deep C# analysis
```

- When a Command and a Skill share the same name, the **Skill takes precedence**.
- Skills with a `description` field can be automatically triggered by Claude during conversation, without manual invocation.
