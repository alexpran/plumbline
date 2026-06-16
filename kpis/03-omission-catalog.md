# KPI 03 — Omission Catalog

> **The third measurement, the first to activate.**
> A living record of brief insufficiencies, distilled from real drift episodes.

## Why this exists

Brief insufficiencies don't repeat themselves identically — they repeat themselves *as patterns*. The same author, working on similar systems, omits the same kinds of things across different briefs: state initialization, edge cases on data distribution, boundary of responsibility between human and model, behavior on duplicate input.

These patterns are invisible from any single episode. They become visible only when episodes are aggregated.

Working memory does not aggregate. By the second or third week, the details of an earlier drift have evaporated. What remains is a vague sense that *"something didn't work last time"* — too unstructured to feed back into future briefs.

The Omission Catalog is the external memory that closes this loop. Each entry costs under five minutes to record. The aggregate, after fifteen to twenty entries, reveals the pattern.

## What it measures

The Omission Catalog does not measure individual briefs. It measures **the author's recurring blind spots** across briefs, over time.

After enough entries, the catalog answers questions like:
- Which categories of omission are most frequent in my briefs?
- Which categories cost the most when they slip through?
- Which can be promoted to checks in **Brief Linting**?
- Which can be turned into pre-baked questions in **Assumption Surfacing**?

The catalog is the upstream input to the other two KPIs. Without it, they remain static checklists. With it, they evolve.

## When to record

A new entry is recorded **immediately after a drift is detected and traced**, not at the end of the day, not at the end of the week.

The trigger is operational: every time you correct AI-generated output and the correction reveals a brief insufficiency (not an implementation bug, not a model hallucination on something the brief covered correctly), record an entry. Five minutes, then back to work.

If you find yourself asking *"is this worth recording?"*, the answer is yes. False positives are cheap. False negatives are expensive — they are exactly the entries that would have closed the loop.

## What is NOT recorded

The catalog tracks **brief insufficiencies**, not implementation bugs. The distinction matters:

- ✅ Brief said nothing about behavior when the input list is empty → record
- ✅ Brief assumed state was readable from source X, but state lives in source Y → record
- ✅ Brief did not specify boundary of responsibility for parameter filling → record
- ❌ Model produced syntactically incorrect code → not a brief problem
- ❌ Frontend signal stayed `true` after first dispatch → implementation bug, not catalog material
- ❌ Library version incompatibility → environment issue, not catalog material

The test: *if the brief had been more complete, would this drift have been prevented?* If yes, record. If no (the drift is independent of brief quality), don't.

## Entry schema

Each entry is a YAML object. The schema is intentionally minimal — overhead kills adoption.

```yaml
- id: OC-NNN
  date: YYYY-MM-DD
  feature: "Short identifier of the feature being built"
  drift_type: "One-line description of what went wrong"
  detected_by: "How you noticed: ui-test | structural-check | code-review | runtime"
  brief_omission: "What the brief failed to specify, in one sentence"
  category: "kebab-case-slug"
  preventable_by:
    - kpi: brief-linting | assumption-surfacing
      signal: "What signal would have caught this"
  cost_of_drift: "Rough estimate: minutes lost, rework cycles, user impact"
  notes: "Optional. Anything that helps future-you remember context."
```

### Field guidance

- **`id`** — sequential, zero-padded (`OC-001`, `OC-002`...). No clever schemes.
- **`feature`** — enough to recognize the feature six months later, no more. Avoid full ticket IDs unless they're durable.
- **`drift_type`** — what *actually happened*, factual, no interpretation.
- **`detected_by`** — which layer of verification caught it (sintomatic / structural / semantic, in the language of the methodology). This data lets you measure whether layered drift detection is working.
- **`brief_omission`** — the single sentence that, if added to the brief, would have prevented the drift. This is the hardest field. Take a minute on it. It's the field that makes the entry useful.
- **`category`** — short, kebab-case, reused across entries. Don't invent a new category if an existing one fits. Categories emerge from aggregation, not from upfront design.
- **`preventable_by`** — which other KPI(s) could have caught this, and how. Empty if you genuinely can't see how either KPI would have caught it (this itself is interesting data — those are the entries that demand new KPIs).
- **`cost_of_drift`** — rough is fine. The point is to weight categories later, not to do precision accounting.
- **`notes`** — for context that doesn't fit anywhere else. Often empty.

## Worked example (illustrative) — `suggest_related_tags`

The following entries are a constructed example, not logged episodes. They illustrate how the schema applies to a feature that suggests related tags for an article being drafted in a content editor.

