# Parking Lot

> **Memory of unfollowed branches.**
> Questions raised during work that cannot be resolved now and should not be lost.

## Purpose

During Plumbline cycles, conversations branch. Some branches lead to immediate
decisions; others surface questions too large, too out-of-scope, or too dependent on
other context to be resolved in the current session. Without a system, these branches
evaporate — recovered (if at all) months later, at higher cost.

The parking lot is the methodology's memory of unfollowed branches. Each entry costs a
minute to record and remains visible until explicitly resolved or abandoned.

## Format

Each entry uses this minimal schema:

```
## PL-NNN — Short title

- Date raised: YYYY-MM-DD
- Source: brief reference to the session or discussion that raised it
- Type: architectural-decision | product-decision | methodology-gap | technical-investigation | other
- Description: 2-4 lines on what the point is, why it was raised, what needs to be understood or decided
- Blocking: what (if anything) is blocked until this is resolved
- Status: open | in-discussion | resolved | abandoned
- Resolution: empty until resolved; when resolved, brief summary and pointer to consequent brief / decision / ADR
```

## Three rules

1. **Raise immediately.** When a point is parked, the entry is created on the spot. No
   "I'll note it later". The five-minute rule of the catalog applies: if it costs more,
   it doesn't happen.
2. **Periodic review.** Every N sessions or defined interval, scan the parking lot to
   check what's still relevant, what's become urgent, what can be abandoned.
3. **`abandoned` is legitimate.** Not every entry needs to resolve. Some lose relevance
   as context shifts. Marking abandoned is a valid outcome, not a failure.

## Scope note

This file can mix two kinds of parked question: **methodology-level** (open questions
about Plumbline itself) and **project-level** (open questions about the project where
Plumbline is being applied). The `Type` field distinguishes them — `methodology-gap` is
methodology-level; the rest are project-level. If the volume grows, split into two files
(one for methodology, one per project repo). See DD-14 in `DECISIONS.md`.

---

## Entries

<!-- Add PL entries here, newest at the bottom. Append-only by convention. -->

---

## Backlog summary (manual until tooling)

- Total entries: 0
- Open: 0
- In discussion: 0
- Resolved: 0
- Abandoned: 0

---

*Living artifact. Review cadence determined empirically.*
