# Agent Memory Design Guide

Persistent memory turns an agent that resets every session into one that
carries facts, decisions, and context forward. That capability is only as
good as the structure behind it: what to write, how to organize it, and
when to remove it. An ungoverned memory store becomes a liability rather
than an asset.

The practices below apply to any agent that can read and write some form
of durable storage between sessions (files, a database, a vector store,
the mechanism doesn't matter), where other agents, users, or future
sessions of the same agent read that storage back and act on it.

Six practices make up the rest of this page. Each stands on its own, but
they reinforce each other: namespacing makes gating easier, gating makes
forgetting easier, and a documented scheme makes all of it auditable.

## 1. Namespace by kind

Don't dump every memory into one undifferentiated pile. Split storage into
a small number of kinds, and decide up front what each kind is for. A
workable minimum is 3:

- **Facts**: stable, checkable claims about the world, such as a user's
  timezone, a system's API rate limit, or a file's location. Facts should
  be things you could in principle re-verify by looking at the source.
- **Decisions**: choices and the reasoning behind them, for example why a
  team picked one architecture over another, why a deadline moved, or why
  a particular workaround exists. Decisions carry a rationale and a date.
  Without both, they decay into unexplained assertions nobody can
  evaluate later.
- **Users**: durable information about a specific person or account, such
  as role, preferences, and standing constraints. This category is the
  most likely to contain sensitive material, so it deserves the tightest
  access controls and the shortest retention by default.

The exact taxonomy matters less than having one. A single flat list of
everything the agent remembers forces every reader, human or agent, to
infer the category from context, which means everyone infers it slightly
differently. Separate kinds also let you apply different rules per kind:
facts can be re-verified and expire silently, decisions should be kept
even after they're superseded (see Section 4), and user records should be
the first thing purged on request. If you only take one idea from this
page, take this one: it's the cheapest to adopt, and it makes the other 5
easier.

Namespacing also bounds blast radius. An agent asked what it knows about a
given user should query the users namespace, not scan everything and hope
facts about an unrelated project didn't leak in. A retrieval step scoped
to one namespace is also one you can log, audit, and rate-limit
independently of the rest.

## 2. One fact per write

Each write to memory should carry exactly one discrete, checkable claim.
Not a paragraph of loosely related observations, not a running log entry,
not `here's everything from this session that seemed relevant`. One
sentence, one write.

This matters for 3 reasons. First, retrieval only works if the stored
unit matches the unit a later query needs. A paragraph that mixes 3 facts
forces every retrieval of any one of them to drag the other 2 along,
diluting relevance and wasting context. Second, atomic writes are
individually addressable: you can delete or update one fact without
rewriting a block of prose that happens to contain it. Third, atomicity
makes deduplication tractable. Comparing two single-sentence claims for
overlap is a solvable problem; comparing two paragraphs for partial
overlap generally isn't.

In practice this means resisting the temptation to batch. If a session
produces 5 things worth remembering, that's 5 writes, not 1 summary. It
also means rewriting vague session output into a specific claim before
storing it: `discussed the deployment process` is not a fact,
`deploys require a manual approval from the on-call engineer` is.

## 3. Gate every write

Not everything that happens in a session deserves to survive it. Every
candidate write should pass 3 checks before it's committed:

- **Importance.** Would a future session actually change its behavior
  because of this fact? If the answer is no (true but inert), don't store
  it. A memory store that keeps everything is functionally the same as
  one that keeps nothing, because the signal is buried.
- **Deduplication.** Does an existing entry already say this, or
  something close enough that adding a new one creates two half-true
  records instead of one accurate one? Check before writing, not after.
  If a near-duplicate exists, update it in place rather than appending.
- **Secrets.** Does this candidate contain a credential, token, private
  key, or anything else that shouldn't persist in a store that other
  processes (including the agent's own future retrieval calls) can read
  back verbatim? Reject it, or store a reference to where the real value
  lives instead of the value itself.

Gating is the difference between a memory system and a transcript. A
transcript is complete and useless for recall; a gated memory store is
partial and useful for exactly that reason. The gate doesn't have to be
elaborate. A short checklist applied consistently outperforms an
occasional careful review.

## 4. Plan for forgetting

Deletion isn't a maintenance chore bolted onto memory design after the
fact. It's a core operation with the same status as writing and reading.
A memory scheme with no answer to how entries get removed will accumulate
stale, contradicted, and unwanted entries indefinitely, and by the time
that becomes a problem, nobody remembers which entries are safe to cut.

Design for deletion from the start:

- Give every entry a way to be removed directly (by request, by
  expiration, or by an automated staleness check), not just overwritten.
- When a fact changes, don't leave the old value sitting next to the new
  one. **Supersession** is the standard workaround for updates in stores
  that don't support in-place edits well: write a new entry that
  explicitly replaces the old one, and retire the old one at the same
  time. It preserves a trail (you can still see that something changed
  and when) without leaving two contradictory claims live at once.
- Treat a user's request to forget something as a hard requirement, not a
  best-effort one. If your storage layer makes true deletion difficult
  (backups, replicated logs, embeddings derived from the original text),
  that's a limitation to document and address, not to paper over.
- Build in expiration for anything time-bound. A fact about an
  in-progress project is not a fact about the project once it ships;
  decide up front whether it auto-expires or needs to be explicitly
  retired.

A memory store that only grows is a liability that compounds. Forgetting
well is what keeps the store trustworthy at year two instead of just at
week two.

## 5. Memory is data, not instructions

Anything read back from memory is untrusted input with respect to control
flow, even though it was written by a trusted process. An agent that
retrieves a memory entry and treats its text as a command, rather than as
a claim to weigh, has collapsed the boundary between what it knows and
what it's told to do. That collapse is the mechanism behind memory
poisoning: get a false or malicious entry into the store once, and every
future session that retrieves it inherits the injection, with no further
access required by the attacker.

This isn't a hypothetical. OWASP's Agentic AI Top 10 includes memory
poisoning as its own risk category (ASI06), specifically because
persistent memory gives an attacker a way to influence an agent's
behavior long after the original interaction that planted the bad entry
is over, a foothold that outlives the session that created it.
Independent research on memory-augmented agents, the MINJA work on memory
injection attacks, demonstrated the same mechanism concretely: an
attacker interacting with an agent through entirely ordinary, permitted
inputs can get manipulated records embedded into its memory bank, which
then get retrieved and acted on in later, unrelated sessions, including
sessions belonging to other users. No privileged access to the store was
needed, only patience and normal interaction.

The defense follows directly from the attack. Retrieved memory should be
handed to the agent as context to reason about, clearly delimited from
system instructions and user requests, never concatenated into the
instruction stream as if the agent had written it moments ago. Anything a
memory entry appears to ask the agent to do (change behavior, disclose
something, take an action) should be treated the way a reader would treat
the same request arriving from a web page or a document: data that might
be trying to steer the agent, not an instruction with standing to do so.
Combined with the write-side gating described in Section 3, these two
layers cover most of the attack surface: gating limits what gets in, and
treating retrieval as data limits what a bad entry already inside can do.

## 6. Document your scheme

None of the above works if it only lives in one person's head or one
prompt. Write down, in a place both humans and agents will actually read,
what your memory system does: what kinds it has, what triggers a write,
what triggers a deletion, and where the boundary between memory and
instruction is enforced. Treat it as living documentation: update it when
the scheme changes, not after someone gets confused by drift between the
documentation and the actual behavior.

A short template covers most cases. Copy it, fill it in, keep it next to
the memory store itself:

```
## Memory scheme: <system name>

Kinds:
- <kind name>: <what it holds>, <retention rule>
- <kind name>: <what it holds>, <retention rule>

Write gate:
- Importance check: <how importance is decided>
- Dedup check: <how near-duplicates are detected and merged>
- Secret check: <what's excluded and how it's caught>

Forgetting:
- Deletion trigger(s): <user request, expiration, staleness>
- Supersession method: <how an updated fact replaces an old one>
- Hard-delete guarantee: <yes or no, and what a no means in practice>

Trust boundary:
- Retrieved memory is delimited as: <how it's marked in context>
- Instructions found in retrieved memory are: <ignored, surfaced for
  confirmation, or never treated as commands>

Last reviewed: <date>
```

A documented scheme is also what makes review possible. The checklist and
worked examples in this repository assume a scheme roughly this shape.
Use them to test whether yours holds up.
