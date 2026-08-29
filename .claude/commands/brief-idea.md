Compile all session research into a single build-brief.md — the rich handoff document used as input to the build pipeline.

Usage: /brief-idea <slug>

All paths below are relative to the idea-validator repo root (this repo).

The slug to act on is: $ARGUMENTS

Follow Mode 6 in `CLAUDE.md`:

1. Read `ideas-index.md` to confirm the slug exists.
2. Read all available session files under `sessions/$ARGUMENTS/` (idea.md, summary.md, all 6 persona files, debate-notes.md if present). Skip missing files gracefully but note what's absent.
3. Synthesize into `sessions/$ARGUMENTS/build-brief.md` using the build brief format defined in Mode 6.
4. Confirm: `✓ Build brief written to sessions/$ARGUMENTS/build-brief.md`

If persona files are missing (idea was generated but not fully validated), note the gaps in the brief and recommend running /validate-idea $ARGUMENTS before investing.
