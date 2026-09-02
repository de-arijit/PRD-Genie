# Ground Truth & Baseline Test Report — PRD Genie

*Phase: Baseline (Days 4–7). Maps to reference "Sprint 2: Ground truth, first baseline." Ground truth
itself lives in `ground-rules.md` (already built) — this report is where you run the pipeline against
it and record what actually happened, in the format a TPM would circulate after a first baseline pass.*

---

## Ground truth reference

Scoring criteria: see `ground-rules.md`. Summary of what's being tested:

- 9 tests scoring **Requirement Extraction** (T1, T2, T3, T5, T6, T7, T8, T9, T10)
- 2 tests scoring **PRD Generation** (T4, T11)
- 1 test scoring **Story Breakdown** (T12)
- Cross-cutting: groundedness, UNKNOWN-over-invention, model-tier neutrality (see ground-rules.md)
- Plus 3 **Gap Analysis (extended)** sub-tests against T2/T5/T9's outputs — not part of the numbered
  12, scored separately per ground-rules.md's own instruction.

All 16 agent invocations below ran against `gpt-4o-mini` (OpenAI primary) via the actual n8n node
(`@n8n/n8n-nodes-langchain.openAi` v2.3, `resource: text` / `operation: response`), executed once
each, no retries or prompt edits mid-run. Full raw outputs are preserved in this task's Summary;
this table holds excerpts for readability.

## Baseline run — results

*One row per test. This is the literal table the playbook's Step 5 requires ("5 points specifically for
baseline dataset tested with documented results") — TAs check each row against the expected criteria.*

| Test ID | Input (summary) | Output (summary/excerpt) | Expected keywords present? | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|---|
| T1 | Detailed: filter reports by date/category/status, <2s load, PM Sarah, Q3 deadline | 4 STATED requirements, all 4 facts present verbatim-sourced; 1 UNKNOWN (ownership) | Yes — all 4 | No | **PASS** |
| T2 | Vague: "better reporting… like Competitor X" | 1 STATED + 1 AMBIGUOUS, 1 missing-info item (Competitor X specifics UNKNOWN) | Yes — flags ambiguous, lists missing info | No | **PASS** |
| T3 | Contradictory: auto-refresh 5s vs. minimize API calls | Both sides captured in Contradictions section, neither dropped/resolved | Yes | No | **PASS** |
| T4 | Export PDF+CSV, PDF must include logo, CSV must preserve formulas | PRD generated; FR table's Source column preserved verbatim quotes, but Section 5 Acceptance Criteria **paraphrased** them into fuller sentences instead of verbatim | Partial — facts present, but not verbatim in the AC section itself | No | **FAIL** |
| T5 | Incomplete: dashboard, real-time (vague), budget TBD | 3 AMBIGUOUS items + 3 missing fields (dashboard scope, timeline, ownership), all UNKNOWN, no assumptions filled | Yes | No | **PASS** |
| T6 | Multi-stakeholder: microservices vs. SPA vs. March deadline | All 3 viewpoints captured distinctly, tension identified in Contradictions, none favored/dropped | Yes | No | **PASS** |
| T7 | Technical: 10,000 users, <200ms p95, Salesforce REST API v52 | All 3 numbers/terms preserved exactly, unrounded — but **not classified as NFRs** (Requirement Extractor's output schema has no FR/NFR distinction) | Partial — numbers correct, NFR classification missing | No | **FAIL** |
| T8 | Persona-heavy: Admin, End User, Auditor | 3 distinct personas captured separately, not merged into generic "users" | Yes | No | **PASS** |
| T9 | Empty: "Meeting happened. Notes: none." | "No requirements extractable from this input." — exact match to spec | Yes | No | **PASS** |
| T10 | Dependency: SSO needs Team Alpha's auth service, ETA unknown | SSO feature extracted, Team Alpha dependency flagged, ETA marked UNKNOWN (not invented) | Yes | No | **PASS** |
| T11 | Full PRD from T1's extraction | All 10 sections present, correctly ordered/labeled; but Persona's **"Key Need" field contains an inferred claim** ("efficiently manage project reporting") not traceable to any T1 extraction line | Partial — structure compliant, one field not traceable | **Yes** | **FAIL** |
| T12 | User stories from T11's PRD | 2 stories, exact "As a [persona], I want…, so that…" format, both with priority tags (Must) and FR source IDs; NFRs correctly excluded | Yes | No | **PASS** |

**Baseline pass rate:** 9 / 12
**Baseline hallucination rate:** 1 / 12 (8.3%) — the T11 Persona Key Need field
**Model tier that served each run:** OpenAI (gpt-4o-mini) 12 / 12 · qwen3.5:4b 0 / 12 · mistral 0 / 12
(fallback tiers not wired yet — Loop phase work, per this task's explicit scope)

### Gap Analysis (extended capability) — scored separately, not in the 12

| Sub-test | Input | Output (summary) | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|
| T2 | T2's extractor output (1 AMBIGUOUS item) | 2 concrete, specific questions about the Competitor X comparison | No | **PASS** |
| T5 | T5's extractor output (3 AMBIGUOUS + 3 missing fields) | 5 concrete questions, one per flagged gap (features, timeline, ownership, real-time definition, budget) | No | **PASS** |
| T9 | T9's extractor output ("no requirements extractable") | 1 question asking for the actual meeting notes/transcript | No | **PASS** |

**Gap Analysis pass rate:** 3 / 3, zero hallucinations.

## Notable findings

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

## Handoff to The Loop phase

This baseline is the "before" state for Cycle 1 in `cycle-template.md`. Carry the failing tests and
their root causes forward as that cycle's starting point — don't re-diagnose from scratch on Day 8.

**Open items carried forward:**
1. PRD Generator's Section 5 (Acceptance Criteria) needs a stronger verbatim-preservation
   instruction — Rule 5 already says "preserve exact wording" but T4 shows it isn't holding
   consistently across every section.
2. PRD Generator's Persona "Key Need" field needs the "Not specified in source" fallback enforced
   more strictly — T11 shows it infers a plausible-sounding need instead.
3. Decide whether `ground-rules.md`'s T7 criterion ("classified as NFRs") should move to the PRD
   Generation capability (where NFR classification actually happens), or whether Requirement
   Extractor's prompt should gain a lightweight FR/NFR tag.
4. Decide whether T4's ground-rule text should drop its "User Stories" reference, or whether T4's
   pipeline mapping should be extended through Story Breakdown.
5. Investigate the should→must paraphrase drift starting at the Requirement Extractor stage — not
   a standalone failure yet, but worth closing before it compounds on a harder input.

No failures were fixed as part of this task, per explicit instruction — this report reflects one
clean, unmodified run per test.