```yaml
- id: OC-001
  date: 2026-01-12
  feature: "Brief A — suggest_related_tags capability (editor assistant)"
  drift_type: "getCurrentTags() read state from the persisted article record (empty pre-save) instead of the in-progress editor draft"
  detected_by: "ui-test"
  brief_omission: "Brief did not specify which source of truth holds the in-progress draft state before persistence"
  category: "state-source-not-specified"
  preventable_by:
    - kpi: brief-linting
      signal: "Stateful feature without explicit declaration of state source(s)"
    - kpi: assumption-surfacing
      signal: "Model would have asked: 'Should I read current tags from the persisted record or the in-flight draft state?'"
  cost_of_drift: "1 debug cycle (~30 min) + 1 brief amendment"
  notes: "Recurring class of bug in stateful features pre-persistence. Worth promoting to a Brief Linting check."

- id: OC-002
  date: 2026-01-13
  feature: "Brief A — suggest_related_tags capability"
  drift_type: "Model proactively filled a tool input parameter (existingTagLabels) that was meant to be injected by the runtime"
  detected_by: "structural-check"
  brief_omission: "Brief did not declare boundary of responsibility for tool parameter population — what the model fills vs. what the runtime injects"
  category: "responsibility-boundary-unspecified"
  preventable_by:
    - kpi: assumption-surfacing
      signal: "Model would have asked: 'Which input parameters am I responsible for, and which are injected by the runtime?'"
  cost_of_drift: "1 debug cycle + minor refactor"
  notes: "Particularly relevant for tool-using agents. May deserve a dedicated section in brief template for tool-based features."

- id: OC-003
  date: 2026-01-14
  feature: "Brief A — suggest_related_tags capability"
  drift_type: "Shared topic taxonomy returned zero candidates for niche subjects, breaking the suggestion flow"
  detected_by: "ui-test"
  brief_omission: "Brief did not consider the data distribution of the source (the taxonomy is sparse for specialized topics)"
  category: "data-source-distribution-unexamined"
  preventable_by:
    - kpi: brief-linting
      signal: "Feature depending on external data source without declared assumptions on coverage / sparseness"
  cost_of_drift: "Architectural pivot — replaced the static taxonomy lookup with a generative fallback (Brief A2 v2)"
  notes: "High-cost drift. Worth a dedicated check: any external dependency must declare assumed coverage characteristics."

- id: OC-004
  date: 2026-01-15
  feature: "Brief A — suggest_related_tags capability"
  drift_type: "Routing confusion between editor-assistant capabilities — the runtime could not disambiguate which capability to invoke"
  detected_by: "ui-test"
  brief_omission: "Brief did not specify the disambiguation contract when a user prompt could plausibly match multiple capabilities"
  category: "multi-handler-disambiguation-missing"
  preventable_by:
    - kpi: assumption-surfacing
      signal: "Model would have asked: 'How should the runtime choose when a user prompt could match more than one capability?'"
  cost_of_drift: "1 debug cycle + minor logic addition"
  notes: "Generalizes beyond this feature: any framework with N handlers needs an explicit disambiguation contract."
```

Note that all four entries trace back to **brief omissions**, not model errors or implementation bugs. A fifth episode in the same iteration (a frontend `loading()` signal staying `true` after first dispatch) is *not* in the catalog — it was an implementation bug independent of brief quality.

## Categories: emergent, not designed

Resist the temptation to define categories upfront. Categories emerge from the catalog itself, after roughly fifteen to twenty entries, when patterns of repetition become visible.

When defining a new category for an entry, prefer specific over general (`state-source-not-specified` over `state-issues`). Specific categories merge later if they prove redundant; general categories never split.

After every ~20 entries, perform a **category review**: list all distinct categories, count occurrences, and identify the top three by frequency *and* the top three by cost. These six (often overlapping) become the next promotion candidates.

## Promotion: from catalog to upstream KPIs

The catalog's purpose is not to exist as an archive. It is to feed the other two KPIs.

A category is **promoted to Brief Linting** when it has appeared three or more times. The promotion produces a new line in the lint checklist:
- `state-source-not-specified` (3 occurrences) → new check: *"For stateful features, does the brief explicitly declare the source of truth for each state component?"*

A category is **promoted to Assumption Surfacing** when its instances share a clear question pattern that the model could have asked but didn't. The promotion produces a new line in the surfacing prompt:
- `responsibility-boundary-unspecified` (multiple occurrences) → new prompt addition: *"For each tool parameter, declare whether it is filled by the model or injected by the framework."*

After promotion, the catalog category does not disappear — it remains a permanent record of where a check came from. New occurrences continue to be logged; if a promoted check fails to prevent recurrence, the check itself is revisited.

## Storage and format

A single file: `plumbline/catalog.yaml` in the project repository.

Append-only by convention. Edits to existing entries are rare and acceptable (typo fixes, retroactive note additions). Deletions are not. If an entry turns out to be irrelevant in retrospect, mark it with `relevance: low` rather than removing it — the fact that you initially recorded it is itself information.

The file is plain YAML, queryable with any tool (`yq`, `grep`, custom scripts). No database, no service, no UI. The simplicity is the design.

## Logbook (companion artifact)

Alongside the catalog, maintain a minimal `plumbline/logbook.md` to record the *experimentation* of Plumbline itself:
- Which features did you apply Plumbline to this week?
- Which KPIs did you actually use, vs. skip?
- What broke, what felt like overhead, what surprised you?

Two lines per cycle is enough. The logbook is to Plumbline what the catalog is to your briefs: external memory of pattern over time.

