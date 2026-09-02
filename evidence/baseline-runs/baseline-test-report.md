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

## Baseline run — results

*One row per test. This is the literal table the playbook's Step 5 requires ("5 points specifically for
baseline dataset tested with documented results") — TAs check each row against the expected criteria.*

| Test ID | Input (summary) | Output (summary/excerpt) | Expected keywords present? | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|---|
| T1 | Detailed: filter reports by date/category/status, <2s load, PM Sarah, Q3 deadline | 4 STATED requirements, all 4 facts present verbatim-sourced; 1 UNKNOWN (ownership) | Yes — all 4 | No | **PASS** |
| T2 | Vague: "better reporting… like Competitor X" | 1 STATED + 1 AMBIGUOUS, 1 missing-info item (Competitor X specifics UNKNOWN) | Yes — flags ambiguous, lists missing info | No | **PASS** |
| T3 | Contradictory: auto-refresh 5s vs. minimize API calls | Both sides captured in Contradictions section, neither dropped/resolved | Yes | No | **PASS** |
| T4 — Section 5 *(Cycle 1)* | Export PDF+CSV, PDF must include logo, CSV must preserve formulas | All 3 acceptance criteria now byte-for-byte verbatim quotes — the fix worked. But Section 1 (Product Overview) now contains **invented** values ("Product Name: Report Export Feature", "Document Version: 1.0", "Status: Draft") never present in any source | Yes — verbatim now | **Yes** (new — Section 1, unrelated to the fixed field) | **PASS** |
| ↳ *T4 baseline (original)* | *(same input)* | *PRD generated; FR table's Source column preserved verbatim quotes, but Section 5 Acceptance Criteria **paraphrased** them into fuller sentences instead of verbatim* | *Partial* | *No* | *FAIL* |
| T4 — Story stage *(Cycle 1, new row)* | T4's PRD → Story Breakdown | 3 stories generated (one per FR row), but each criterion was grammatically restructured into "I want [goal]" phrasing — not byte-verbatim. Structural tension: the "As a…, I want…" format inherently requires rephrasing a declarative criterion into a first-person want-statement | No — paraphrased in story form | No | **FAIL** |
| T5 | Incomplete: dashboard, real-time (vague), budget TBD | 3 AMBIGUOUS items + 3 missing fields (dashboard scope, timeline, ownership), all UNKNOWN, no assumptions filled | Yes | No | **PASS** |
| T6 | Multi-stakeholder: microservices vs. SPA vs. March deadline | All 3 viewpoints captured distinctly, tension identified in Contradictions, none favored/dropped | Yes | No | **PASS** |
| T7 (Requirement Extraction) | Technical: 10,000 users, <200ms p95, Salesforce REST API v52 | All 3 numbers/terms preserved exactly, unrounded (this half of the original T7 row is unchanged and still passes under the corrected spec) | Yes | No | **PASS** |
| T7 — NFR note *(Cycle 1, new row, first attempt)* | Same input, scored at PRD Generator's Section 4.2 | Section 4.2 (Non-Functional Requirements) reads "Not specified in source" — all 3 constraints landed only in Section 4.1 as Functional Requirements. Never classified as NFRs. Not fixed this cycle (out of scope — see cycle-01.md) | No — classification missing | No | **FAIL** |
| T8 | Persona-heavy: Admin, End User, Auditor | 3 distinct personas captured separately, not merged into generic "users" | Yes | No | **PASS** |
| T9 | Empty: "Meeting happened. Notes: none." | "No requirements extractable from this input." — exact match to spec | Yes | No | **PASS** |
| T10 | Dependency: SSO needs Team Alpha's auth service, ETA unknown | SSO feature extracted, Team Alpha dependency flagged, ETA marked UNKNOWN (not invented) | Yes | No | **PASS** |
| T11 *(Cycle 1)* | Full PRD from T1's extraction | Persona **Key Need now correctly reads "Not specified in source"** — the originally-flagged hallucination is fixed. But new invented content appeared in Section 1 (Product Overview), Section 2 (Success Metrics), and Section 8 (Assumptions) — none traceable to T1's extraction. Still trips T11's own "fills a section with generic filler" FAIL condition, just via different sections | Partial — structure compliant, 3 different fields not traceable | **Yes** (relocated, not resolved) | **FAIL** |
| ↳ *T11 baseline (original)* | *(same input)* | *All 10 sections present, correctly ordered/labeled; but Persona's "Key Need" field contains an inferred claim ("efficiently manage project reporting") not traceable to any T1 extraction line* | *Partial* | *Yes* | *FAIL* |
| T12 | User stories from T11's PRD (re-run against Cycle 1's new T11 output as a regression check) | 2 stories, exact "As a [persona], I want…, so that…" format, both with priority tags and FR source IDs; NFRs correctly excluded. No regression from the PRD Generator change | Yes | No | **PASS** |

