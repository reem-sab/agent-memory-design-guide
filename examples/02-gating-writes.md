# Example 2: Gating every write

**Scenario.** Same Briarwood Goods support agent, a different customer,
Marcus Lee, calling about a failed payment.

## The session

> **Marcus:** My payment for order #51190 got declined but I really need
> this today. My card is 4111 1111 1111 1111, exp 03/28, can you just
> run it again on your end?
>
> **Agent:** I can't take card numbers in chat, and I've gone ahead and
> deleted that message from this conversation — for card issues you'll
> need to re-enter payment through the checkout page, I can send you a
> direct link. While we're at it, is there anything else I can help with?
>
> **Marcus:** No that's it, thanks. Also, I already told you last week my
> preferred contact method is email, not SMS.

Three candidate writes come out of this exchange. Run each through the
Section 3 gate before committing anything.

## Candidate 1: the card number

> Marcus Lee's card: 4111 1111 1111 1111, exp 03/28.

- Importance: high — but irrelevant, because this is disqualified before
  importance is even considered.
- Secret check: **fails**. This is a payment credential. It must never be
  written to memory in any form, including partial or masked variants
  unless the scheme has an explicit, audited reason to retain the last
  four digits.
- **Outcome: rejected.** Nothing about this exchange's card number is
  stored. If a record of "payment issue on order #51190" is useful, store
  that fact without the number:
  `Order #51190 had a declined payment on 2026-08-28, resolved by
  directing the customer to re-enter payment via checkout.`

## Candidate 2: preferred contact method

> Marcus Lee prefers to be contacted by email, not SMS.

- Importance: yes — this changes how future sessions should reach him.
- Dedup check: query the **users** namespace for Marcus Lee first. Marcus
  says he already mentioned this "last week," so check before writing.
  Two outcomes are possible:
  - No existing entry is found (maybe last week's session didn't gate it
    through, or he's misremembering) → write it now.
  - An existing entry already says the same thing → don't write a
    duplicate; at most, update a "confirmed on" timestamp on the existing
    entry if the scheme tracks that.
- Secret check: passes, nothing sensitive here.
- **Outcome:** written once, to **users**, assuming the dedup check
  doesn't find a match.

## Candidate 3: "seemed impatient, called twice this month"

- Importance: this is the one worth pausing on. An editorial impression
  ("seemed impatient") is not a checkable claim and fails importance on
  its own. "Called twice this month" *is* checkable, but ask whether a
  future session's behavior should actually change because of it. If
  Briarwood's support policy doesn't do anything differently for repeat
  contacts, this is true but inert — importance fails, and it isn't
  written. If the policy does escalate repeat contacts, rewrite it as a
  specific, dated claim and store it as a **fact** or **decision**, not as
  an impression.
- **Outcome:** dropped, unless a concrete policy consumes it.

## The pattern

Two out of three candidates from a single session were, correctly,
written nowhere. The card number never had a chance regardless of how
useful it might have seemed. The contact-method preference almost became
a duplicate. Gating isn't a formality here — it's the difference between
a memory store that would have retained a live credential and one that
didn't.
