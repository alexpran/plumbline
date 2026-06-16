# KPI 02 — Assumption Surfacing

> **A structured pass in which the model declares its assumptions, ambiguities, and open questions, before producing any output.**
> The brief, audited by the entity that will implement it.

## Why this exists

Brief Linting reads the brief from the outside, against fixed categories. Assumption Surfacing reads the brief from the inside, by asking the model what it would have to *interpret* in order to act on it.

The two operations are complementary. Brief Linting catches structural omissions visible at the surface of the text. Assumption Surfacing catches semantic ambiguities visible only when the brief is being processed for execution.

Models are good at exposing uncertainty when explicitly asked, and bad at exposing it when left to decide on their own. Default-mode generation hides interpretive gaps under fluent prose. A dedicated pass — one that prohibits implementation and demands only declarations of uncertainty — extracts the data that would otherwise become silent drift.

## When to run it

After Brief Linting has been passed (or accepted with known alarms), before the implementation prompt.

The pass is meant for non-trivial briefs. For trivial ones, the overhead exceeds the benefit. As with Brief Linting, the calibration is rough: features that produce more than ten lines of code or modify more than one file qualify.

The pass runs in a **fresh chat**, with the brief attached as context, and no prior implementation history. Prior history contaminates the pass — the model carries forward assumptions made in earlier exchanges instead of surfacing them as new declarations.

## The prompt

The exact prompt to use. Copy and paste, do not paraphrase. Wording is calibrated for structured output.

> Before implementing the feature described in the brief below, do NOT write any code, configuration, or implementation. Reply only with the three numbered lists that follow.
>
> 1. **ASSUMPTIONS.** Every interpretive assumption you are making about the brief. For each, state: (a) what you are assuming, (b) why, (c) confidence on a 1–5 scale.
>
> 2. **AMBIGUITIES.** Every point in the brief that admits more than one interpretation. For each, state: (a) the ambiguous point, (b) the possible interpretations, (c) which one you would choose if I do not respond.
>
> 3. **QUESTIONS.** The questions you would ask a product manager if one were available. Maximum 5, ordered by impact on implementation.
>
> Respond in the same language as the brief. Do not add a preamble, conclusion, or commentary. Only the three lists.

The constraints — fresh chat, no implementation, fixed format, no preamble — are not stylistic. Each one closes a known failure mode of unstructured surfacing: context contamination, accidental implementation, output that drifts into prose, hedging that drowns the signal.

## Reading the result

The output is interpreted against five patterns. The decision rule below is operational, not exhaustive — calibrate it after a few cycles.

| Output pattern | Interpretation | Action |
|---|---|---|
| 0 assumptions, 0 ambiguities, 0 questions | Either the brief is genuinely complete, or the model is suppressing uncertainty | **Suspect.** Re-run with a more aggressive prompt variant, or run Brief Linting if not done yet. |
| 1–3 small assumptions, all confidence 4–5 | Healthy brief, normal interpretive residue | **Proceed.** Optionally confirm the riskiest assumption before implementing. |
| 4–7 mixed assumptions, several at confidence 2–3 | Recoverable holes | **Iterate.** Answer the ambiguities and questions, integrate into the brief, re-run the pass. |
| 8+ assumptions, many low-confidence, structural ambiguities | Brief is not ready | **Stop.** Rewrite the brief; do not patch it via clarifications in chat. |
| Questions concern architecture, not details | Misalignment is at the "what", not the "how" | **Stop.** Architectural discussion is needed before returning to brief-level work. |

A note on hedging: not all assumptions count equally. The model will sometimes list trivially true assumptions to look thorough (*"I assume the database is reachable", "I assume the tests will pass"*). These are noise. The signal is in assumptions that, if false, change what gets built.

After a few cycles of running the pass, the user develops calibration for distinguishing signal from noise. Until that calibration is in place, treat all listed assumptions as potentially relevant.

## What to do with the output

1. **Triage** the assumptions: signal vs. noise.
2. **Confirm or correct** signal assumptions in the brief itself, not in the chat. The brief is the durable artifact; chat is volatile.
3. **Resolve** ambiguities by choosing the interpretation explicitly and updating the brief.
4. **Answer** the questions in the brief if they are content questions; defer them to architectural discussion if they are scope questions.
5. **Re-run the pass** in a fresh chat with the updated brief. A clean run (pattern 1–3 small assumptions) is the green light to implementation.

The cycle is short by design. Most briefs need one or two iterations of this pass before they are ready. Briefs that need five or more iterations are signaling that the underlying problem is not yet understood; iteration on the brief is no longer the right tool.

## Calibration to the author's working language