**Cycle 1 pass rate:** 11 / 14 (corrected-spec structure — T4 and T7 now each span two rows; see
`evidence/cycles/cycle-01.md` for the full before/after breakdown and grand total)
**Original baseline pass rate (for reference):** 9 / 12
**Cycle 1 hallucination rate:** 2 / 14 (14.3%) — both from the new Section 1/2/8 issue (T4-Section5,
T11), not from the fields Cycle 1 specifically targeted
**Model tier that served every run:** OpenAI (gpt-4o-mini) — 100%. Fallback tiers not wired yet
(Loop phase work).

### Gap Analysis (extended capability) — scored separately, not in the 12

| Sub-test | Input | Output (summary) | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|
| T2 | T2's extractor output (1 AMBIGUOUS item) | 2 concrete, specific questions about the Competitor X comparison | No | **PASS** |
| T5 | T5's extractor output (3 AMBIGUOUS + 3 missing fields) | 5 concrete questions, one per flagged gap (features, timeline, ownership, real-time definition, budget) | No | **PASS** |
| T9 | T9's extractor output ("no requirements extractable") | 1 question asking for the actual meeting notes/transcript | No | **PASS** |

**Gap Analysis pass rate:** 3 / 3, zero hallucinations.

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

## Handoff — status after Cycle 1

This baseline was the "before" state for Cycle 1 (`evidence/cycles/cycle-01.md`). Original open
items below are marked resolved/still-open based on that cycle's actual re-score results, not
assumed.

**Original open items — resolved or updated by Cycle 1:**
1. ~~PRD Generator's Section 5 verbatim-preservation~~ — **RESOLVED.** Confirmed fixed by direct
   re-score (T4-Section5 now PASS).
2. ~~PRD Generator's Persona "Key Need" grounding~~ — **RESOLVED for the specific field.** Confirmed
   fixed by direct re-score. T11 as a whole still fails, but for a different, newly-surfaced reason
   (see item 6 below) — not because this fix didn't work.
3. T7's "classified as NFRs" ground-rule placement — **RESOLVED** (spec corrected: moved to a new
   PRD Generation note, scored against Section 4.2). The underlying *behavior* (PRD Generator
   doesn't actually classify anything as an NFR) is **still open** — carried forward as item 7.
4. T4's "User Stories" ground-rule reference — **RESOLVED** (spec corrected: T4 now explicitly
   splits into a PRD Generator stage and a separate Story-Breakdown stage).
5. Should→must priority-inference drift — **still open**, confirmed non-deterministic this cycle.
   Not fixed (out of scope — the fix belongs in PRD Generator's priority-inference logic, not
   Requirement Extractor, since Requirement Extractor's own quoted field is already correct).

**New items carried into Cycle 2:**
6. **(Top priority)** Section 1 (Product Overview) / Section 2 (Success Metrics) / Section 8
   (Assumptions) invented content in 2 of 3 post-fix PRD Generator runs — needs a larger sample to
   confirm the pattern, then likely a broader groundedness reinforcement across all sections, not
   another narrow field-level patch.
7. PRD Generator's Section 4.2 needs explicit guidance on what qualifies as a Non-Functional
   Requirement — currently everything defaults into Section 4.1 regardless of type.
8. T4's story-stage verbatim-vs-"As a…, I want…"-format tension needs a product decision: relax the
   ground rule's literal wording for that row, or let Story Breakdown embed the verbatim criterion
   as a quoted addendum rather than folding it into the want-clause itself.

One prompt was changed this cycle (PRD Generator only), scoped exactly as instructed — Requirement
Extractor, Story Breakdown, and Gap Analyzer remain untouched since the baseline run.
