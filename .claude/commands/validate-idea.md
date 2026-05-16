Kick off a startup idea validation across 6 expert personas.

If no idea is present in the user's message ($ARGUMENTS is empty or doesn't contain an idea), respond with:

> I'm here to help you validate your next big idea. Paste your elevator pitch (or the path to a `.md` file containing it) and I'll kick off a full investigation across 6 expert perspectives: market analysis, VC attractiveness, technical feasibility, GTM viability, brand/domain, and a devil's advocate.

If an idea or elevator pitch is present in $ARGUMENTS, proceed immediately:

1. Generate a slug from the first 3–4 significant words (lowercase, hyphenated).
2. Create `_idea-validator/sessions/[slug]/` and write the idea to `_idea-validator/sessions/[slug]/idea.md`.
3. Spawn these 6 agents in parallel using the Agent tool. Each agent reads their persona file from `_idea-validator/personas/[persona].md`, researches the idea using WebSearch and WebFetch, and writes findings to `_idea-validator/sessions/[slug]/[persona].md`:
   - Market Analyst (`market-analyst`)
   - VC Investor (`vc-investor`)
   - Technical Feasibility Expert (`technical-expert`)
   - GTM & Marketing Expert (`gtm-marketing`)
   - Brand & Domain Expert (`brand-domain`)
   - Devil's Advocate (`devil-advocate`)
4. Synthesize all findings into `_idea-validator/sessions/[slug]/summary.md`.
5. Present the summary and enter debate mode as panel moderator.

Full orchestration instructions are in `_idea-validator/CLAUDE.md`.
