# Plumbline

**A preventive discipline for AI-assisted software development.**

> Reactive verification does not scale. Uncertainty is observable before it becomes a defect.

---

## What this is

Plumbline is a working system, written down. It comes from one founder's daily
practice building a production SaaS with AI in the loop — not from a research lab,
not from a framework vendor, and not (yet) from anyone who didn't write it.

That last part is the point, not a disclaimer. Most methodologies you read are
opinions with no paper trail. This one is the opposite: honest primary-source
material from real cycles on real software, published so you can try it, adapt it,
and tell me where it breaks.

## The thesis

Most quality work in AI-assisted development happens *after* the model produces
something — you review, you test, you debug. That loop doesn't scale as you
delegate more of the work. The more you hand off, the more the after-the-fact
checking becomes the bottleneck.

Plumbline moves the discipline **upstream**. The uncertainty that turns into a
defect later is usually visible earlier — in a vague brief, an unstated assumption,
a quietly missing case. Catch it at the brief stage, before code exists, and you
spend minutes instead of debugging hours.

## The three checks

- **Brief Linting** — a pre-flight pass on the brief *itself*, before the model
  acts on it. Is it complete? Unambiguous? Scoped? A bad brief is a bug you haven't
  written yet.
- **Assumption Surfacing** — an explicit step that forces the hidden assumptions in
  a brief or plan into the open, so you confirm or kill them *before* implementation
  rather than discovering them in review.
- **Omission Catalog** — a retrospective, append-only registry of what got missed.
  Not a pre-flight check: a memory. Recurring blind spots become visible over time
  and feed back into better briefs.

## How it's organized

- **`templates/`** — the living artifacts you keep as you work (parking lot,
  decisions log, logbook, catalog). Append-only. Shipped here empty.
- **`prompts/`** — setup prompts for two distinct contexts: the *application* chat
  (where the work happens) and the *registry* chat (where the method itself evolves).
  Keeping them separate is deliberate.
- **`kpis/`** — the spec for each of the three checks.
- **`MANIFESTO.md`** — why this exists, at length.

## Try it

Run it on the next real brief you write — not a toy task, a real one. Then open a
**[field report](../../issues/new?template=field-report.md)** and tell me what
happened: what surfaced, what was pure overhead, what you changed by instinct.

Field reports are the most valuable thing you can send. They are how this stops
being one person's practice and starts being something tested.

## What this is *not*

- **Not tied to a stack.** No language, framework, or tool is assumed. If a part
  reads as stack-specific, that's a bug — tell me.
- **Not proven at scale.** The evidence base is single-author, single-project.
  Treat it accordingly.
- **Not a replacement for testing.** It moves discipline upstream; it doesn't
  remove anything downstream.

## Status

Early. Single-author. Evolving in the open. Expect rough edges and breaking changes
to the artifacts as field reports come in.

## License

Documentation and methodology: **[CC BY 4.0](LICENSE)** — use it, adapt it, build
on it; just keep the attribution. (A `plumb` CLI, if and when it ships, will carry
its own permissive code license.)
