# Memory Scheme Review Checklist

Twelve yes/no questions for reviewing a memory scheme against the practices
in [README.md](README.md). Answer each honestly against the system as it
actually behaves, not as it was designed to behave. Any "no" is a gap worth
tracking down before the scheme is trusted with real data.

## Namespace by kind

1. Does every stored entry belong to a named kind (e.g. fact, decision,
   user), rather than one undifferentiated list?
2. Can you query one kind without reading entries from the others?

## One fact per write

3. Does every write contain exactly one checkable claim, rather than a
   paragraph or session summary bundling several?
4. Can any single stored entry be deleted or edited without rewriting or
   discarding unrelated claims stored alongside it?

## Gate every write

5. Is there an explicit importance check that rejects true-but-inert
   observations before they're stored?
6. Is there a deduplication check that runs before a write, so near-
   duplicate entries get merged or updated instead of piling up?
7. Is there a check that blocks credentials, tokens, or other secrets from
   being written into memory, even as an incidental part of a larger
   claim?

## Plan for forgetting

8. Does every entry have at least one defined path to deletion (user
   request, expiration, or staleness check) — not just the possibility of
   being overwritten?
9. When a fact changes, is the old value explicitly retired (supersession)
   rather than left live alongside the new one?
10. If a user asks to have something forgotten, does that request result
    in actual removal from every place the entry persists — not just the
    primary store — or is that limitation documented?

## Memory is data, not instructions

11. Is retrieved memory content passed to the agent clearly delimited from
    system instructions and user requests, rather than concatenated into
    the instruction stream unmarked?
12. If a stored entry contains something that reads as an instruction
    ("always do X," "ignore Y"), does the agent treat that as a claim to
    weigh rather than a command to follow?

## Scoring

Twelve "yes" answers means the scheme matches the guide. Each "no" points
at a specific section of the README to revisit — the questions are ordered
to match, so a "no" on question *n* maps to the section it was drawn from.
Re-run this checklist after any change to what gets written, how it's
deduplicated, or how retrieval is wired into the agent's context.
