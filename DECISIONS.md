# Plumbline — Methodology Design Decisions

This document records the cross-cutting design decisions that shape Plumbline as a methodology. It does not cover identity choices (name, tone, branding) — those belong elsewhere.

Each decision follows the format:

- **Decision** — one declarative sentence.
- **Rationale** — why, in 1-3 sentences.
- **Alternatives considered** — what was rejected and why.
- **Reversibility** — how hard it is to change later (high / medium / low).

`Reversibility` matters: it signals to adopters which decisions are foundational and which can be customized in their own forks without breaking the methodology.

KPI-specific design decisions live in the `## Design decisions` section of each KPI spec.

---

## DD-01: English is the language of all methodology artifacts

**Decision:** specs, manifesto, schemas, templates, and documentation are written in English. The methodology operates on briefs in any language.

**Rationale:** an open-source methodology aimed at international adoption needs a single authoritative reference language. Mixing or translating core artifacts creates ambiguity about which version is canonical. The user's working language belongs to the user, not the framework — Plumbline adapts to it, not the other way around.

**Alternatives considered:**
- *Bilingual core (EN + IT)*: rejected. Two canonical sources lead to drift between versions.
- *Italian only*: rejected. Limits adoption to the Italian developer community without a meaningful gain.

**Reversibility:** low. The choice of canonical language is foundational — changing it later means rewriting the entire corpus.

---

## DD-02: Plain text artifacts only, no service or database

**Decision:** all Plumbline artifacts (catalog, logbook, briefs, KPI specs) are plain text files — Markdown for prose, YAML for structured data. No database, no service, no UI is required to use the methodology.

**Rationale:** plain text is queryable with universal tools (`grep`, `yq`, custom scripts, AI assistants), survives tool churn, lives in version control alongside the code it describes, and has zero infrastructure cost. A methodology that requires running services to be useful loses adopters at the install step.

**Alternatives considered:**
- *Database-backed catalog*: rejected. The marginal benefit (fast queries) does not justify the infrastructure burden for files that grow at the rate of a few entries per week.
- *Web UI for catalog editing*: rejected. Adds a maintenance surface for a methodology that should remain operational and lightweight.

**Reversibility:** high. Tooling can be layered on top of plain text without invalidating the format. A future `plumb` CLI reads YAML — it does not replace it.

---

## DD-03: The methodology is preventive, not prescriptive

**Decision:** Plumbline measures uncertainty before implementation begins. It does not prescribe phases, agent roles, workflow sequences, or deliverable structures.

**Rationale:** prescriptive methodologies fail under variance — every team, every project, every brief differs enough that rigid sequences create friction. Plumbline is built on the observation that the real bottleneck is not workflow design but the gap between author intent and model interpretation, and that gap can be measured directly without imposing process.

**Alternatives considered:**
- *Phase-based methodology (BMAD-style)*: rejected. Already exists, addresses a different problem (workflow orchestration), and creates rigidity Plumbline explicitly avoids.
- *Role-based methodology (multiple specialized agents)*: rejected. Conflates the question of "how to coordinate AI agents" with the question of "how to verify input quality" — Plumbline is only about the second.

**Reversibility:** low. This decision is the methodology's identity. Changing it produces a different methodology.

---

## DD-04: KPIs are independent of stack, model, and language

**Decision:** the three KPIs (Brief Linting, Assumption Surfacing, Omission Catalog) work regardless of programming language, AI model, framework, or natural language of the brief.

**Rationale:** a methodology tied to specific tools (Claude only, Python only, Cursor only) becomes obsolete when those tools change. The principles behind the KPIs — measure brief quality, surface model uncertainty, catalog omissions — are tool-agnostic. Tooling layers (templates, prompts, future CLI) may be specific; the principles are not.

**Alternatives considered:**
- *Optimize for one model family*: rejected. Creates lock-in and forces re-validation every time models change.
- *Couple KPIs to a specific in-house framework*: rejected. The author's experience with a metadata-driven framework inspired the methodology, but generalizing beyond any single framework is precisely what gives Plumbline its broader applicability.

