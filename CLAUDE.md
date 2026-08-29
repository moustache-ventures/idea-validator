> **The global CLAUDE.md rules always apply here too** (tool preferences, no `/tmp`, no `curl`, plan before code, etc.).

# Idea Lab — Orchestration

This directory is the central idea workspace: **generate** micro-SaaS candidates and **validate** them through a panel of 6 expert personas. All ideas — generated, validated, rejected, and pivoted — are tracked in `ideas-index.md` as a compounding knowledge base.

```
idea-validator/
├── personas/            ← expert personas (6 validators + 1 pre-validator)
├── profiles/            ← founder constraints (read before generating)
├── sessions/
│   ├── [slug]/          ← full validation sessions
│   └── generated/       ← pre-validator candidates from /generate-idea
│       └── [YYYY-MM-DD]/
└── ideas-index.md       ← master memory: every idea ever explored
```

---

## Mode 1 — Validate an idea (`/validate-idea`)

### On session start

Read `ideas-index.md` first. Check if a similar idea has already been explored — if so, surface the prior findings before starting fresh. If the user's message does not contain an idea or elevator pitch, respond with:

> I'm here to help you validate your next idea. Paste your elevator pitch (or the path to a `.md` file containing it) and I'll run a full investigation across 6 expert perspectives.

If an idea is present, go straight to investigation.

### Investigation flow

1. **Generate a slug** from the first 3–4 significant words, lowercase, hyphenated (e.g. `ai-invoice-parser`).
2. **Create `sessions/[slug]/`** and write the raw idea to `sessions/[slug]/idea.md`.
3. **Spawn all 6 persona agents in parallel** using the Agent tool.
4. **Wait for all 6 agents** to complete and write their output files.
5. **Synthesize** findings into `sessions/[slug]/summary.md`.
6. **Update `ideas-index.md`** — add or update the entry for this slug with verdict and scores.
7. **Present the summary** and enter debate mode.

### Agent prompts (spawn all 6 in parallel)

For each agent, fill in `[slug]` and `[persona-file]` — use absolute paths:

```
You are a specialist analyst validating a startup idea.

1. Read the idea: /home/thibanir/Project/portfolio/idea-validator/sessions/[slug]/idea.md
2. Read your persona and instructions: /home/thibanir/Project/portfolio/idea-validator/personas/[persona-file].md
3. Do real research using WebSearch and WebFetch — do not make things up.
4. Write your full findings to: /home/thibanir/Project/portfolio/idea-validator/sessions/[slug]/[persona-file].md

Be specific, cite sources, and include a score out of 10 for your dimension.
```

| Persona file | Output file |
|---|---|
| `market-analyst` | `sessions/[slug]/market-analyst.md` |
| `bootstrapper` | `sessions/[slug]/bootstrapper.md` |
| `technical-expert` | `sessions/[slug]/technical-expert.md` |
| `gtm-marketing` | `sessions/[slug]/gtm-marketing.md` |
| `brand-domain` | `sessions/[slug]/brand-domain.md` |
| `devil-advocate` | `sessions/[slug]/devil-advocate.md` |

### Summary format

```markdown
# [Idea Name] — Validation Report
*[date]*

## Verdict
[2–3 sentences: build / explore further / pass, and the single most important reason]

## Scores
| Dimension | Score | One-liner |
|---|---|---|
| Market opportunity | /10 | |
| Technical feasibility | /10 | |
| GTM viability (no social) | /10 | |
| Bootstrapper viability | /10 | |
| AI commoditisation risk | /10 (10 = very high risk) | |
| **Overall** | **/10** | |

## Key findings

### Market
[3–5 bullets from market analyst — size, trend, competition]

### Technical
[3–5 bullets — list each key integration as ✅ Available / ⚠️ Restricted / ❌ Unavailable]

### GTM
[3–5 bullets — primary channel without social presence, CAC estimate, first 90 days]

### Bootstrapper take
[3–5 bullets — MRR ceiling, maintenance burden, willingness to pay, AI commoditisation risk]

### Brand options
[Top 3 name candidates with domain status]

### Red flags
[Top 3 from devil's advocate — the most important ones]

## Open questions
[Things the panel couldn't answer that need founder research before proceeding]
```

