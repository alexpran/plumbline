# Plumbline

## The problem

AI-assisted development has a reliability problem. The problem is not the AI.

The problem is that we verify the wrong things, at the wrong time, with the wrong tools. We write a brief, hand it to a model, get back code, and then start checking. Tests run. Reviewers read. UIs are clicked. By the time we notice that the model misunderstood the brief — or that the brief itself was ambiguous — we've already paid the cost of the implementation, and we're paying the cost of the rework.

This is reactive verification. It is the dominant mode of AI-assisted development today, and it does not scale.

It does not scale because the cost of detection grows with the distance between the misunderstanding and the moment we notice it. A misread assumption surfaced before implementation costs one prompt to correct. The same misread assumption surfaced after a feature is shipped costs hours of debugging, rework, and — in multi-tenant or compliance-bound systems — sometimes a data migration.

The industry's response has been to invest more in better models, better prompts, and better post-hoc review tools. These help. They are not enough. As long as verification remains reactive, the bottleneck stays where it has always been: the brief, and the gap between what the author meant and what the model interpreted.

## The shift

Plumbline starts from a different premise: **uncertainty is observable before it becomes a defect.** It can be measured on the brief itself, before the first line of code is generated. It can be surfaced from the model in a controlled, structured way, before any implementation begins. And it can be catalogued across episodes, so that the same omission is never paid for twice.

This is not a workflow. It is not a sequence of phases or a set of agent roles. Plumbline is **preventive**: a small set of measurements you take before the AI starts writing, and a small set of practices for learning from the times you didn't.

A plumbline doesn't build walls. It reveals when they're crooked, before the next course of bricks is laid.

## What Plumbline values

We have learned, by building real systems with AI assistance, to value:

**Preventive measurement over reactive review.**
The cheapest defect to fix is the one that never reaches the codebase. Verification that happens after implementation is necessary, but it is the last line of defense, not the first.

**Structured contracts over generated code.**
Code that is its own specification cannot be checked against a separate intent. Methodologies that scale rest on artifacts — schemas, metadata, specs, declarative configurations — that can be verified mechanically, independent of the code that implements them.

**Surfaced uncertainty over confident output.**
A model that lists its assumptions is more useful than a model that hides them behind fluent prose. The single most reliable predictor of drift is the gap between how confident the output sounds and how grounded the brief actually is.

**Catalogued omissions over isolated post-mortems.**
Every drift episode contains information about a class of brief insufficiency. Logged systematically, these episodes become a preventive checklist. Logged once and forgotten, they become recurring bugs.

**Stable anchors over rigid workflows.**
A small number of non-negotiable checkpoints — what must be verified, what must be declared explicit, what must be catalogued — outperform prescriptive end-to-end processes. Anchors permit adaptation. Workflows resist it.

## What Plumbline is not

Plumbline is not a replacement for testing, code review, or runtime monitoring. It does not generate code. It does not orchestrate agents. It does not prescribe a sequence of phases or a set of named roles.

Plumbline is the layer that comes before all of these. It is the discipline of asking, before the model starts writing: *do I know enough yet, and does the model know that I know enough?*

## What we propose

Three measurements, applicable to any AI-assisted development workflow, regardless of stack, language, or tooling:

1. **Brief Linting** — static signals on the brief itself, before any prompt is sent.
2. **Assumption Surfacing** — a structured pass in which the model declares its assumptions, ambiguities, and open questions, before producing any output.
3. **Omission Catalog** — a living record of brief insufficiencies, each entry derived from a real drift episode, each entry feeding back into the previous two.

The measurements are independent of any specific framework, model, or programming language. They are operational, not theoretical. They are designed to be tried on a single feature this week, calibrated on five features this month, and trusted on every feature next quarter.

## The bet

The bet behind Plumbline is that the next phase of AI-assisted development will not be won by faster code generation. It will be won by builders who learn to measure the quality of their inputs as rigorously as they measure the quality of their outputs.

The plumbline is older than software. The principle has not changed. Only the wall has.

---

*Draft v0.1 — May 2026*
