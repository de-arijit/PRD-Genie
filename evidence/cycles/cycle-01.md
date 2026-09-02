# Evaluation Cycle Log — Cycle #1

*One of these per Run→Score→Learn→Change→Re-score iteration. Copy this template for each new cycle.
Feeds both the dashboard's cycle log (via `POST /cycles`) and the Q3/Q4 writeup directly — write it in
language you'd be comfortable pasting straight into the submission.*

**Date:** 2026-09-02
**Trigger for this cycle:** The baseline run (Task 6) found 3 failures — T4, T7, T11 — out of 12
tests, all in PRD Generation. Two spec corrections were also made to `ground-rules.md` at the same
time: T7's "classified as NFRs" clause moved from Requirement Extraction (which has no FR/NFR
concept) into a new PRD Generation note scored against Section 4.2; T4 split into two stages
(PRD Generator's Section 5, and a new Story-Breakdown-stage row for verbatim carry-forward), since
the original single T4 row conflated both.
**Cycle type:** ☒ Correctness (fixing a failed/hallucinated test) ☐ Token optimization ☐ Both

---

## 1. Run

- Which test(s): **T4** (both stages: PRD Generator's Section 5, and the new Story-Breakdown-stage
  row), **T7** (PRD Generator's Section 4.2 NFR table — first attempt under the corrected spec, not
  a re-run), **T11** (PRD Generator, Persona Key Need grounding). Targeted only — the other 9
  numbered tests plus the 3 Gap Analysis sub-tests already passed clean in the baseline and are
  re-run in Section 5 below purely as a regression check, not because they were in scope for a fix.
- Which model tier served each: OpenAI (`gpt-4o-mini`) for every invocation this cycle — fallback
  tiers are not wired yet (Loop-phase work).
- Pipeline version / prompt version tested: Requirement Extractor, Story Breakdown, and Gap Analyzer
  prompts are byte-identical to the baseline run (untouched, per this cycle's explicit scope). PRD
  Generator's prompt is the one thing that changed — every result below is labeled "before" (the
  original, unmodified PRD Generator prompt) or "after" (the changed prompt, see Section 4).

## 2. Score

Compare against the corrected `ground-rules.md`. Fill one row per test run this cycle.

### Before any change (confirming reproducibility, plus first-ever T7 attempt)

| Test | Pass/Fail | Hallucination? | Notes |
|------|-----------|-----------------|-------|
| T4 — Section 5 (before) | **FAIL** | No | Reproduced exactly: "PDF must include company logo." became "The export feature must allow users to successfully generate and download reports in PDF format that include the company logo." — paraphrased, not verbatim. Same failure mode as the baseline run. |
| T4 — Story stage (before) | **FAIL** | No | Unexpected: Story Breakdown responded "No user stories generated — the PRD's Functional Requirements section had nothing to break down," despite the PRD's Section 4.1 clearly containing 3 real rows. This is Story Breakdown's own first-ever exposure to a T4-shaped PRD (never tested before this cycle) — flagged as a one-off non-deterministic misfire pending the "after" re-run, not assumed to be systematic. |
| T11 (before) | **PASS on this specific attempt** | No | Did **not** reproduce the baseline's hallucination — Persona Key Need correctly read "Not specified in source — flagged for stakeholder input" this time, and Priority for FR-001 correctly read "Should" (matching the true source wording) rather than the baseline's "Must." This is an important, honest finding: the original T11 hallucination is **not deterministically reproducible** — it fired once (baseline) and didn't fire here. The underlying prompt gap that *allowed* it is still real regardless of whether this specific sample triggered it. |
| T7 (before / first attempt) | **FAIL** | No | Section 4.2 (Non-Functional Requirements) reads "Not specified in source" — all 3 technical constraints (10,000 users, 200ms p95, Salesforce REST API v52) landed only in Section 4.1 as Functional Requirements, correctly worded and unrounded, but never classified as NFRs anywhere. Confirms the corrected spec's new criterion is unmet under the current (pre-fix) prompt, as expected — PRD Generator's Section 4.2 instruction is just a bare table header with no guidance on what belongs there. |

**Aggregate (before):** 1 / 4 passed on this specific sample · hallucination rate 0% · fallback rate 0%
*(T11's "pass" here reflects sampling variance, not a resolved issue — see Learn below.)*

### After the change (Section 4)

| Test | Pass/Fail | Hallucination? | Notes |
|------|-----------|-----------------|-------|
| T4 — Section 5 (after) | **PASS** | **Yes** | All 3 acceptance criteria now byte-for-byte verbatim quotes ("Users need to export reports as PDF and CSV.", "PDF must include company logo.", "CSV must preserve formulas."). The targeted fix worked cleanly. But Section 1 (Product Overview) now contains invented values never in any source — "Product Name: Report Export Feature", "Document Version: 1.0", "Status: Draft" — none of which Section 1's (unchanged) instructions were touched by this cycle's edit. Logged as a hallucination per the cross-cutting groundedness rule, even though the specific Section-5 criterion this test targets passes. |
| T4 — Story stage (after) | **FAIL** | No | Story Breakdown correctly generated 3 stories this time (confirming the "before" result was indeed a one-off misfire, not systematic — see Learn). But each criterion got grammatically restructured to fit "I want [goal]" phrasing (e.g. "PDF must include company logo" → "I want the PDF to include the company logo") — not byte-verbatim, so still fails the corrected spec's literal "unparaphrased" requirement. See Learn for why this is a structural tension, not a simple bug. |
| T11 (after) | **FAIL** | **Yes** | Persona Key Need now correctly reads "Not specified in source" — the targeted hallucination is fixed. But, same pattern as T4-after: new invented content appeared in Section 1 (Product Name/Version/Status), Section 2 (Success Metrics: "Reports filtered effectively and results loading time under defined thresholds" — not stated anywhere), and Section 8 (Assumptions: "The filtering requirements are finalized and not subject to change" — invented). This directly triggers T11's own FAIL condition ("fills a section with generic filler") via different sections than the original failure. Net result: **T11 still fails**, for a different and now broader reason. One genuine improvement: Section 10 (Timeline) correctly picked up "Deadline: Q3" this time, which neither "before" run had used despite it being available all along. |
| T7 (after) | **FAIL** | No | Unchanged, as expected — this cycle's fix didn't touch Section 4.2 or NFR classification at all. Section 4.2 still reads "Not specified," all 3 constraints still only in 4.1. Notably, Section 1 in *this* run was correctly "Not specified" throughout (no invention) — the Section-1-invention issue did not appear here, reinforcing that it's stochastic rather than a deterministic regression tied to this cycle's specific edit (2 of 3 "after" runs showed it, this one didn't). |

**Aggregate (after, targeted 4):** 2 / 4 passed on their named criterion · hallucination rate 50%
(2/4 — both from the new Section 1/2/8 issue, not from the criteria being fixed) · fallback rate 0%

### Full regression check (per this template's own instruction — Story Breakdown consumes PRD
Generator's output directly, so it's the most likely place a regression would surface)

| Test | Pass/Fail | Hallucination? | Notes |
|------|-----------|-----------------|-------|
| T1 | PASS | No | Unaffected (Requirement Extractor untouched) — consistent with baseline. |
| T2 | PASS | No | Unaffected — consistent with baseline. |
| T3 | PASS | No | Unaffected — consistent with baseline. |
| T5 | PASS | No | Unaffected — consistent with baseline. |
| T6 | PASS | No | Unaffected — consistent with baseline. |
| T7 (Requirement Extraction — numbers preserved) | PASS | No | Unaffected — the numbers-preservation half of T7 (now a separate row from the NFR note) still passes; only the new PRD-Generation NFR row fails (see above). |
| T8 | PASS | No | Unaffected — consistent with baseline. |
| T9 | PASS | No | Unaffected — consistent with baseline. |
| T10 | PASS | No | Unaffected — consistent with baseline. |
| T12 | **PASS** | No | **The key regression check.** Story Breakdown ran against the *new* (after-fix) T11 PRD output — 2 stories, correct "As a [persona], I want…, so that…" format, both with priority tags and FR source IDs, NFRs correctly excluded. No regression — T12 holds up cleanly even against the changed upstream PRD. |
| T2 (Gap Analysis) | PASS | No | Unaffected — 2 concrete questions, consistent with baseline. |
| T5 (Gap Analysis) | PASS | No | Unaffected — 5 concrete questions, consistent with baseline. |
| T9 (Gap Analysis) | PASS | No | Unaffected — asks for the actual input, consistent with baseline. |

**Aggregate (regression, 13 tests):** 13 / 13 passed · hallucination rate 0% · **zero regressions
found.**

### Cycle 1 grand total (corrected-spec structure — 14 numbered-capability rows, since T4 and T7
now each span two stages, plus 3 Gap Analysis sub-tests tracked separately)

**11 / 14 passed** · **2 / 14 hallucination_detected: true** (T4-Section5, T11 — both from the new
Section 1/2/8 issue) · Gap Analysis: **3 / 3 passed, 0 hallucinations**

## 3. Learn

**T4 / Section 5 — root cause:** Section 5's original placeholder text was `(Per key feature,
specific testable criteria.)` — this describes an *authoring* task ("write testable criteria"), not
an *extraction* task ("copy the stated criteria"), and nothing in that specific line said
"verbatim." The prompt's general verbatim-preservation guidance existed elsewhere (in a paragraph
discussing the Functional Requirements table's Source column), but was never explicitly re-anchored
to Section 5 itself — so the model defaulted to its natural instinct to produce a polished,
complete-sentence "criterion" rather than quote the source. This is exactly the kind of "competing
instruction" the task's own hypothesis named: nothing forbade rewriting into full sentences, and
"write testable criteria" reads more naturally as an invitation to draft than to quote.

**T11 hallucination — root cause:** The Persona block's `- **Key Need:**` and
`- **Current Workaround:**` sub-fields had *zero* attached instructions — just the bare label. Rule
1's general "never invent" guidance exists elsewhere in the prompt, but the field itself gave no
signal that a plausible-sounding inference about what a "Project Manager" would typically need is
still an invention if the input never actually said it. This matches the task's own hypothesis
precisely: the fallback rule wasn't triggered because the *persona itself* had something concrete
(Sarah's name and role), and that concreteness seems to have made the model treat the *whole
persona block* as "has enough to work with," rather than checking each sub-field's grounding
independently. Confirmed non-deterministic via the "before" reproduction: it fired in the original
baseline sample and did not fire in this cycle's reproduction attempt — the instruction gap is real
and worth closing regardless of any single sample's outcome.

**New finding (not hypothesized in advance): Section 1/2/8 invention.** After the fix, 2 of 3 PRD
Generator runs (T4, T11) invented content in Product Overview, Success Metrics, and/or Assumptions —
sections whose instructions this cycle's edit never touched. The third run (T7) showed no such
invention. This is not attributable to the specific wording I added (Section 5 and Persona block
only) — it's most plausibly either (a) pre-existing model variance that the small "before" sample
size (2 runs) simply didn't happen to surface, or (b) a subtle emergent effect where making some
parts of the prompt more detailed/confident-sounding shifts the model's overall register elsewhere
in the same generation. Given the mixed evidence (T7's clean run undercuts a simple "my edit caused
it" story), I'm not attempting a fix this cycle — it's outside the two root causes this cycle scoped
to fix, and deserves its own dedicated Learn step with a larger sample before changing anything.
Flagged as the top carried-forward priority below.

**Cross-agent drift (informational, not fixed this cycle, as instructed):** Requirement Extractor's
own prose sometimes paraphrases "should" as "must" while its quoted `source:` field stays accurate.
Observed both "Should" and "Must" priorities for the *same* T1 input's FR-001 across different
re-runs this cycle and the baseline — confirming this is genuinely non-deterministic (which field
PRD Generator's priority-inference logic weighs more heavily varies run to run), not a fixed
one-directional bug. Decision: **worth a future cycle**, and the fix belongs in PRD Generator (key
priority-inference off the quoted `source:` field only, never the paraphrased summary line) rather
than in Requirement Extractor, since Requirement Extractor's own quoted field is already correct —
it's PRD Generator's consumption of it that's inconsistent.

## 4. Change

- Exact change made (`architecture/agent-prompts/02-prd-generator.md`):

  ```
  BEFORE:
    - **Key Need:**
    - **Current Workaround:**
  )

  AFTER:
    - **Key Need:** Fill this ONLY if the input explicitly states or directly implies what this
      specific person needs — not what someone in this role would plausibly need in general. A
      guess that sounds reasonable for the persona's job title is still an invented claim if the
      input never actually says it. Having a name or role for the persona is not, by itself,
      grounds for filling in a need — if the input gives you the persona's identity but nothing
      about what they need, write "Not specified in source — flagged for stakeholder input" here.
    - **Current Workaround:** Same rule as Key Need — fill it only if the input states one;
      otherwise "Not specified in source — flagged for stakeholder input."
  )
  ```

  ```
  BEFORE:
  ## 5. Acceptance Criteria
  (Per key feature, specific testable criteria.)

  AFTER:
  ## 5. Acceptance Criteria
  (Per key feature: copy the exact source wording as the criterion, verbatim. Do NOT rewrite it into
  a complete sentence, do NOT restate it in your own words, and do NOT turn it into a "Users can…"
  or "The system must allow…" style description. If the extractor's line says "PDF must include
  company logo," the criterion is exactly "PDF must include company logo" — nothing more polished,
  nothing rephrased. If a feature has no source-stated criterion, write "Not specified in source —
  flagged for stakeholder input" for that feature instead of drafting one yourself.)
  ```

  (A short pointer to this cycle log was also added to the prompt file's own "Notes for whoever
  wires this into n8n" section — documentation only, not a behavioral change.)

- Why this specific change should fix the root cause identified above: Section 5's new text
  explicitly forbids the exact failure mode observed ("do NOT turn it into a 'Users can…' style
  description") and gives a matched worked example using the actual T4 input, closing the ambiguity
  that let the model default to drafting. The Persona block's Key Need/Current Workaround now
  explicitly names the exact reasoning shortcut that caused the hallucination ("having a name or
  role is not, by itself, grounds for filling in a need") — directly countering it rather than
  relying on the general Rule 1 to be inferred at this specific field.
- Scope of the change: **PRD Generator's prompt only**, localized to two OUTPUT-section field-level
  instructions. No RULES renumbering, no changes to Requirement Extractor, Story Breakdown, or Gap
  Analyzer, per this cycle's explicit scope.

## 5. Re-score

- Same test(s), same input(s), re-run against the changed prompt: **T4 (both stages), T11, T7** —
  see the "After the change" table in Section 2 above for full results.
- Result: pass/fail now vs. before —
  - T4/Section 5: FAIL → **PASS** (fixed, though a new hallucination surfaced elsewhere in the same
    output — see below).
  - T4/Story stage: FAIL → **FAIL** (different reason: the "nothing to break down" misfire didn't
    recur, but verbatim-in-story-format is a structural tension, not something this cycle's change
    could have fixed since Story Breakdown is out of scope).
  - T11: FAIL (baseline) → PASS (this cycle's before-reproduction, non-deterministically) →
    **FAIL** (after-change, for a new reason — Key Need is fixed, Section 1/2/8 now fails instead).
  - T7: FAIL → **FAIL** (unchanged, correctly untouched this cycle, as intended).
- Did the fix regress anything that was previously passing? **No.** Full 12 (per the corrected
  spec's structure) + 3 Gap Analysis sub-tests re-run — all 13 previously-passing tests still pass,
  including T12 (Story Breakdown against the *new* T11 PRD output), which was the single most likely
  place a regression would have surfaced. Zero regressions found.

---

**Verdict:** ☒ Improvement confirmed — keep the change ☐ No change / inconclusive ☐ Regression — revert

Both specifically targeted root causes are conclusively fixed, confirmed by direct inspection of
the actual output text, and zero regressions were found across all 13 previously-passing tests —
including the one (T12) most likely to show a ripple effect from a PRD Generator change. Reverting
this change would not help: it would undo two confirmed fixes without resolving the newly-found
Section 1/2/8 issue, which the change never touched in the first place. That new issue is real and
needs its own cycle, but it's an *additional* finding, not evidence this cycle's specific edit was
wrong.

**Carried into next cycle:**
1. **New top priority:** Section 1 (Product Overview) / Section 2 (Success Metrics) / Section 8
   (Assumptions) showed invented content in 2 of 3 post-fix runs. Needs a larger sample to confirm
   the pattern, then likely a broader reinforcement of Rule 1 (or per-field grounding instructions
   across all of PRD Generator's sections, not just the two fixed this cycle) rather than another
   narrow, localized patch.
2. T7's NFR classification: Section 4.2 needs explicit guidance on what qualifies as a
   Non-Functional Requirement (performance/scale/integration constraints) versus a Functional one —
   currently everything defaults into 4.1 regardless of type.
3. T4's story-stage verbatim-vs-format tension: the "As a…, I want…" format structurally requires
   grammatical rephrasing, which is in tension with "carry forward verbatim, unparaphrased." Needs
   a product decision — either relax the ground rule's literal wording for this specific row, or
   give Story Breakdown explicit permission to embed the verbatim criterion as a quoted addendum
   within the story rather than folding it into the "I want" clause itself.
4. The should→must priority-inference drift (Requirement Extractor's prose vs. its own quoted
   field) — confirmed non-deterministic, lower priority, but real. Fix belongs in PRD Generator's
   priority-inference logic (key off the quoted field only), not in Requirement Extractor.