**Reversibility:** medium. Tool-specific extensions can be added without breaking the principle, but if the core KPIs ever required a specific stack, the methodology would lose its general appeal.

---

## DD-05: Append-only is the convention for all logged artifacts

**Decision:** the Omission Catalog and the Logbook are append-only by convention. Entries are not deleted; entries that prove irrelevant in retrospect are marked (e.g., `relevance: low`), not removed.

**Rationale:** the value of these artifacts is *aggregate*, not individual. Deleting entries — even ones that seem useless in isolation — destroys data that may matter when patterns are extracted later. The fact that an entry was originally recorded is itself information about the author's perception at the time.

**Alternatives considered:**
- *Allow free editing and deletion*: rejected. Same reason archived emails or git history shouldn't be silently rewritten — past observations have evidentiary value even when wrong.
- *Auto-archive low-relevance entries*: rejected. Adds complexity without clear benefit; manual marking is sufficient.

**Reversibility:** high. A team can adopt different conventions in their fork without changing artifact formats.

---

## DD-06: Categories and patterns emerge from data, not from upfront design

**Decision:** Plumbline does not ship with a predefined taxonomy of brief omissions, drift types, or assumption patterns. These structures are observed empirically by users from their own catalog after sufficient entries (~15-20).

**Rationale:** upfront taxonomies in operational catalogs consistently fail because they are built on assumed patterns rather than observed ones. They either miss important distinctions or create phantom ones. Real patterns reveal themselves through aggregation. The methodology trusts users to extract their own patterns from their own data.

**Alternatives considered:**
- *Ship a recommended taxonomy*: rejected. Even well-intended starter taxonomies anchor users on the framework author's blind spots rather than their own.
- *Crowdsource a community taxonomy*: deferred. Could be useful in the long term as Plumbline accumulates community data, but requires maturity the project does not yet have.

**Reversibility:** high. A user or team can choose to seed initial categories without breaking the methodology, accepting the trade-off documented above.

---

## DD-07: Promotion is the mechanism by which the methodology learns

**Decision:** when patterns emerge in the Omission Catalog (a category recurs ≥3 times, an assumption pattern repeats), they are promoted to upstream KPIs — Brief Linting checks or Assumption Surfacing prompt additions.

**Rationale:** without an explicit promotion mechanism, the catalog becomes a passive archive instead of an active feedback loop. The promotion rule turns episodic observations into preventive checks, which is the entire point of recording episodes in the first place.

**Alternatives considered:**
- *No formal promotion (users are expected to internalize patterns)*: rejected. Internalization fails under cognitive load — the patterns evaporate just like the original episodes.
- *Automatic promotion via scripted threshold*: deferred. Possible as future tooling, but premature without sufficient real catalogs to calibrate thresholds.

**Reversibility:** medium. The threshold (currently ≥3 occurrences) can be tuned per team. The mechanism itself is structural to the methodology.

---

## DD-08: Briefs are delivered as standalone .md files

**Decision:** briefs are delivered to implementers as standalone `.md` files, not as inline chat content.

**Rationale:** a brief lives across multiple operations (lint, surfacing, execution, review, archive) and multiple actors (strategic AI, implementer, human reviewer). Inline chat content cannot be referenced unambiguously, versioned, archived, or re-applied without loss. A file is the unit of address for all of these.

**Alternatives considered:**
- *Inline chat as canonical brief*: rejected. Cannot be referenced across sessions or operations without copy-paste, which introduces silent divergence.
- *Hybrid (file for long briefs, inline for short ones)*: rejected. The boundary "long enough to warrant a file" is fuzzy; the resulting inconsistency was the problem PL-017 originally surfaced.

**Reversibility:** high. The format is conventional, not structural. A team adopting an in-IDE brief tool would still produce a file-equivalent artifact.

---

## DD-09: KPI prompts are preparatory, not attached to executive briefs