### Debate mode

After presenting the summary, you are the **panel moderator**. You hold all 6 expert opinions in context.

- When the user challenges a finding, defend it from that expert's perspective with the evidence behind it
- If the user makes a compelling counter-argument, update your position and say so explicitly
- When the idea pivots significantly, offer to spawn a fresh investigation on the updated concept
- When the session concludes (verdict reached or user moves on), update `ideas-index.md` with the final verdict

**Saving debate notes:** Whenever a meaningful decision, pivot, or important caveat emerges from the debate, append it to `sessions/[slug]/debate-notes.md`. Use this format:

```markdown
- **[YYYY-MM-DD]** [What was decided or clarified — one sentence. Include the reasoning if non-obvious.]
```

Examples worth saving: scope decisions ("decided to exclude X from MVP"), pricing pivots, a red flag the founder addressed with new information, a channel the founder confirmed they have access to, a niche refinement. Do not save routine back-and-forth — only decisions that should inform the build.

### Resuming sessions

If `sessions/[slug]/` already exists with persona files present, read them directly rather than re-running. Only re-run if the user explicitly asks.

---

## Mode 2 — Generate ideas (`/generate-idea`)

**Portfolio model — apply throughout this entire mode:**
- Each product is built for under $300 and monetised via **niche-targeted paid ads** (Facebook/Instagram or Google). This is the primary GTM engine.
- The goal per site is **10–50 paying customers** generating **$200–$500/mo**. Not $5k/mo — that's not the model. 20 sites × $300/mo = $6k/mo total.
- **Competition is expected and normal.** Competitors validate the market. Free tools have ads, bad UX, and poor SEO — there is almost always a slice to grab. The only true kill signal is a free, ad-free, deeply-integrated tool from Google / Microsoft / Meta with no UX gap.
- **Smaller niche = lower CPCs = better ROI** on paid ads. Prefer specific, reachable niches over broad markets.
- Score ideas against this model, not against a venture-scale benchmark.

**Australian market focus — apply throughout this entire mode:**
- **Start by looking for gaps in Australia.** The AU market (~2.6M small businesses) is often 2–3 years behind the US/EU SaaS curve. Proven models that dominate in the US frequently have no credible AU-specific competitor yet.
- **AU-specific compliance = natural moat.** Tools that handle GST, BAS, ABN lookup, Fair Work Act obligations, or state-based licensing create switching costs that generic US tools can't easily replicate.
- **"Built for Aussies" is a real differentiator** for local SMBs — local support, AUD pricing, and AU-specific terminology (tradies, PAYG withholding, superannuation) build trust generic tools miss.
- **AU niche specificity = lower CPCs.** Geo-targeting AU audiences on Facebook/Google is cheaper than US/EU. A tool for AU tradies reaches a tighter, cheaper audience than "plumbers globally."
- **Global expansion is Phase 2, not Phase 1.** If an idea has AU-first viability, build and validate there first. Global reach is upside, not a launch requirement.
- Ideas with no AU angle are not eliminated — but AU-viable ideas rank higher in Step 4.

### On session start

1. Read `profiles/founder.md` for hard constraints.
2. **Read `ideas-index.md`** — note all previously explored ideas to avoid repeating them. Look for unexplored niches or pivot angles from ideas that scored high on some dimensions but failed on others.
3. **Check for a focus area:** If the user provided keywords or a theme (e.g. "parents with young kids", "pet owners"), note it — it drives Step 1 and the session directory name. If nothing was provided, proceed in default (broad) mode.

### Generation flow

#### Step 1 — Niche × pain matrix

> **Focused mode (focus area provided):** Skip the broad niche list below. Generate 12–15 candidates that all fall within the specified domain — cover different sub-niches and pain angles inside it. Apply the AU market lens, B2B/B2C balance guidance, and all portfolio constraints as usual. Proceed to Step 2.
>
> **Default mode (no focus area):** Follow the full niche × pain matrix below.

