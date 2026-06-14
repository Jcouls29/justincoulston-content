---
title: Architecture Reviews Should Focus on Risk
summary: Most architecture reviews check boxes. The useful ones rank risks — and spend the meeting on the few decisions that are expensive to reverse.
category: Distributed Systems
publishedDate: 2026-06-08
tags:
  - review
  - risk
  - architecture
relatedArticles:
  - stress-testing-software-architectures
relatedDiagrams:
  - multi-tenant-isolation-models
keyTakeaways:
  - Review the decisions that are hard to reverse, not the ones that are easy to change.
  - Rank risks explicitly; a review without a ranked list is just a status meeting.
  - The isolation model is a one-way door — spend review time there.
---

A lot of architecture reviews are really status meetings in disguise. Everyone
nods at the diagram, someone asks about logging, and the meeting ends without
changing a single decision. A review earns its hour only if it surfaces risk and
forces a choice on the decisions that are expensive to undo.

## One-way doors

Borrowing the framing: some decisions are two-way doors — cheap to walk back — and
some are one-way doors. Reviews should spend almost all of their time on the
one-way doors. Choosing a logging library is reversible. Choosing how you isolate
tenant data usually is not:

:::diagram{id="multi-tenant-isolation-models"}
:::

Moving from a pool model to a silo model after you have a thousand tenants is a
migration project, not a config change. That is exactly the kind of decision a
review should pressure-test — ideally by [stress-testing it on
paper](https://example.com/stress-test) first.

## Rank the risks

I end every review with a ranked list, highest risk first:

```text
1. Tenant isolation model        (one-way door, high blast radius)
2. Sync dependency on acquirer   (availability coupling)
3. Schema migration strategy     (operational risk)
```

If the review didn't change the order of that list — or what we plan to do about
the top item — it didn't do its job.