**Decision:** KPI 01 (Brief Linting) and KPI 02 (Assumption Surfacing) prompts are applied to a brief as preparatory pipeline steps. They are not embedded in the brief consigned to the implementer.

**Rationale:** a KPI prompt and an executive brief have different audiences and different action constraints. The KPI 02 prompt explicitly prohibits action until the three lists are produced; the brief contains action instructions. Embedding the prompt in the brief produces a constraint conflict — the implementer cannot follow both simultaneously without ambiguity. Catalogued as OC-005 after this conflict surfaced in a real cycle.

**Alternatives considered:**
- *Attach KPI prompts to the brief as preamble*: rejected. Creates the constraint conflict above and conflates two distinct phases (auditing the brief vs. executing it).
- *Embed KPI compliance signals inside brief sections (e.g., "this brief passed KPI 01 with N alarms")*: deferred. Plausible long-term, but premature without a defined signal format.

**Reversibility:** high. The separation is a pipeline-stage convention, not a structural property of the brief format.

---

## DD-10: Brief Linting is re-applied at every brief iteration after integration

**Decision:** when a brief is integrated in response to Assumption Surfacing (KPI 02) or any other revision, Brief Linting (KPI 01) is re-applied to the new version before proceeding.

**Rationale:** integration introduces new content — additional sections, conditionals, references, scope clarifications — that can produce new lint alarms invisible to the previous lint pass. Skipping re-lint silently accepts a version of the brief that has not been audited. The cost of re-running KPI 01 is minutes; the cost of executing on an un-linted integrated brief is whatever the missed alarm represents.

**Alternatives considered:**
- *Lint only the original brief*: rejected. The pattern PL-018 surfaced — integration introducing alarms — appears to be structural, not exceptional.
- *Diff-only lint (lint only what changed)*: deferred. Plausible optimization, but requires diff tooling Plumbline does not yet have and would add complexity for marginal benefit at current scale.

**Reversibility:** medium. The rule is procedural and easily adjustable, but its absence reintroduces the audit gap; reverting would be a regression unless replaced by an equivalent mechanism.

---

## DD-11: The parking-lot is a living artifact

**Decision:** Plumbline maintains a parking-lot as a living artifact alongside catalog, logbook, and session-notes. The parking-lot records questions raised during cycles that cannot be resolved in the current session and should not be lost. Entries are added on the spot, then resolved or abandoned over time; they are not deleted.

**Rationale:** without an external memory for unfollowed branches, questions raised mid-session evaporate when the strategic AI's working memory cycles or the human moves on. Catalog records drift episodes, logbook records cycle use, but neither captures unresolved questions. The parking-lot fills that gap with minimal cost (one entry, one minute) and provides the input that the promotion mechanism (DD-07) operates on.

**Alternatives considered:**
- *Keep open branches in session memory only*: rejected. The pattern that triggered this DD — branches lost between sessions — is exactly the failure mode this rejects.
- *Use existing tools (issue tracker, todo list)*: rejected. Existing tools are project-scoped or task-scoped; the parking-lot is methodology-scoped and lives alongside the methodology's own artifacts. Mixing them obscures the methodology's own learning signal.

**Reversibility:** high. The artifact is plain markdown with a minimal schema; a future split (methodology-scope vs project-scope) is already anticipated in the file's own provisional scope decision and would not invalidate the DD.

---

## DD-12: Spec artifacts and operational artifacts are stored separately

**Decision:** Plumbline distinguishes spec artifacts (KPI specs, manifesto, schemas, templates) from operational artifacts (catalog, logbook, parking-lot, session-notes, decisions). Spec artifacts are pedagogical and may contain examples; operational artifacts contain only real entries produced during real use.

**Rationale:** mixing pedagogical examples with real entries pollutes both. A reader of the operational catalog cannot trust that entries reflect actual drift episodes; a reader of a spec cannot find the canonical guidance without sifting through accumulated data. Separation preserves the integrity of each: the spec teaches, the operational artifact records.