Generate 12–15 candidate ideas. Each = specific niche + specific pain. **Start by looking for gaps in Australia** — niches where US/EU tools are well-established but AU-specific competitors are thin or absent.

**Balance B2B and B2C.** Aim for roughly half the candidates targeting consumers or lifestyle niches, and half targeting small businesses or professionals. If the matrix is drifting heavily toward compliance/regulated professions, correct it — compliance is one moat, not the only one.

**Niches (AU-first, mix of B2B and B2C):**

*B2B — trade & professional services:*
- AU tradies: electricians (sparkies), plumbers, builders, tilers, painters, concreters, HVAC technicians, pest controllers, pool technicians, arborists
- AU compliance-heavy businesses: bookkeepers (BAS/GST), migration agents, NDIS providers, childcare operators, real estate agents, mortgage brokers
- AU local services: pet groomers, cleaners, landscapers, personal trainers, tattoo artists, nail salons, driving instructors
- AU solo professionals: designers, copywriters, photographers, videographers, nutritionists, consultants
- AU food & hospitality: food trucks, caterers, market stallholders, small bakeries, meal prep services

*B2C — consumers & lifestyle:*
- AU pet owners: dog owners, cat owners, reptile keepers, bird owners — vet reminders, breed-specific care, pet cost tracking
- AU renters & home buyers: bond calculators, rental application tools, suburb comparison, stamp duty calculators, moving checklists
- AU parents: school holiday activity finders, AU school term calendars, extracurricular scheduling, kids' meal planners
- AU outdoor & hobby: gardening by AU climate zone, native plant ID, surf/beach conditions, fishing spot logbooks, camping site finders
- AU health & fitness consumers: gym class booking helpers, AU Medicare/private health claim trackers, nutrition tools tailored to AU products
- AU travellers: road trip planners for AU routes, caravan park finders, working holiday visa trackers for backpackers

**Pains (B2B):** scheduling & booking, invoicing & payments (GST-compliant), client communication & follow-up, lead generation & quote requests, compliance & paperwork, staff/contractor scheduling, review & reputation management, contract & proposal generation.

**Pains (B2C):** tracking & reminders, cost calculation & comparison, planning & checklists, personalised recommendations, form-filling & applications, logging & journalling.

**Pains:** scheduling & booking, invoicing & payments, client communication & follow-up, lead generation & quote requests, compliance & paperwork, staff/contractor scheduling, inventory & reordering, review & reputation management, social media content generation, reporting & analytics, onboarding new customers, contract & proposal generation.

Write each as: `[Target customer] needs help with [specific pain] — idea: [product concept in 5–10 words].`

#### Step 2 — Hard constraint filter (eliminate before spending research tokens)

Apply each as binary yes/no. Eliminate any candidate failing ANY of these:

**Legal:** no legal/medical/financial/tax advice; no core HIPAA or PCI DSS dependency; no platform ToS scraping.

**Operational:** no two-sided marketplace; no physical goods; no UGC moderation at scale.

**GTM (founder has no social presence beyond personal Facebook):**
- Must have at least one viable paid-ads channel: Facebook/Instagram ads (local businesses, SMBs, consumers) or Google Search (high-intent keywords). Fallback: cold email with an obtainable list, or directory listings.
- Eliminate only if the *sole* realistic channel is Reddit, Twitter/X, LinkedIn groups, niche Slack/Discord, or audience building from zero — not just because those channels would help.

**Build:** solo-buildable MVP in 2–4 weeks; clear Stripe-chargeable model; estimated build cost under $300.

**Competition is NOT a filter criterion.** Do not eliminate ideas because competitors exist. Eliminate only if a free, ad-free, deeply-integrated tool from Google / Microsoft / Meta already solves this with no meaningful UX gap. Existing paid competitors = market confirmed.

**Avoid repeats:** cross-check against `ideas-index.md` — skip any niche+pain combination already explored unless proposing a meaningful pivot angle.

Target: 5–8 survivors.

#### Step 3 — Research surviving candidates in parallel

