# KPI 01 — Brief Linting

> **Static signals on the brief itself, before any prompt is sent.**
> A 90-second checklist that flags categories of insufficiency a human author tends to miss.

## Why this exists

Briefs feel complete when their author writes them. The author has the full context in working memory — the missing parts are filled in mentally and never reach the page. The model, lacking that context, hallucinates the missing parts with confident-sounding output.

Brief Linting is the discipline of reading the brief as if you were not its author. The checklist makes this possible by naming categories of omission an author cannot see, but a structured pass can detect.

It is the cheapest of the three KPIs to apply: no model interaction, no log discipline, no waiting. It is also the one with the highest false-negative rate — categories not yet promoted from the Omission Catalog will be missed. Brief Linting alone is not enough; it is the first filter.

## When to run it

After the brief is written, before the implementation prompt is sent to the model.

The pass is meant to be applied to every non-trivial brief. "Non-trivial" is a judgment call; for calibration, anything that produces more than ten lines of code or modifies more than one file qualifies.

Skipping the pass is cheap on the surface (saves 90 seconds). It is expensive on aggregate, because the omissions Brief Linting would have caught become exactly the entries that end up in the Omission Catalog later, with full debugging cost attached.

## The checklist

Eight categories. For each, ask: *does the brief address this, explicitly, in writing?* Mental answers do not count. If the answer is "I know what I meant but it's not written down", the category is in alarm.

| # | Category | Trigger to look for | Alarm threshold |
|---|---|---|---|
| 1 | **Entities named but not defined** | Domain nouns used without specifying fields, states, relationships | ≥ 1 central entity undefined |
| 2 | **Verbs without structured object** | Actions described without specifying actor, role, permissions, effects | ≥ 2 occurrences |
| 3 | **References to "existing logic" without anchor** | Phrases like *"as in module X"* without pointer to brief, code, or spec | ≥ 1 unanchored reference |
| 4 | **Unresolved conditionals** | *"If / when / unless / optionally"* without specifying the branch | > 3 open conditionals |
| 5 | **Missing unhappy path** | Only success scenarios described; no errors, no edge cases | always alarm if absent |
| 6 | **Permissions / roles unmentioned** | Multi-actor feature without specifying who can do what | always alarm if ≥ 2 actors involved |
| 7 | **Initial state undeclared** | Stateful feature without specifying starting point | always alarm if feature is stateful |
| 8 | **Duplicate-input behavior unspecified** | Create / import / register operations without specifying behavior on existing record | always alarm for create-like operations |

## Reading the result

The output is not a score. It is a list of categories in alarm.

- **0–1 alarms** → brief is ready. Proceed to implementation.
- **2–3 alarms** → integrate the brief addressing the alarmed categories, re-lint.
- **4+ alarms** → brief is not ready. Rewrite, do not patch.

The threshold of 2 is deliberately low. A brief in alarm on two categories will reliably produce drift in the implementation pass; the time saved by patching is paid back triple in debugging.

## Worked example (illustrative) — `suggest_related_tags`

> A constructed example, not a logged episode. It shows how the catalog entries from
> one feature map back to alarms that Brief Linting would have raised at lint time.

Consider a capability that suggests related tags for an article being drafted in a
content editor. Four drifts surfaced during its implementation (catalogued as
`OC-001`–`OC-004` in the Omission Catalog spec). Each traces back to a specific Brief
Linting category that would have been visible at lint time:

| Catalog entry | Brief Linting category that would have caught it |
|---|---|
| `OC-001` (state read from the wrong source) | #7 Initial state undeclared |
| `OC-002` (model fills a parameter the runtime should inject) | #6 Permissions / roles unmentioned (the "actor" here is the framework vs. the model) |
| `OC-003` (taxonomy returns zero candidates for niche topics) | #1 Entities named but not defined (assumed properties of the taxonomy not declared) + #5 Missing unhappy path (zero-result case) |
| `OC-004` (routing ambiguity between capabilities) | #4 Unresolved conditionals (which capability fires when a prompt matches several) |

