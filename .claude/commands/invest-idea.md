Commit to building an idea: mark it as build, generate the build brief if missing, and surface the bootstrap command.

Usage: /invest-idea <slug>

The idea-validator repo lives at: /home/thibanir/Project/portfolio/idea-validator/

The slug to act on is: $ARGUMENTS

Follow Mode 5 in `/home/thibanir/Project/portfolio/idea-validator/CLAUDE.md`:

1. Read `/home/thibanir/Project/portfolio/idea-validator/ideas-index.md`.
2. Find the table row matching $ARGUMENTS. If not found, tell the user and list closest slugs.
3. Check whether `sessions/$ARGUMENTS/summary.md` exists. If not, warn the user and ask whether to proceed anyway.
4. Update that row: Status → `build`, prepend `Building — ` to the Verdict text.
5. Write the updated ideas-index.md.
6. Check whether `sessions/$ARGUMENTS/build-brief.md` exists. If not, run the /brief-idea flow (Mode 6) to generate it now — do not skip this step.
7. Print the exact bootstrap command using the build brief as the idea file:

```
Ready to launch. Run this from the portfolio root:

  ./scripts/new-product.sh $ARGUMENTS sessions/$ARGUMENTS/build-brief.md

Make sure ANTHROPIC_API_KEY and ORG_PAT are exported first.
```