**Session directory:**
- Default mode: `sessions/generated/[YYYY-MM-DD]/` (append `b`, `c`, etc. if that date directory already exists)
- Focused mode: `sessions/generated/[YYYY-MM-DD]-[focus-slug]/` where `focus-slug` is the focus area kebab-cased (e.g. `parents-young-kids`)

Create the directory, then spawn one research agent per surviving candidate in parallel.

Prompt template for each agent:

```
You are a startup pre-validator doing quick web research on a single idea.

Portfolio model context (apply throughout your research):
- Each product is built for under $300 and monetised via niche-targeted paid ads (Facebook/Instagram or Google).
- Target: 10–50 paying customers, $200–$500/mo per site. This is a portfolio play — modest per-site MRR is by design.
- Competition is expected and normal. Free tools always have weaknesses (ads, bad UX, poor SEO). Only flag competition as a kill signal if it's a free, ad-free, deeply-integrated Google/Microsoft/Meta tool with no UX gap.
- Smaller niche = lower CPCs = better ROI for paid ads. Niche specificity is a positive, not a negative.

Australian market focus (apply throughout your research):
- Research the Australian market first. Look for AU-specific competitors (search "[keywords] australia" and site:.com.au). Absence of AU competitors is an opportunity.
- Flag if this is a lagging market — a model proven in the US/EU that hasn't reached AU yet.
- Note any AU compliance angles (GST, BAS, ABN, Fair Work Act, superannuation) that would create a natural moat.
- AU SMB owners are geo-targetable on Facebook/Google — assess AU reachability specifically.

Idea: [one-sentence idea brief]
Output file: /home/thibanir/Project/portfolio/idea-validator/sessions/generated/[YYYY-MM-DD]/[slug].md

1. Read your persona: /home/thibanir/Project/portfolio/idea-validator/personas/pre-validator.md
2. Read the founder constraints: /home/thibanir/Project/portfolio/idea-validator/profiles/founder.md
3. Do real research using WebSearch and WebFetch — do not make things up.
4. Write your findings to the output file above.
```

#### Step 4 — Synthesize and rank

Read all output files. Rank survivors by:
1. **AU market viability** — is there a real gap in Australia? Lagging market (proven US/EU tool absent in AU) or AU compliance moat = highest signal (highest weight)
2. GTM viability without social presence — specifically, reachability via geo-targeted paid ads in AU
3. AI commoditization risk (low = good) — only penalise if functionality is already shipped, not just hypothetically possible
4. Bootstrapper viability: low maintenance burden + passive revenue model (target $200–$500/mo per site; portfolio model means even modest per-site MRR compounds)
5. Build speed (under $300 to launch)

Write `sessions/generated/[YYYY-MM-DD]/summary.md` (or the focused-mode path). The summary header should reflect the mode:
- Default: `# Summary — [YYYY-MM-DD]`
- Focused: `# Summary — [YYYY-MM-DD] · Focus: [focus area as typed]`

#### Step 5 — Update ideas-index.md and write elevator pitches

For each candidate (not just top 3), add an entry to `ideas-index.md` with status `generated` or `weak` or `killed`.

For each top-3 idea, write `sessions/generated/[session-dir]/[slug]-idea.md` as an elevator pitch ready for `/validate-idea`:

```markdown
# [Product Name]

## The problem
[2–3 sentences: who has the pain, why it's real, what they do today]

## The solution
[2–3 sentences: what the product does, who it's for, how it's delivered]

## Business model
[Pricing model, target price point, who pays]

## Why now / why AI
[What makes this viable now that wasn't viable 2 years ago]

## GTM hypothesis
[Primary acquisition channel and first 10 customers approach — no social media required]
```

**Present results as idea cards, not a table.** For each ranked candidate, print a card using this format:

```
## [Rank]. [Product Name] (`[slug]`) — GTM [X]/10 · Bootstrap [X]/10

**Who it's for:** [Plain-English description of the target customer — no jargon]
**The pain:** [What they struggle with today, in one sentence a non-expert would understand]
**What it does:** [What the product does, in plain English — avoid acronyms]
**AU angle:** [Why this works in Australia specifically — compliance moat, lagging market, or local niche]
**Best channel:** [Primary acquisition channel + why it's reachable without social presence]
**MRR range:** $X–$Y/mo · **Hardest risk:** [Single biggest thing that could kill it]
```

