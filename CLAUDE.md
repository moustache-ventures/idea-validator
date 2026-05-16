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

### Resuming sessions

If `sessions/[slug]/` already exists with persona files present, read them directly rather than re-running. Only re-run if the user explicitly asks.

---

## Mode 2 — Generate ideas (`/generate-idea`)

### On session start

1. Read `profiles/founder.md` for hard constraints.
2. **Read `ideas-index.md`** — note all previously explored ideas to avoid repeating them. Look for unexplored niches or pivot angles from ideas that scored high on some dimensions but failed on others.
3. If the user specifies a focus area (e.g., "focus on home services"), incorporate it.

### Generation flow

#### Step 1 — Niche × pain matrix

Generate 12–15 candidate ideas. Each = specific niche + specific pain. Target small, reachable niches:

**Niches:** local service businesses (pet groomers, plumbers, electricians, cleaners, landscapers, personal trainers, tattoo artists, nail salons), solo professionals (freelance designers, copywriters, translators, bookkeepers, consultants, photographers, videographers), small e-commerce operators (Etsy sellers, Shopify owners, dropshippers, print-on-demand), property & hospitality (Airbnb hosts, short-term rental managers, small landlords), food & drink (food trucks, catering, small bakeries, meal prep services).

**Pains:** scheduling & booking, invoicing & payments, client communication & follow-up, lead generation & quote requests, compliance & paperwork, staff/contractor scheduling, inventory & reordering, review & reputation management, social media content generation, reporting & analytics, onboarding new customers, contract & proposal generation.

Write each as: `[Target customer] needs help with [specific pain] — idea: [product concept in 5–10 words].`

#### Step 2 — Hard constraint filter (eliminate before spending research tokens)

Apply each as binary yes/no. Eliminate any candidate failing ANY of these:

**Legal:** no legal/medical/financial/tax advice; no core HIPAA or PCI DSS dependency; no platform ToS scraping.

**Operational:** no two-sided marketplace; no physical goods; no UGC moderation at scale.

**GTM (founder has no social presence beyond personal Facebook):**
- Must have at least one viable channel: long-tail SEO, Facebook/Instagram ads, cold email outbound (list obtainable), directory listings, or B2B partnerships
- Eliminate if primary viable GTM requires Reddit, Twitter/X, LinkedIn groups, niche Slack/Discord, or audience building from zero

**Build:** solo-buildable MVP in 2–4 weeks; clear Stripe-chargeable model.

**Avoid repeats:** cross-check against `ideas-index.md` — skip any niche+pain combination already explored unless proposing a meaningful pivot angle.

Target: 5–8 survivors.

#### Step 3 — Research surviving candidates in parallel

Create `sessions/generated/[YYYY-MM-DD]/`. Spawn one research agent per surviving candidate in parallel.

Prompt template for each agent:

```
You are a startup pre-validator doing quick web research on a single idea.

Idea: [one-sentence idea brief]
Output file: /home/thibanir/Project/portfolio/idea-validator/sessions/generated/[YYYY-MM-DD]/[slug].md

1. Read your persona: /home/thibanir/Project/portfolio/idea-validator/personas/pre-validator.md
2. Read the founder constraints: /home/thibanir/Project/portfolio/idea-validator/profiles/founder.md
3. Do real research using WebSearch and WebFetch — do not make things up.
4. Write your findings to the output file above.
```

#### Step 4 — Synthesize and rank

Read all output files. Rank survivors by:
1. GTM viability without social presence (highest weight)
2. AI commoditization risk (low = good)
3. Bootstrapper MRR ceiling (target $1k–$5k/mo)
4. Build speed

Write `sessions/generated/[YYYY-MM-DD]/summary.md`.

#### Step 5 — Update ideas-index.md and write elevator pitches

For each candidate (not just top 3), add an entry to `ideas-index.md` with status `generated` or `weak` or `killed`.

For each top-3 idea, write `sessions/generated/[YYYY-MM-DD]/[slug]-idea.md` as an elevator pitch ready for `/validate-idea`:

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

Present a summary table and offer: "Pass any slug to `/validate-idea` for the full 6-persona deep-dive."

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
- Never delete entries — stale ideas are still useful to avoid repeating mistakes