## Design decisions

KPI-specific design decisions for the Omission Catalog. Cross-cutting methodology decisions (language, file formats, prescriptive vs. preventive stance, append-only convention, emergent categorization, promotion mechanism) are recorded in `decisions.md`.

### DD-OC-01: Brief insufficiencies, not implementation bugs

**Decision:** the catalog records only drifts traceable to brief insufficiency. Implementation bugs, model hallucinations on covered topics, and environment issues are explicitly excluded.

**Rationale:** mixing categories of failure dilutes the catalog's signal. Brief insufficiencies are systematically preventable through better upstream artifacts; implementation bugs are caught by other tools (tests, linters, code review). Conflating them produces a list too noisy to mine for patterns.

**Alternatives considered:**
- *Catalog all failures, tag by type*: rejected. The volume of implementation bugs would overwhelm the signal of brief omissions, and users would stop logging both.
- *Two separate catalogs*: rejected. Implementation bugs already have established tooling (issue trackers, test reports). Plumbline addresses a gap, not a duplicate.

**Reversibility:** medium. A team could choose to use the same schema for a broader bug catalog, but should keep it in a separate file to preserve the signal of the omission catalog.

---

### DD-OC-02: Five-minute ceiling per entry

**Decision:** entry recording is designed to take under five minutes. The schema has nine fields; most are short strings.

**Rationale:** any logging discipline that exceeds five minutes per entry is abandoned within weeks. The catalog must be cheap enough to survive busy days. Information that takes longer to record is not recorded at all, and the catalog becomes worthless.

**Alternatives considered:**
- *Richer schema with structured analysis fields (root cause, blast radius, contributing factors)*: rejected. Closer to incident postmortems than to operational catalogs; appropriate for high-stakes systems but overkill here.
- *Free-form text only*: rejected. Free-form entries cannot be aggregated or queried, defeating the purpose of building a catalog over time.

**Reversibility:** high. Teams can add fields if they observe their own discipline holds up. The default is conservative.

---

### DD-OC-03: `detected_by` doubles as telemetry for layered drift detection

**Decision:** the `detected_by` field uses a fixed vocabulary (`ui-test`, `structural-check`, `code-review`, `runtime`) corresponding to the layers of drift detection in the methodology.

**Rationale:** the field could have been free-form. Constraining it produces a side benefit: aggregated data on which detection layer catches which categories of drift. This data validates (or invalidates) the methodology's claim that layered detection works, and identifies which layer is doing the most work.

**Alternatives considered:**
- *Free-form description of detection*: rejected. Loses the ability to aggregate.
- *Multiple values allowed*: rejected for v0.1. Real episodes are usually caught at a single layer; multi-layer detection can be added later if patterns demand it.

**Reversibility:** high. Vocabulary can be extended; multi-value can be added; the field can be made optional.

---

### DD-OC-04: `cost_of_drift` is a free-form string, not an ordinal scale

**Decision:** the `cost_of_drift` field accepts any string the author finds informative. No required units, no ordinal scale (low/medium/high).

**Rationale:** different episodes have wildly different cost dimensions — debug minutes, rework cycles, user impact, architectural pivots. Forcing them all into one scale loses information. Free-form strings let the author capture the cost dimension that actually matters for that episode. Aggregate analysis comes from reading entries, not from summing numbers.

**Alternatives considered:**
- *Ordinal scale (low/medium/high)*: rejected. Tempting, but compresses too much. A two-hour debug and an architectural pivot are both "high" on this scale, yet they tell very different stories.
- *Quantified minutes*: rejected. Spurious precision. Authors cannot reliably estimate cost in minutes for diverse outcomes, and the false precision invites premature optimization.

**Reversibility:** high. A team that wants ordinal classification can add a parallel field without changing the existing one.

---

### DD-OC-05: Logbook is a separate artifact from the catalog

**Decision:** the catalog records drifts; the logbook records the use of the methodology itself. They are kept as two distinct files.

**Rationale:** these artifacts answer different questions. The catalog answers "where do my briefs fail?"; the logbook answers "where does Plumbline help me?". Mixing them obscures both signals. A drift episode and an experiment in methodology adoption have different cadences, different audiences (the catalog is reusable across teams; the logbook is personal), and different shelf lives.

**Alternatives considered:**
- *Single combined log*: rejected. The signals interfere with each other.
- *No logbook at all*: rejected. Without it, the methodology cannot be improved over time — there is no record of which KPIs were actually used and which were skipped.

**Reversibility:** high. Combining or splitting artifacts is a file-level operation, not a structural one.

---

## Open questions

- Optimal cadence for category review: 20 entries, monthly, both?
- How does the catalog evolve when shared across a team (multiple authors, same project)?
- When does the catalog saturate — i.e., when do new entries stop revealing new categories?
- Should promotion threshold (≥3 occurrences) scale with catalog size, or stay fixed?

These are answered empirically, after sustained use.

---

*Status: Draft v0.1 — May 2026. To be revised after fifteen to twenty real entries.*
