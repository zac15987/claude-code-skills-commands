---
name: git-push
description: >
  Stage the changes relevant to the current task, commit with a conventional commit
  prefix and description, then push to the remote. Use when the user asks to commit and
  push, "git push", "幫我提交並推送", or similar. The commit message language can be
  specified as an argument (e.g. "en-us" for English, "zh-tw" for Traditional Chinese);
  default to the user's last-used or preferred language when none is given.
---

# Git Push

Stage the changes relevant to the current task, commit with a conventional commit prefix and description, then push to the remote.

The commit message language is taken from the invocation argument (e.g. `en-us` for English, `zh-tw` for Traditional Chinese). If none is given, use the user's preferred language.

## Rules

- Run `git status` and `git diff` first to understand the changes.
- **Stage based on context, not blindly `git add -A`.** Determine which changed files belong to the current task (the work just discussed/done in this session) and stage only those.
  - If the working tree contains unrelated changes, or it's ambiguous which files belong together, **stop and ask the user** which files to include rather than guessing.
  - Never stage sensitive files (e.g. `.env`, credentials, key files) — flag them to the user if present.
- Choose an appropriate conventional commit prefix (feat, fix, refactor, docs, style, test, chore, perf, ci, build, revert).
- Write the commit description in the specified language.
- Commit and then push to the current remote branch.

## Multi-line commit messages (Windows — known footgun, claude-code issue #65162)

- NEVER use PowerShell here-string syntax (`@'...'@`) inside the **Bash** tool. Bash treats `@` as a literal char, so stray `@` leak into the message head/tail and the command still exits 0 (silent corruption).
- Prefer **repeated `-m` flags** (one per paragraph) — shell-agnostic, no quoting/here-string pitfalls. This is the default.
- If you need precise line breaks within a paragraph: use the **PowerShell** tool with a real `@'...'@` here-string (closing `'@` at column 0), OR bash `$'line1\nline2'`. Do not mix shell idioms.
- After committing a multi-line message, self-verify with `git log -1 --format=%B` before pushing.