After the cards, print a compact **Weak / Kill** section as a single table (slug + one-sentence reason), then offer: "Pass any slug to `/validate-idea` for the full 6-persona deep-dive."

---

## ideas-index.md — master memory

Every idea that passes through this workspace — generated, validated, rejected, or pivoted — gets an entry. This prevents regenerating the same ideas and surfaces pivot opportunities.

**Format for each entry:**

```markdown
## [slug] — [idea name]
- **Status:** generated | validated | build | pass | killed | pivoted-to:[new-slug]
- **Date:** YYYY-MM-DD
- **Niche:** [e.g. pet groomers]
- **Pain:** [e.g. invoicing]
- **One-liner:** [product concept]
- **Scores:** Market /10 · GTM /10 · Bootstrap /10 · AI-risk /10 (omit if not validated)
- **Verdict:** [build / explore / pass / weak / killed] — [one sentence reason]
- **Session:** [sessions/[slug]/ or sessions/generated/[date]/[slug].md]
```

**Maintenance rules:**
- Generator adds entries at Step 5 with status `generated`, `weak`, or `killed`
- Validator updates entry after investigation with full scores and final verdict
- After debate/pivot, update status to `pivoted-to:[new-slug]` and create new entry for the pivot
- `/pass-idea` sets status to `pass`; `/invest-idea` sets status to `build`
- Never delete entries — stale ideas are still useful to avoid repeating mistakes

---

## Mode 3 — List ideas (`/list-ideas`)

Read `ideas-index.md` and display every tracked idea sorted by Overall score descending. Ideas with no Overall score go to the bottom, sorted by status.

### Display format

Print a ranked table then a legend:

```
# Idea Shortlist — ranked by overall score

Rank  Score  Status      Slug                        One-liner
────  ─────  ──────────  ──────────────────────────  ─────────────────────────────────────────
 1     8/10  generated   arborist-report-tool        ISA TRAQ inspection → PDF for arborists
 2     7/10  generated   catering-event-ops          Post-contract ops for small caterers
 ...

── Unscored / pre-validation ──
  —    weak  florist-order-manager               ...
  —   killed shopify-return-automation           ...

Statuses: generated=ready to validate · explored=validated · build=committed · pass=skipped · weak=low potential · killed=dead
```