The brief, lint-checked, would have shown four alarms across four categories. By the
threshold above, that is rewrite territory — not patch.

Brief Linting in retrospect is easier than at the time. The point is that the
categories are *visible from the surface of the brief*, before any code is written.

## Calibration to the author's working language

Categories are language-agnostic. Triggers are not.

For category #4 (unresolved conditionals), the trigger is *"if / when / unless / optionally"* in English. In Italian, it is *"se / quando / a meno che / qualora / opzionalmente"*. In any other language, the same logical role is filled by different words.

To calibrate Brief Linting to a working language, fill in once the table below in the user's own copy of this file:

| Category | English triggers | Working-language triggers |
|---|---|---|
| #1 | *"a/an X", "the X"* without definition section | (fill in) |
| #4 | *"if, when, unless, optionally"* | (fill in) |

The other categories are mostly structural (presence/absence of sections) and translate without trigger calibration.

This is a one-time setup, not a recurring task.

## Companion artifact: brief template

Brief Linting can be a checklist applied after writing, or a structure imposed during writing. The companion artifact `templates/brief-template.md` (to be added) embeds the eight categories as required sections in a brief skeleton. A brief written from the template is alarm-free by construction.

The template is optional. Some authors prefer to write freely and lint after; others prefer to fill a structure. Both are valid. The KPI is the same.

---

## Design decisions

### DD-BL-01: Eight categories, no more

**Decision:** the checklist has eight categories. Adding more is allowed only via promotion from the Omission Catalog.

**Rationale:** a checklist of more than ten items stops being scanned and starts being skipped. Eight is the upper limit for a 90-second pass. Promotion from the catalog ensures new categories arrive only when justified by real recurrence in the user's own briefs.

**Alternatives considered:**
- *Comprehensive list of 15-20 categories from the start*: rejected. Adoption fails before the value emerges.
- *Three or four high-level categories*: rejected. Too coarse to be actionable; alarms become impossible to disambiguate.

**Reversibility:** medium. A team can extend the checklist within their own fork, but should resist doing so without three or more catalog entries justifying each new category.

---

### DD-BL-02: Output is qualitative, not a score

**Decision:** the result of Brief Linting is a list of categories in alarm. There is no aggregate score (no "Brief Health: 73/100").

**Rationale:** composite scores hide signal instead of revealing it. A brief with 73 can have a debilitating omission in one category and clean reads everywhere else, or have mild issues spread across many. The two situations require different responses. A category-level output preserves this information; a numeric score destroys it.

**Alternatives considered:**
- *Weighted score per category*: rejected. Weights are arbitrary, and tuning them is an exercise without empirical grounding.
- *Pass / fail boolean*: rejected. Loses the ability to distinguish between "needs integration" and "needs rewrite".

**Reversibility:** high. A team that wants a derived metric can compute one from the alarm list without changing the KPI itself.

---

### DD-BL-03: Triggers are calibrated, categories are not

**Decision:** the categories are fixed across languages and contexts; only the linguistic triggers (the words that signal a conditional, an action verb without structure, etc.) are calibrated by the user.

**Rationale:** the categories represent universal properties of insufficient briefs (entities not defined, edge cases not addressed, etc.) that hold across human languages and technical domains. The triggers — the surface markers that flag those properties — are language-specific. Separating them keeps the methodology portable while remaining usable.

**Alternatives considered:**
- *Translate categories themselves to other languages*: rejected. Categories are conceptual; translation introduces drift in meaning.
- *Provide pre-built trigger sets for major languages*: deferred. Useful in the long run but premature without community contributions.

**Reversibility:** high. Trigger sets are user data, freely editable.

---

## Open questions

- Should categories #5–#8 (the "always alarm if absent" ones) be enforced in the brief template by required headers, removing them from the lint pass entirely?
- How do the alarm thresholds change for very short briefs (e.g., under 50 words)?
- Is there a category of insufficiency that recurs across users but is missing from this list?

---

*Status: Draft v0.1 — May 2026. To be revised after the first KPI 03 promotion cycle.*
