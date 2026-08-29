List all ideas tracked in ideas-index.md, sorted by overall score descending.

The idea-validator repo lives at: /home/thibanir/Project/portfolio/idea-validator/

Follow Mode 3 in `/home/thibanir/Project/portfolio/idea-validator/CLAUDE.md`:

1. Read `/home/thibanir/Project/portfolio/idea-validator/ideas-index.md`.
2. Parse every idea row from the table.
3. Sort by Overall score descending; ties broken by Bootstrap score. Unscored ideas go below scored ones. `pass` and `killed` ideas go last.
4. Render the ranked table with columns: Rank, Score, Status, Slug, One-liner (verdict truncated to ~60 chars).
5. Print the status legend at the bottom.
