Stage all changes, commit with a conventional commit prefix and description, then push to the remote.

The commit message language should be: $ARGUMENTS

Rules:
- Run `git status` and `git diff` first to understand the changes
- Choose an appropriate conventional commit prefix (feat, fix, refactor, docs, style, test, chore, perf, ci, build, revert)
- Write the commit description in the specified language (e.g. "en-us" for English, "zh-tw" for Traditional Chinese)
- Stage only the relevant changed files (avoid staging sensitive files like .env or credentials)
- Commit and then push to the current remote branch
