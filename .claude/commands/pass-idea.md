Mark an idea as deliberately skipped (status: pass).

Usage: /pass-idea <slug>

The idea-validator repo lives at: /home/thibanir/Project/portfolio/idea-validator/

The slug to act on is: $ARGUMENTS

Follow Mode 4 in `/home/thibanir/Project/portfolio/idea-validator/CLAUDE.md`:

1. Read `/home/thibanir/Project/portfolio/idea-validator/ideas-index.md`.
2. Find the table row where the Slug column matches $ARGUMENTS. If not found, tell the user and list the closest slug names.
3. Update that row: Status → `pass`, and prepend `Passed — ` to the Verdict text.
4. Write the updated ideas-index.md.
5. Confirm: `✓ [slug] marked as pass. It will be excluded from future generation rounds.`
