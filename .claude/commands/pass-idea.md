Mark an idea as deliberately skipped (status: pass).

Usage: /pass-idea <slug>

All paths below are relative to the idea-validator repo root (this repo).

The slug to act on is: $ARGUMENTS

Follow Mode 4 in `CLAUDE.md`:

1. Read `ideas-index.md`.
2. Find the table row where the Slug column matches $ARGUMENTS. If not found, tell the user and list the closest slug names.
3. Update that row: Status → `pass`, and prepend `Passed — ` to the Verdict text.
4. Write the updated ideas-index.md.
5. Confirm: `✓ [slug] marked as pass. It will be excluded from future generation rounds.`