**Alternatives considered:**
- *Single file per concept (spec + examples + real entries together)*: rejected. The pollution effect above is severe enough that even a small operational artifact becomes hard to read after a few months.
- *Tag-based distinction inside a single file (e.g., `type: pedagogical` vs `type: real`)*: rejected. Adds schema complexity, and visually scanning a tagged file is harder than opening a different file. Directory separation is cheaper.

**Reversibility:** high. Both kinds of artifact are plain text; merging or splitting them is a file operation, not a methodology change.

---

## DD-13: Session-notes is a living artifact

**Decision:** Plumbline maintains session-notes as a living artifact alongside catalog, logbook, and parking-lot. Session-notes records the methodological reasoning that occurred during a session — micro-patterns, half-formed intuitions, connections made in real time — that informed decisions but did not reach a formal artifact (DD, KPI rule, catalog entry, parking-lot item). Notes are append-only by convention.

**Rationale:** catalog records drift episodes, logbook records cycle use, parking-lot records unfollowed branches. None of these capture the *reasoning around* the events — the why behind a decision, the intuitions noticed but not codified, the connections that informed a triage. Without session-notes, that material lives only in the strategic AI's session memory and is lost when the session ends. Session-notes is the scratchpad that makes the methodology's own thinking durable across sessions.

**Alternatives considered:**
- *Promote significant reasoning to DD or KPI rules immediately*: rejected. Premature promotion is the failure mode the 2-3 cycles rule (DD-07 promotion threshold) prevents; reasoning needs a place to live *before* it's mature enough to promote.
- *Use chat history as the durable record of reasoning*: rejected. Chat history is not addressable, not editable, and is bound to a specific AI session. It cannot be reviewed at a future consolidation session the way an artifact can.

**Reversibility:** high. The artifact is plain markdown with no required structure beyond dating and topic grouping; teams can adapt the conventions in their own forks.

---

## DD-14: Methodology development and methodology application are distinct activities

**Decision:** Plumbline recognizes two distinct activities that frequently flow through the same session: *methodology development* (evolving Plumbline itself — DDs, KPI rules, structural patterns) and *methodology application* (using Plumbline on a target project — architectural decisions, product decisions, technical investigations). Operational artifacts that can mix the two must preserve the distinction; artifacts that are mono-activity by construction do not.

**Rationale:** the distinction is invisible to *pure adopters* (everything is application) and *pure contributors* (everything is development), but for *author-appliers* — who develop the methodology while running it on their own project — both activities share the same conversations and accumulate into the same artifacts. Without a way to tell them apart, methodology-level patterns get buried in project noise, and project decisions get treated as methodology data. The catalog, logbook, session-notes, and decisions are methodology-development by construction; the parking-lot is the one operational artifact where the distinction must be applied, and the existing `Type` field already discriminates (`methodology-gap` → development; `architectural-decision` / `product-decision` / `technical-investigation` → application).

**Alternatives considered:**
- *Treat all artifact content as a single scope*: rejected. The first end-to-end cycle showed parking-lot entries from the two activities accumulating without separation, and triage required disambiguating them by hand — a cost that grows with artifact size and is silent until it becomes painful.
- *Hard separation into distinct files today* (`plumbline/parking-lot.md` for methodology-scope, `docs/parking-lot.md` per project repo): rejected for now. The parking-lot file itself anticipates this split as a future state, but prescribing it after a single cycle would codify a filesystem structure before there is evidence the split reduces friction more than it adds overhead. To be reconsidered after Plumbline is applied to a second project, or after the parking-lot crosses a size threshold where mixed-scope review becomes painful.

**Reversibility:** medium. The principle — that the two activities are structurally distinct — shapes how every artifact is read and is close to identity-level (compare DD-03). The mechanism — derivation from the existing `Type` field, consistent file sectioning — is conventional and easily revisable. A future formalization (separate files, explicit `Scope` field) is anticipated but not foreclosed.

---

*Status: Draft v0.1 — May 2026. Decisions added as they emerge from methodology development.*
