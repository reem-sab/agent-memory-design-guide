# Contributing

This repository stays small on purpose. Before proposing an addition,
weigh it against that goal: the guide is meant to be readable in one
sitting, and every new page is something a reader has to decide whether
to skip.

## What's in scope

- Corrections to README.md: a claim that's wrong, unclear, or no longer
  matches current research.
- Updates to [references.md](references.md): a citation that's moved, a
  more current source for an existing claim, or a genuinely load-bearing
  new source (not just related reading).
- New worked examples that illustrate a practice the existing 3 examples
  don't cover well, using the same invented company (Briarwood Goods) or
  a clearly-labeled alternative.
- Fixes to [checklist.md](checklist.md) or [SKILL.md](SKILL.md) where
  they've drifted from the current README.

## What's out of scope

- Vendor names, product endorsements, or framework-specific instructions
  in README.md, checklist.md, or the SKILL.md body. This guide is meant
  to apply regardless of storage backend or agent framework; keep it that
  way.
- A 7th practice. The 6 sections in the README cover write, organize,
  gate, delete, trust, and document. A proposal that doesn't fit one of
  those probably belongs in a fork or a different document, not a new
  section here.
- Tooling, SDKs, or reference implementations. This repository is
  documentation, not code.

## Style

All prose contributions should follow the
[Sui documentation style guide](https://docs.sui.io/references/contribute/style-guide):
second person, active voice, sentence-case headings, no em dashes, no
quotation marks other than the one exception the guide itself carves out,
Oxford commas, and numerals for quantities. Run a quick check before
opening a pull request:

```bash
grep -nP '\x{2014}' *.md examples/*.md
```

If that command returns anything, replace the em dash with a comma,
parentheses, or a second sentence before submitting.

## Opening a pull request

State which section of the README or which file the change affects, and
why. For a factual correction, link a source. For a new example, explain
which practice it demonstrates that the existing examples don't.
