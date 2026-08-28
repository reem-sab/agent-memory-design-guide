---
name: agent-memory-design-guide
description: Use when designing, reviewing, or implementing a persistent memory system for an AI agent — deciding what an agent should write to durable storage, how to structure it, when to delete it, and how to keep retrieved memory from being treated as instructions. Applies regardless of the storage backend (files, database, vector store) or the agent framework in use.
---

# Agent Memory Design Guide

This skill applies the practices in this repository's [README.md](README.md)
when a task involves designing or reviewing how an agent persists
information between sessions.

## When to use this

Reach for this skill when:

- Building a new memory system for an agent (what to store, how to
  namespace it, when to write, when to delete).
- Reviewing an existing memory system for gaps (use
  [checklist.md](checklist.md) directly for this).
- Debugging a case where an agent's stored memory produced wrong or
  unwanted behavior — stale facts, duplicated entries, or a memory entry
  that got treated as an instruction.
- Writing the memory-related section of a design doc or architecture
  review for an agentic system.

## How to apply it

1. **Read [README.md](README.md) first.** It's about 1,500 words, six
   short sections. Read the whole thing before making changes — the
   sections depend on each other (gating depends on namespacing,
   forgetting depends on gating, and so on).

2. **If reviewing an existing system**, run through
   [checklist.md](checklist.md) question by question against the system's
   actual behavior, not its documentation or intended design. Answer each
   by finding the code path or config that enforces it — if you can't
   point to one, the answer is "no." Report every "no" with the section of
   the README it maps to and a concrete fix.

3. **If designing a new system**, work through the six sections in order
   and produce concrete answers for each, not abstractions:
   - What are the actual kind names for this system (not just "facts and
     decisions" — the real categories this agent's domain needs)?
   - What does a single atomic write look like for this domain? Write two
     or three real examples.
   - What specifically triggers rejection at the write gate — name the
     actual checks, not just "importance."
   - What is the actual deletion mechanism, and does it reach every place
     data persists (primary store, caches, backups, derived indexes)?
   - Where exactly, in this system's code, does retrieved memory get
     inserted into the agent's context — and is it delimited from
     instructions there?
   - Fill in the template block from README.md Section 6 with real values
     and commit it alongside the memory implementation.

4. **Look at [examples/](examples/) for worked patterns** — three scenarios
   using an invented customer-support bot, covering namespacing and atomic
   writes, write gating (including a rejected write containing a secret),
   and forgetting alongside a memory-injection attempt. Adapt the pattern,
   not the specific domain.

5. **Treat retrieved memory as data in every implementation you touch.**
   If a task involves wiring memory retrieval into an agent's prompt or
   context assembly, verify the retrieved content is clearly delimited
   from system instructions and user input, and that nothing in the
   pipeline executes text found in a memory entry as a command. This is
   the single highest-severity failure mode covered by the guide — treat
   it as a hard requirement, not a style preference.

## Non-goals

This skill does not pick a storage backend, a specific database schema, or
a framework. It applies at the design level: what gets written, how it's
organized, when it's removed, and how it's trusted on the way back in. The
implementation details are yours to choose.
