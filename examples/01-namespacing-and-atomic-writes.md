# Example 1: Namespacing and atomic writes

Briarwood Goods runs a support agent for its online housewares store. A
customer, Dana Okafor, opens a chat about a delayed order.

## The session

> **Dana:** Hi, my order #48213 was supposed to arrive Tuesday, it's now
> Friday. I'm in the Pacific timezone if that affects anything, and I'd
> like it upgraded to expedited shipping at no extra cost because it's
> late.
>
> **Agent:** Sorry about the delay. I can see order #48213 is still in
> transit. I've upgraded it to expedited shipping at no charge; it should
> arrive within 2 business days.

## What not to store

A single paragraph dumped into one memory entry:

```
Dana Okafor contacted support about order #48213 being late, she's in
Pacific time, agent upgraded her to free expedited shipping, seemed a
bit frustrated but was polite about it.
```

This is a session summary, not a memory write. It mixes a durable user
fact (timezone), a case-specific decision (the shipping upgrade and why),
and an editorial aside (seemed a bit frustrated) that isn't a checkable
claim at all. Stored this way, none of it can be retrieved, updated, or
deleted independently later.

## What to store instead

3 atomic writes, each to the namespace it belongs in:

The **users** namespace gets Dana's timezone:

```text
Dana Okafor (account #48213-holder) is in the Pacific timezone.
```

The **decisions** namespace gets the shipping upgrade and why:

```text
Order #48213 was upgraded to expedited shipping at no charge on
2026-08-28, because the original delivery estimate was missed by 3 days.
```

The **facts** namespace gets the general policy, only if it reveals
something beyond this one case:

```text
Orders that miss their delivery estimate by more than 48 hours are
eligible for a free expedited-shipping upgrade per current support
policy.
```

The editorial aside about Dana's mood is dropped entirely. It isn't a
checkable claim, and the importance check from Section 3 of the guide
should filter it before it ever reaches a write.

## Why this matters later

A future session that pulls up Dana's account queries the users namespace
and gets her timezone without wading through case history. A
support-policy audit queries decisions and facts without touching any
individual customer's personal data. Because each write is atomic, if
Dana updates her timezone next year, that one users entry is edited in
place; the shipping decision and the policy fact are untouched.
