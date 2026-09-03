# Ground Truth & Baseline Test Report — PRD Genie

*Phase: Baseline (Days 4–7). Maps to reference "Sprint 2: Ground truth, first baseline." Ground truth
itself lives in `ground-rules.md` (already built) — this report is where you run the pipeline against
it and record what actually happened, in the format a TPM would circulate after a first baseline pass.*

---

## Ground truth reference

Scoring criteria: see `ground-rules.md`. Summary of what's being tested:

- 9 tests scoring **Requirement Extraction** (T1, T2, T3, T5, T6, T7, T8, T9, T10)
- 2 tests scoring **PRD Generation** (T4's Section 5, T11) plus a **T7 NFR note** (added by Cycle 1,
  see below)
- 2 tests scoring **Epic / User Story Breakdown** (T12, plus a **T4 story-stage row** added by Cycle 1)
- Cross-cutting: groundedness, UNKNOWN-over-invention, model-tier neutrality (see ground-rules.md)
- Plus 3 **Gap Analysis (extended)** sub-tests against T2/T5/T9's outputs — not part of the numbered
  set, scored separately per ground-rules.md's own instruction.

All 16 agent invocations below ran against `gpt-4o-mini` (OpenAI primary) via the actual n8n node
(`@n8n/n8n-nodes-langchain.openAi` v2.3, `resource: text` / `operation: response`), executed once
each, no retries or prompt edits mid-run. Full raw outputs are preserved in this task's Summary;
this table holds excerpts for readability.

