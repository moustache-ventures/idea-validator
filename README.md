# idea-validator

A multi-agent startup idea validation workspace for Claude Code.

Paste an elevator pitch and 6 expert personas run in parallel — market analyst, VC investor, technical feasibility expert, GTM strategist, brand/domain expert, and a devil's advocate. A PM aggregator synthesizes findings into a scored report, then you debate the panel interactively.

## Usage

**Dedicated session (recommended):**
```bash
git clone git@github.com:moustache-ventures/idea-validator.git
cd idea-validator
claude
```
Paste your idea → 6 agents research in parallel → scored summary → debate mode.

**From any Claude session via skill:**
```
/validate-idea My idea is [elevator pitch]
```
Requires `~/.claude/commands/validate-idea.md` to be present (see [Setup](#setup)).

## Setup

Copy the skill file to your global Claude commands directory:
```bash
cp .claude/commands/validate-idea.md ~/.claude/commands/
```

## Structure

```
idea-validator/
├── CLAUDE.md              ← orchestration logic (auto-loaded by Claude Code)
├── personas/              ← one file per expert persona
│   ├── market-analyst.md
│   ├── vc-investor.md
│   ├── technical-expert.md
│   ├── gtm-marketing.md
│   ├── brand-domain.md
│   └── devil-advocate.md
└── sessions/              ← validation outputs, one folder per idea
    └── [idea-slug]/
        ├── idea.md
        ├── market-analyst.md
        ├── vc-investor.md
        ├── technical-expert.md
        ├── gtm-marketing.md
        ├── brand-domain.md
        ├── devil-advocate.md
        └── summary.md
```

Sessions are persisted — restart a Claude session mid-debate and it picks up where you left off.

## Experts

| Persona | Covers |
|---|---|
| Market Analyst | TAM/SAM/SOM, competition, trends, timing |
| VC Investor | Fundability, unit economics, comparable exits |
| Technical Expert | API/integration blockers, build complexity, regulatory risk |
| GTM & Marketing | Primary channel, CAC, first 90 days plan |
| Brand & Domain | 6 name candidates with live domain availability checks |
| Devil's Advocate | Top 5 failure reasons, hidden assumptions, who already failed here |