Rules:
- Sort first by Overall score descending; ties broken by Bootstrap score descending
- `pass` and `killed` ideas go after all scored ideas (they're decided)
- Show the Verdict one-liner (truncated to ~60 chars if needed)
- If the index has no scored ideas yet, say so and suggest running `/generate-idea`

---

## Mode 4 — Pass an idea (`/pass-idea <slug>`)

Mark an idea as deliberately skipped so it doesn't resurface in generation or lists.

### Steps

1. Read `ideas-index.md`.
2. Find the table row where the Slug column matches `<slug>`. If not found, tell the user and list the closest slug names.
3. Update that row:
   - **Status** column → `pass`
   - **Verdict** column → prepend `Passed — ` before the existing verdict text (keep the rest)
4. Write the updated `ideas-index.md`.
5. Confirm: `✓ [slug] marked as pass. It will be excluded from future generation rounds.`

---

## Mode 5 — Invest in an idea (`/invest-idea <slug>`)

Commit to building an idea: mark it `build` in the index, generate the build brief if needed, and surface the bootstrap command.

### Steps

1. Read `ideas-index.md`.
2. Find the table row matching `<slug>`. If not found, tell the user and list closest slugs.
3. Check whether a full validation session exists at `sessions/[slug]/summary.md`. If it doesn't, warn: `⚠️ No full validation found for [slug] — consider running /validate-idea [slug] first.` Then ask if they want to proceed anyway.
4. Update that row:
   - **Status** column → `build`
   - **Verdict** column → prepend `Building — ` before the existing verdict text
5. Write the updated `ideas-index.md`.
6. **Generate the build brief if missing:** Check whether `sessions/[slug]/build-brief.md` exists. If not, run the `/brief-idea` flow now (Mode 6) to generate it — do not skip this step. The build brief is what gets passed to the pipeline, not the bare elevator pitch.
7. Print the exact bootstrap command to copy-paste, using `sessions/[slug]/build-brief.md` as the idea file:

```
Ready to launch. Run this from the portfolio root:

  ./scripts/new-product.sh [slug] sessions/[slug]/build-brief.md

Make sure ANTHROPIC_API_KEY and ORG_PAT are exported first.
```

---

## Mode 6 — Brief an idea (`/brief-idea <slug>`)

Compile all session research into a single `sessions/[slug]/build-brief.md` — the rich handoff document used as input to the build pipeline. This replaces the bare elevator pitch so the PM agent has full context when writing `SPEC.md`.

### Steps

1. Read `ideas-index.md` to confirm the slug exists. If not found, tell the user and list the closest slugs.
2. Read all available session files — skip gracefully if any are missing, but note what's absent:
   - `sessions/[slug]/idea.md`
   - `sessions/[slug]/summary.md`
   - `sessions/[slug]/market-analyst.md`
   - `sessions/[slug]/bootstrapper.md`
   - `sessions/[slug]/technical-expert.md`
   - `sessions/[slug]/gtm-marketing.md`
   - `sessions/[slug]/brand-domain.md`
   - `sessions/[slug]/devil-advocate.md`
   - `sessions/[slug]/debate-notes.md` (optional — include if present)
3. Synthesize into `sessions/[slug]/build-brief.md` using the format below.
4. Confirm: `✓ Build brief written to sessions/[slug]/build-brief.md`

If persona files are missing (idea was generated but not fully validated), note the gaps in the brief and recommend running `/validate-idea [slug]` before investing.

### Build brief format

```markdown
# [Product Name] — Build Brief
*Generated: [YYYY-MM-DD]*
*Slug: [slug]*

## The product
[Refined elevator pitch — problem, solution, who it's for. Incorporate any pivots from debate-notes.md.]

## Target customer
[Specific niche + pain. AU market context: estimated number of AU businesses in this niche.]

## Business model
[Pricing model, price point, billing cadence, Stripe integration approach]

## Market context
- **AU landscape:** [Are there AU-specific competitors? Lagging market opportunity? Compliance moat?]
- **AU market size:** [Estimated number of reachable AU buyers]
- **Top competitors:** [2–3 names, their pricing, and their real weaknesses — the gaps we exploit]
- **Trend:** [One sentence on market direction]

## Technical scope

### Must-have (MVP)
[Tight feature list — informed by bootstrapper's MRR ceiling and devil's advocate scope warnings]

### Explicitly out of scope (v1)
[What NOT to build — prevents scope creep in Phase 4]

### Key integrations
[From technical-expert — each as ✅ Available / ⚠️ Restricted / ❌ Unavailable, with one-line note]

### Hardest technical dependency
[Single biggest risk. Note any workarounds discussed in the session.]

## GTM playbook
- **Primary channel:** [channel + why it works without social presence]
- **AU targeting:** [Geo-targeting approach — Facebook audience definition, Google keywords, or cold email list source]
- **CAC estimate:** [$X per customer via primary channel]
- **First 10 customers:** [Specific approach — not "find customers online"]
- **Messaging angle:** [Core value proposition in one sentence, AU-specific if relevant]

## Brand
- **Chosen name:** [from brand-domain]
- **Domain:** [status — available / registered / best alternative]
- **Tagline candidate:** [one option]

## Red flags to mitigate
[Top 3 from devil's advocate — each with a specific mitigation to bake into the build or GTM]

## Debate outcomes
[Key decisions, pivots, or refinements from the session. Omit this section if no debate-notes.md exists.]

## Open questions for Phase 1
[Unresolved questions the PM agent should address in SPEC.md before design and build begin]
```