> **Updated by Cycle 1** (`evidence/cycles/cycle-01.md`) — the T4, T7, and T11 rows below reflect
> Cycle 1's re-score against a fixed PRD Generator prompt and a corrected `ground-rules.md` (T7's
> NFR clause moved from Requirement Extraction into a new PRD Generation note; T4 split into a PRD
> Generator stage and a new Story-Breakdown stage). **Original baseline results are preserved
> below each updated row, not erased** — see the cycle log for full root-cause analysis, the exact
> prompt diff, and why T11 still fails despite its originally-flagged hallucination being fixed.
>
> **Updated again by Cycle 2** (`evidence/cycles/cycle-02.md`) — a properly-sampled 9-run
> measurement confirmed Cycle 1's Section 1/2/8 finding at 3/9, root-caused, and fixed with one
> consolidated Rule 1 change. **T11 now passes cleanly** — every issue found across both cycles is
> resolved for that test. Separately, `03-story-breakdown.md` was updated per **ADR-006**
> (`docs/02-architecture-decision-record.md`) to resolve the T4 story-stage verbatim-vs-format
> tension; the design is confirmed correct, though T4's story-stage row itself still fails this
> round on a narrower matching-reliability issue (see cycle-02.md's Thread B). Rows below reflect
> Cycle 2's re-score; Cycle 1's numbers are preserved beneath them, not overwritten.
>
> **Updated again by Cycle 3** (`evidence/cycles/cycle-03.md`) — T4's story-stage matching
> reliability is now **fully resolved**: PRD Generator's Section 5 acceptance criteria carry an
> `FR-XXX:` ID prefix, and Story Breakdown matches by exact ID instead of inferring from wording,
> confirmed at 10/10 (100%) across 5 fresh runs plus a final re-verification. T7's NFR classification
> was attempted through 4 escalating fix approaches — all reverted after the last two caused a
> pipeline-breaking refusal regression worse than the original bug — and **remains open**, carried
> forward as a documented gap. This is Baseline phase's closing cycle; see the closing summary at
> the bottom of this report.

## Baseline run — results

*One row per test. This is the literal table the playbook's Step 5 requires ("5 points specifically for
baseline dataset tested with documented results") — TAs check each row against the expected criteria.*

| Test ID | Input (summary) | Output (summary/excerpt) | Expected keywords present? | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|---|
| T1 | Detailed: filter reports by date/category/status, <2s load, PM Sarah, Q3 deadline | 4 STATED requirements, all 4 facts present verbatim-sourced; 1 UNKNOWN (ownership) | Yes — all 4 | No | **PASS** |
| T2 | Vague: "better reporting… like Competitor X" | 1 STATED + 1 AMBIGUOUS, 1 missing-info item (Competitor X specifics UNKNOWN) | Yes — flags ambiguous, lists missing info | No | **PASS** |
| T3 | Contradictory: auto-refresh 5s vs. minimize API calls | Both sides captured in Contradictions section, neither dropped/resolved | Yes | No | **PASS** |
| T4 — Section 5 *(Cycle 3)* | Export PDF+CSV, PDF must include logo, CSV must preserve formulas | Section 5 now carries `FR-XXX:` ID prefixes ahead of each verbatim criterion (e.g. `FR-002: "PDF must include company logo."`) — additive change for Thread A's matching fix, verbatim discipline unaffected and reconfirmed | Yes — verbatim | No | **PASS** |
| ↳ *T4 — Section 5, Cycle 2* | *(same input)* | *3 acceptance criteria byte-for-byte verbatim (fix from Cycle 1 holding). Section 1/2/8 all clean this round* | *Yes — verbatim* | *No* | *PASS* |
| ↳ *T4 Cycle 1* | *(same input)* | *Verbatim criteria confirmed, but Section 1 invented Product Name/Document Version/Status* | *Yes — verbatim* | *Yes* | *PASS* |
| ↳ *T4 baseline (original)* | *(same input)* | *PRD generated; FR table's Source column preserved verbatim quotes, but Section 5 Acceptance Criteria **paraphrased** them into fuller sentences instead of verbatim* | *Partial* | *No* | *FAIL* |
| T4 — Story stage *(Cycle 3)* | T4's PRD → Story Breakdown, exact-ID-match rule | **10/10 (100%) matching accuracy across 5 fresh full-chain runs** plus 1 final re-verification. Every criterion PRD Generator provided in Section 5 was correctly and exclusively attached to its matching FR row's story via exact `FR-XXX:` lookup, regardless of how many of the 3 available criteria Section 5 surfaced that run (1, 2, or 3). Zero false matches, zero cross-contamination, zero silent drops | Yes — all available criteria attached | No | **PASS** *(was FAIL in Cycle 2)* |
| ↳ *T4 story stage, Cycle 2* | *(same input)* | *3 stories generated per ADR-006's format; FR-003 attached correctly, FR-001 correctly had none, but FR-002's available criterion was not attached — a matching-reliability miss* | *No — 1 criterion available but omitted* | *No* | *FAIL* |
| ↳ *T4 story stage, Cycle 1* | *(same input)* | *3 stories generated, each criterion grammatically restructured into "I want" phrasing — not byte-verbatim. Structural tension identified* | *No — paraphrased* | *No* | *FAIL* |
| T5 | Incomplete: dashboard, real-time (vague), budget TBD | 3 AMBIGUOUS items + 3 missing fields (dashboard scope, timeline, ownership), all UNKNOWN, no assumptions filled | Yes | No | **PASS** |
| T6 | Multi-stakeholder: microservices vs. SPA vs. March deadline | All 3 viewpoints captured distinctly, tension identified in Contradictions, none favored/dropped | Yes | No | **PASS** |
| T7 (Requirement Extraction) | Technical: 10,000 users, <200ms p95, Salesforce REST API v52 | All 3 numbers/terms preserved exactly, unrounded — unaffected by either cycle's changes | Yes | No | **PASS** |
| T7 — NFR note *(Cycle 3 — 4 fix attempts made and reverted)* | Same input, scored at PRD Generator's Section 4.2 | Attempted through 4 escalating fixes this cycle (RULES-section Rule 2 alone, Rule 2 + worked example, Rule 2 without example): the last two caused a severe new regression (0/5 combined — the model wrongly refused to generate any PRD at all). All attempts reverted; final prompt is the pre-Rule-2 baseline, confirmed safe (2/2 sanity runs, no refusals). Section 4.2 still reads "Not specified in source" on this specific all-constraints input — the original, known limitation. See cycle-03.md Thread B for the full escalation history | No — classification missing | No | **FAIL** |
| ↳ *T7 — NFR note, Cycle 2* | *(same input)* | *Section 4.2 still reads "Not specified in source" — all 3 constraints still only in Section 4.1. Section 1/2/8 clean this round* | *No — classification missing* | *No* | *FAIL* |
| T8 | Persona-heavy: Admin, End User, Auditor | 3 distinct personas captured separately, not merged into generic "users" | Yes | No | **PASS** |
| T9 | Empty: "Meeting happened. Notes: none." | "No requirements extractable from this input." — exact match to spec | Yes | No | **PASS** |
| T10 | Dependency: SSO needs Team Alpha's auth service, ETA unknown | SSO feature extracted, Team Alpha dependency flagged, ETA marked UNKNOWN (not invented) | Yes | No | **PASS** |
| T11 *(Cycle 3)* | Full PRD from T1's extraction | Re-confirmed clean with Cycle 3's final locked prompt: all 10 sections present/traceable, Persona Key Need correctly "Not specified," Section 1/2/8 clean. **Notably, Section 4.2 correctly classified "Results must load in under 2 seconds" as NFR-001/Performance on this mixed-input case** — evidence the classification gap is worse specifically on all-constraint inputs like T7's, not a total failure (see cycle-03.md Thread B) | Yes — fully traceable | No | **PASS** |
| ↳ *T11, Cycle 2* | *(same input)* | *Fully clean. Section 5 verbatim, Persona Key Need correctly "Not specified," and Section 1/2/8 all clean* | *Yes — fully traceable* | *No* | *PASS* |
| ↳ *T11 Cycle 1* | *(same input)* | *Key Need fixed, but Section 1/2/8 invented content instead* | *Partial* | *Yes* | *FAIL* |
| ↳ *T11 baseline (original)* | *(same input)* | *All 10 sections present, correctly ordered/labeled; but Persona's "Key Need" field contains an inferred claim ("efficiently manage project reporting") not traceable to any T1 extraction line* | *Partial* | *Yes* | *FAIL* |
| T12 *(Cycle 3)* | User stories from T11's PRD (re-run against Cycle 3's freshly-regenerated T11 output) | 1 story (T11's PRD has exactly 1 Functional Requirement row this run — the load-time item correctly classified as NFR, not a story candidate), exact "As a [persona], I want…, so that…" format, priority tag and FR source ID present, Acceptance Criteria line correctly attached via the new exact-ID match. No regression from Thread A's matching-rule change | Yes | No | **PASS** |
| ↳ *T12, Cycle 2* | *(against Cycle 2's T11 output)* | *2 stories, exact format, both with priority tags and FR source IDs; NFRs correctly excluded. Also picked up 2/2 correct verbatim `Acceptance Criteria:` lines per ADR-006* | *Yes* | *No* | *PASS* |

**Cycle 3 pass rate:** 13 / 14 (up from Cycle 2's 12/14 — T4's story-stage row flips to PASS; T7's
NFR note remains the sole open FAIL after 4 reverted fix attempts; see `evidence/cycles/cycle-03.md`)
**Cycle 2 pass rate (for reference):** 12 / 14
**Cycle 1 pass rate (for reference):** 11 / 14
**Original baseline pass rate (for reference):** 9 / 12
**Cycle 3 hallucination rate (this cycle's re-scored items):** 0 / 5 (T4 both stages, T7-extraction,
T7-NFR-note, T11 — zero hallucinations, same as Cycle 2)
**Model tier that served every run:** OpenAI (gpt-4o-mini) — 100%. Fallback tiers not wired yet
(Loop phase work).

### Gap Analysis (extended capability) — scored separately, not in the 12

| Sub-test | Input | Output (summary) | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|
| T2 | T2's extractor output (1 AMBIGUOUS item) | 2 concrete, specific questions about the Competitor X comparison | No | **PASS** |
| T5 | T5's extractor output (3 AMBIGUOUS + 3 missing fields) | 5 concrete questions, one per flagged gap (features, timeline, ownership, real-time definition, budget) | No | **PASS** |
| T9 | T9's extractor output ("no requirements extractable") | 1 question asking for the actual meeting notes/transcript | No | **PASS** |

**Gap Analysis pass rate:** 3 / 3, zero hallucinations.

## Cycle 3 update — notable findings

*(See `evidence/cycles/cycle-03.md` for the full two-thread Run→Score→Learn→Change→Re-score log,
including the complete before/after prompt diffs and every re-score sample.)*

- **T4's story-stage matching is now fully resolved, confirmed at 10/10 (100%).** The root cause was
  genuinely a missing mechanism, not sampling noise: Story Breakdown had no ID-based way to key its
  matching, so it was inferring correspondence from wording similarity every time (Cycle 2's 3/4).
  Fixed at both ends of the PRD-Generator-to-Story-Breakdown handoff — Section 5 now prefixes each
  criterion with its `FR-XXX:` ID, and Story Breakdown's matching rule does an exact-string lookup
  instead of semantic inference. Confirmed across 5 fresh full-chain runs with Section 5 completeness
  varying naturally (1, 2, or 3 of 3 criteria present) plus 1 final re-verification — zero false
  matches, zero cross-contamination, zero silent drops in any run.
- **T7's NFR classification was attempted through 4 escalating fixes and reverted — the failure
  itself is the finding.** RULES-section reinforcement (the exact strategy that fixed Cycle 2's
  Section 1/2/8 groundedness gap) does not automatically transfer to a different *kind* of
  instruction: an active classification/routing decision, not a passive "don't invent" guardrail.
  The first attempt (Rule 2 alone) raised the model's attention to Section 4.2 but produced two
  different failure modes across 2 runs (empty-4.2, then duplicate-classification into both tables).
  The next two attempts (Rule 2 with and without a contrastive worked example) both caused a severe
  regression — 5 of 5 combined runs produced an incorrect "No PRD generated" refusal on input that
  plainly had 3 extractable requirements, a full pipeline break strictly worse than the original
  table-misclassification bug. All changes were reverted; the final prompt matches the pre-Cycle-3
  Section 4.1/4.2 text exactly, confirmed safe via 2 sanity runs plus 5 more incidental executions
  (T4/T11/T12 re-verification) with zero further refusals.
- **A genuine lead surfaced for a future attempt at T7, not chased this cycle:** T11's PRD generation
  (a mixed functional + technical-constraint input) correctly classified its one NFR item into
  Section 4.2 with the *same, unmodified, reverted* prompt — the classification gap appears
  concentrated on inputs like T7's where every extracted item is a technical constraint and none is
  an obviously-functional item to contrast against, rather than being a total, input-independent
  failure. Worth testing a narrower, input-shape-aware fix next time rather than another general
  RULES-section escalation.
- **Zero regressions.** All 9 Requirement Extraction tests and all 3 Gap Analysis sub-tests carry
  forward unchanged (neither prompt was touched this cycle — stated explicitly as the regression-
  check methodology, not silently assumed). T12, which does depend on Story Breakdown's changed
  prompt, was re-run fresh against a newly-generated T11 PRD and held clean.
- **Net result: 13/14, 0 hallucinations across every test scored this cycle** (up from Cycle 2's
  12/14). The one remaining FAIL (T7's NFR note) is not a mystery or a regression risk — it's a
  well-diagnosed, contained, documented limitation with a specific lead for whoever picks it up next.

## Cycle 2 update — notable findings

*(See `evidence/cycles/cycle-02.md` for the full two-thread Run→Score→Learn→Change→Re-score log.)*

- **The Section 1/2/8 pattern was real, not noise — measured at 3/9, exactly consistent, and now
  fixed.** A properly-sized 9-run sample (8 fresh + 1 from Cycle 1 using identical T1 input) found
  invention in exactly 3 runs, always starting with Section 1 (Product Name/Document Version) —
  never scattershot, never a different kind of claim each time. Root cause, confirmed by reading
  the actual prompt rather than guessing: Sections 1/2/8 had zero field-level grounding
  instructions (bare labels/headings), unlike Sections 3/5 which got explicit local reinforcement
  in Cycle 1. Fixed with one consolidated Rule 1 change (per instruction, not three more patches) —
  confirmed clean on all 3 re-scored runs, zero new invention.
- **T11 passes cleanly for the first time**, across the original baseline and both cycles. Every
  issue ever found on this test — Key Need hallucination, then the Section 1/2/8 relocation — is
  now resolved.
- **ADR-006 resolved the T4 story-stage architectural tension, confirmed by design, not just
  intent.** No observed run blended the naturally-paraphrased "As a…" sentence with the verbatim
  criteria line — the separation holds. But per-row *matching* (finding which Section 5 bullet
  corresponds to which FR row) isn't 100% reliable yet: 3 of 4 eligible criteria were correctly
  attached across two observed runs (T4's own re-run: 1/2; T12's regression run: 2/2). T4's
  story-stage row is still scored FAIL this cycle because of that one miss — reported precisely
  rather than rounded up to a clean pass just because the underlying design problem is solved.
- **Zero regressions, again.** All 13 previously-passing tests re-run clean, including T12 against
  a freshly-regenerated T11 PRD — and T12 additionally demonstrated ADR-006's fix working correctly
  on a different input than the one it was designed against.
- **Net result: 12/14, 0 hallucinations in this cycle's re-scored items** (down from Cycle 1's
  2/4) — both cycles' fixes are holding simultaneously, nothing needed to be traded off.

## Cycle 1 update — notable findings

*(See `evidence/cycles/cycle-01.md` for the full Run→Score→Learn→Change→Re-score log. Summary here.)*

- **Both targeted root causes are conclusively fixed.** T4's Section 5 now preserves acceptance
  criteria byte-for-byte verbatim (confirmed by direct inspection, not just re-reading the prompt),
  and T11's Persona Key Need field correctly falls back to "Not specified in source" instead of
  inventing a plausible-sounding need.
- **A new hallucination pattern surfaced in the same runs where the fixes were confirmed working:**
  2 of 3 post-fix PRD Generator runs invented content in Section 1 (Product Overview), Section 2
  (Success Metrics), or Section 8 (Assumptions) — sections this cycle's edit never touched. This is
  now the **top open item** for Cycle 2, likely bigger in scope than either of the two issues this
  cycle fixed. Not attributable with confidence to this cycle's specific edit (the one PRD Generator
  run that *didn't* touch Section 5/Persona-relevant content, T7, showed no such invention) — most
  likely pre-existing model variance that a small "before" sample simply didn't happen to surface.
- **Zero regressions found.** All 13 previously-passing tests (9 Requirement Extraction + T12 +
  3 Gap Analysis) were re-run after the change and still pass — including T12, re-run against the
  *new* T11 PRD output, the single most likely place a PRD Generator change could have rippled into.
- **T11's specific baseline hallucination was not 100% reproducible** — re-running it once before
  making any change (to confirm reproducibility, as instructed) showed it did *not* fire that time.
  The underlying prompt gap was still real and worth closing regardless.
- **T4's new story-stage test revealed a structural tension in the corrected spec itself**: the "As
  a…, I want…" story format requires grammatically rephrasing a declarative criterion into a
  first-person want-statement, which is in real tension with "carry forward verbatim, unparaphrased."
  Flagged for a future cycle's product decision, not fixed here (Story Breakdown is out of scope
  this cycle).
- **T7's NFR classification remains unaddressed**, correctly, per this cycle's explicit scope —
  PRD Generator's Section 4.2 still has no guidance on what qualifies as a Non-Functional
  Requirement, so all 3 technical constraints still land only in Section 4.1.

## Original baseline notable findings (preserved for history)

- **Strongest capability: Requirement Extraction (8/9) and Gap Analysis (3/3), with zero
  hallucinations across all 12 of those invocations.** Every failure and the one hallucination in
  this baseline concentrate specifically in PRD Generation.
- **Weakest capability: PRD Generation (0/2).** Both T4 and T11 failed, for two different but
  related reasons — T4's Acceptance Criteria section paraphrased away from verbatim (even though
  the same facts stayed verbatim in the FR table's Source column, proving the model *can* preserve
  exact wording when explicitly told to — it just didn't apply that discipline consistently across
  every section), and T11's Persona "Key Need" field filled in a plausible-sounding inference
  instead of "Not specified in source." Structural compliance (10 sections, correct order, correct
  "Not specified" fallback used everywhere else) was otherwise solid in both runs — this looks like
  a precision/groundedness gap in specific sub-sections, not a wholesale prompt failure.
- **The persona-string fix from this task's Step 1 resolved cleanly.** T11's PRD renders
  `**Persona:** Sarah — Project Manager` (not the old `Sarah/Project Manager` bug), and T12's Story
  Breakdown correctly derived `As a Sarah, I want…` from it with no artifact. Confirmed by direct
  inspection of both outputs, not just re-reading the prompt.
- **A subtle cross-agent error-propagation pattern, worth flagging even though it didn't cause a
  standalone test failure:** T1's Requirement Extractor restated the filter requirement as "Users
  **must** be able to filter…" in its own prose, even though its own verbatim `source:` quote
  correctly preserved the original "**should**." PRD Generator then keyed its priority assignment
  for that row off the paraphrased "must" rather than the more authoritative quoted "should,"
  marking it `Must` instead of `Should`. T12 then faithfully carried that `Must` forward (correctly,
  per its own rules — the drift originated upstream, not in Story Breakdown). This is exactly the
  kind of small compounding drift the Loop phase should catch before it matters on a less
  forgiving input.
- **Two apparent mismatches between `ground-rules.md`'s literal wording and this task's own
  pipeline-stage scoping**, surfaced by actually running the tests rather than assumed in advance:
  - T7's PASS bar includes "classified as NFRs," but T7 is scored on Requirement Extractor's output
    alone (per this task's explicit pipeline mapping), and that agent's output format has no FR/NFR
    concept at all — NFR classification only happens later, in PRD Generator's Section 4.2.
  - T4's PASS bar mentions "User Stories generated with exactly these criteria," but T4's pipeline
    mapping stops at PRD Generator — Story Breakdown was never run against T4's output.
  - Recorded here rather than silently reinterpreted; worth a decision on whether to adjust the
    ground rule's wording or the pipeline's scope for these two tests.
- **Which tests needed a second look:** T4, T7, T11 (the 3 failures, naturally), plus T3 — it
  passed, but treated one side of the contradiction as STATED and the other as AMBIGUOUS rather
  than presenting both with equal footing, which is worth a manual re-read even though it didn't
  cross the FAIL line.

## Handoff — status after Cycle 3 (Baseline phase close-out)

This baseline was the "before" state for Cycle 1, which fed Cycle 2, which fed Cycle 3
(`evidence/cycles/cycle-01.md`, `cycle-02.md`, `cycle-03.md`). Items below are marked
resolved/still-open based on each cycle's actual re-score results, not assumed.

**Resolved across Cycles 1–3:**
1. ~~PRD Generator's Section 5 verbatim-preservation~~ — **RESOLVED** (Cycle 1, held through Cycles 2-3).
2. ~~PRD Generator's Persona "Key Need" grounding~~ — **RESOLVED** (Cycle 1, held through Cycles 2-3).
3. ~~T7's "classified as NFRs" ground-rule placement~~ — **RESOLVED** (spec corrected, Cycle 1).
4. ~~T4's "User Stories" ground-rule reference~~ — **RESOLVED** (spec corrected, Cycle 1).
5. ~~Section 1 (Product Overview) / Section 2 (Success Metrics) / Section 8 (Assumptions) invented
   content~~ — **RESOLVED** (Cycle 2). Measured at 3/9 on a proper sample, root-caused, fixed with
   one consolidated Rule 1 change, confirmed clean on 3/3 re-scored runs plus zero regressions,
   held through Cycle 3.
6. ~~T4 story-stage verbatim-vs-"As a…, I want…"-format architectural tension~~ — **RESOLVED by
   design** (Cycle 2, ADR-006: a separate verbatim `Acceptance Criteria:` line, kept apart from the
   naturally-paraphrased story sentence). Confirmed no sentence/quote blending in any observed run.
7. ~~T4's story-stage matching reliability~~ — **RESOLVED** (Cycle 3). Root cause: no ID-based
   matching mechanism existed. Fixed with an `FR-XXX:` ID prefix in PRD Generator's Section 5 plus
   an exact-ID-lookup rule in Story Breakdown. Confirmed 10/10 (100%) across 5 fresh runs plus 1
   final re-verification — zero false matches, zero cross-contamination, zero silent drops.

**Still open:**
8. **PRD Generator's Section 4.2 NFR classification** — unaddressed and, as of Cycle 3, actively
   attempted-and-reverted. Four escalating fix approaches across Cycle 3 alone (on top of being
   correctly out of scope in Cycles 1-2) either failed to change behavior or caused a worse
   regression (a pipeline-breaking refusal). Reverted to the known baseline limitation rather than
   ship a fix that intermittently breaks PRD generation entirely. **Specific lead for next attempt:**
   the gap appears concentrated on inputs where every extracted item is a technical constraint (T7's
   shape) — T11's mixed input classified correctly with the same unmodified prompt. A narrower,
   input-shape-aware approach is worth trying before another general RULES-section escalation.
9. Should→must priority-inference drift — still open, confirmed non-deterministic. Low priority
   relative to item 8.
10. Not yet checked: whether PRD Generator's Sections 6, 7, 9, 10 share the same "bare heading, no
    local reinforcement" shape that caused the Section 1/2/8 issue — still not part of any cycle's
    measurement scope. Worth a look, not urgent (no evidence of an actual problem there yet).
11. PRD Generator's own Section 5 *completeness* (independent of Story Breakdown's now-100%-reliable
    matching) varies run to run — observed writing 1, 2, or 3 of 3 available criteria across Cycle
    3's 5-run sample. Not yet measured as its own problem; worth watching.

Three prompts were changed total across all three cycles — PRD Generator (Cycles 1, 2, and 3, though
Cycle 3's Section 4.2 changes were fully reverted) and Story Breakdown (Cycle 2 for ADR-006, Cycle 3
for the FR-ID matching rule). Requirement Extractor and Gap Analyzer remain untouched since the
baseline run, across all three cycles.

---

## Baseline phase — closing summary

**Total tests:** 14 numbered-capability rows (9 Requirement Extraction + T4-Section-5 + T4-Story-
stage + T7-NFR-note + T11 + T12) plus 3 Gap Analysis (extended-capability) sub-tests, scored
separately per `ground-rules.md`'s own instruction — 17 total scored items.

**Final pass rate:** **13 / 14** numbered tests (93%) · **3 / 3** Gap Analysis (100%) · **0
hallucinations** across every test scored in Cycle 3, and 0 across every test re-scored in Cycle 2.
The single remaining FAIL is T7's NFR classification note — a known, well-diagnosed, contained
limitation, not an unexplained gap.

**Three-cycle history, in one paragraph:** Baseline phase started at 9/12 (the original 12-test set,
before ground-rules.md's own T4/T7 corrections split those two tests into two-stage rows each) with
PRD Generation as the clear weak point — both T4's verbatim-acceptance-criteria discipline and T11's
persona-need grounding failed outright. Cycle 1 fixed both root causes directly (verbatim copying,
"Not specified" fallback for Key Need) but its own re-testing surfaced a new pattern — Sections 1/2/8
inventing administrative-sounding boilerplate — pushing the corrected-spec pass rate to 11/14. Cycle
2 measured that pattern properly (a real 3/9, not noise), fixed it with one consolidated groundedness
rule, and separately resolved a genuine architectural tension in the story-format spec via ADR-006,
reaching 12/14 with T11 passing cleanly for the first time and T4's story stage narrowed down to a
matching-reliability gap. Cycle 3 closed that gap completely (T4's story stage now 100% reliable via
deterministic FR-ID matching) but spent most of its effort on T7's NFR classification, which resisted
four escalating fix attempts — the last two of which made things actively worse (a pipeline-breaking
refusal regression) before being caught and reverted — landing Baseline phase at its final 13/14,
with T7 carried forward as an explicitly scoped, well-understood open item rather than a surprise.

**Is Baseline phase complete, and is the pipeline ready for Loop-phase infrastructure work?**
**Yes — ready, with one known item carried into the Loop-phase backlog, not silently dropped.**
13 of 14 numbered tests pass with zero hallucinations across three full cycles of iteration; the one
open item (T7's NFR classification) is narrow in scope (it affects only how Section 4.2 gets
populated, not whether the pipeline produces a valid, grounded PRD — the reverted-to baseline still
generates correct output, just without the NFR/FR split on all-constraint inputs), has a specific,
documented lead for whoever picks it up next, and — critically — is proven *safe*: three cycles of
testing, including this cycle's aggressive escalation attempts, never found a variant of this bug
that produced a hallucination or an ungrounded claim, only a classification gap or (during the
reverted attempts) an overly-cautious refusal. Nothing about T7's open status blocks Loop-phase
infrastructure work (fallback routing, eval service, live dashboard) — that work operates on top of
the pipeline's existing behavior regardless of whether Section 4.2 is populated, and a documented,
bounded gap is exactly the kind of item a live dashboard and ongoing eval-service tracking are built
to surface and monitor going forward, rather than something that needs to block infrastructure work
to fix in isolation first.
