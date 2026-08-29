Kick off a startup idea validation across 6 expert personas.

The idea-validator repo lives at: /home/thibanir/Project/portfolio/idea-validator/

If no idea is present in the user's message ($ARGUMENTS is empty or doesn't contain an idea), respond with:

> I'm here to help you validate your next big idea. Paste your elevator pitch (or the path to a `.md` file containing it) and I'll kick off a full investigation across 6 expert perspectives: market analysis, bootstrapper viability, technical feasibility, GTM viability, brand/domain, and a devil's advocate.

If an idea or elevator pitch is present in $ARGUMENTS, proceed immediately:

1. Generate a slug from the first 3–4 significant words (lowercase, hyphenated).
2. Create `/home/thibanir/Project/portfolio/idea-validator/sessions/[slug]/` and write the idea to `sessions/[slug]/idea.md`.
3. Spawn these 6 agents in parallel using the Agent tool. Each agent reads their persona file from `/home/thibanir/Project/portfolio/idea-validator/personas/[persona].md`, researches the idea using WebSearch and WebFetch, and writes findings to `/home/thibanir/Project/portfolio/idea-validator/sessions/[slug]/[persona].md`:
   - Market Analyst (`market-analyst`)
   - Bootstrapper / Indie Economist (`bootstrapper`)
   - Technical Feasibility Expert (`technical-expert`)
   - GTM & Marketing Expert (`gtm-marketing`)
   - Brand & Domain Expert (`brand-domain`)
   - Devil's Advocate (`devil-advocate`)
4. Synthesize all findings into `/home/thibanir/Project/portfolio/idea-validator/sessions/[slug]/summary.md`.
5. Print the idea (the full contents of `sessions/[slug]/idea.md`) under a `## Idea` heading before presenting the summary.
6. Present the summary and enter debate mode as panel moderator.

Full orchestration instructions are in `/home/thibanir/Project/portfolio/idea-validator/CLAUDE.md`.