The prompt instructs the model to respond in the brief's language. The prompt itself is in English to maximize structured-output compliance — models trained primarily on English instructions follow strict format rules better in their training language, even when generating output in another language.

For users in non-English working languages, the resulting interaction is bilingual: prompt in English, brief in the working language, output in the working language. This is by design and not friction worth eliminating.

## Companion artifact: assumption reading template

When the output of the surfacing pass is non-trivial, capturing it in a structured form helps. The companion artifact `templates/assumption-reading.md` (to be added) provides a short template:

- For each assumption: signal/noise classification, resolution status, confidence-after-resolution.
- For each ambiguity: chosen interpretation, justification.
- For each question: answer or deferred-to-architecture.

The template is optional but reliably useful when the brief warrants more than one iteration of the surfacing pass.

---

## Design decisions

### DD-AS-01: Strict format with no preamble

**Decision:** the prompt enforces three numbered lists with no preamble, no conclusion, no commentary.

**Rationale:** unstructured outputs from this kind of pass drift into reassurance ("the brief is mostly clear, but I have a few minor points...") that buries the actual signal. Structured outputs force the model to produce the data, not narrate around it. The format also makes downstream triage faster.

**Alternatives considered:**
- *Free-form analysis*: rejected. Produces hedged, narrative output that is harder to parse and easier to skim past.
- *More categories (e.g., risks, dependencies, performance considerations)*: rejected. Each category dilutes the signal of the others. Three categories is the maximum that holds across diverse briefs.

**Reversibility:** medium. Format can be extended; removing the strict-format constraint reverts the KPI to general-purpose feedback, which is no longer Assumption Surfacing.

---

### DD-AS-02: Confidence scale on assumptions, not on the brief as a whole

**Decision:** the model rates confidence per assumption, not on the brief overall.

**Rationale:** an overall confidence score is an averaged signal that hides where the uncertainty actually lives. Per-assumption confidence preserves locality: it identifies which specific interpretive moves are unstable. The triage step then operates on per-assumption data, not on a composite.

**Alternatives considered:**
- *Single overall confidence score*: rejected for the reason above.
- *Confidence on ambiguities and questions too*: rejected. Ambiguities are by definition uncertain; rating them is redundant.

**Reversibility:** high. Confidence reporting can be enriched without breaking the methodology.

---

### DD-AS-03: Fresh chat for every pass

**Decision:** the pass runs in a chat with no prior history beyond the brief itself.

**Rationale:** prior chat history contaminates the surfacing in two directions. Earlier discussions cause the model to silently incorporate decisions that should be re-surfaced; earlier corrections cause the model to suppress assumptions it has already been "trained" to make in this session. A fresh chat treats the brief as the only authoritative input and forces the model to surface everything it would interpret freshly.

**Alternatives considered:**
- *Continue in the same chat used for brief discussion*: rejected. Convenient but defeats the KPI's purpose.
- *Maintain a long-running "audit" chat across briefs*: rejected. Same contamination, scaled across features.

**Reversibility:** low. This decision is structural to how the KPI works.

---

### DD-AS-04: English prompt, working-language response

**Decision:** the prompt is in English and instructs the model to respond in the brief's language.

**Rationale:** models follow strict format constraints more reliably in their training language, even when producing output in another language. An English prompt with a localized response gets the best of both: maximum compliance with the format, minimum translation overhead for the user.

**Alternatives considered:**
- *Localize the prompt to each working language*: rejected. Maintains N versions of the canonical prompt with subtle drift between them; format compliance suffers.
- *Force English responses regardless of brief language*: rejected. Adds translation overhead for the user with no methodological benefit.

**Reversibility:** medium. Users can localize their copy of the prompt if they observe better results in their language; the canonical prompt remains in English.

---

## Open questions

- Does prompt effectiveness degrade for very long briefs (the model "drowns" in context)?
    - *One data point (single project, n=1)*: the pass ran on a large integrated brief and held format compliance. Beyond that, it surfaced items structurally invisible to Brief Linting — details that exist only at the intersection of brief and codebase, not on the surface of the brief in isolation. Not a definitive answer, but positive evidence, and empirical support for the complementarity claim in "Why this exists".
- Should the prompt be repeated more than once per iteration to catch latent assumptions on a second pass?
- What is the right calibration for "many low-confidence assumptions" (4? 5? 7?) — does this scale with brief length?
- Is there a model-specific tuning required (e.g., different prompt for reasoning models vs. base instruction-tuned models)?

---

*Status: Draft v0.1 — May 2026. The prompt itself is the artifact most likely to need iteration. Calibrate from real-world output patterns.*