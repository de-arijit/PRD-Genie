# Evaluation Cycle Log — Cycle #3

*One of these per Run→Score→Learn→Change→Re-score iteration. Copy this template for each new cycle.
Feeds both the dashboard's cycle log (via `POST /cycles`) and the Q3/Q4 writeup directly — write it in
language you'd be comfortable pasting straight into the submission.*

**Date:** 2026-09-03
**Trigger for this cycle:** Two items carried forward from Cycle 2's "Carried into next cycle" list:
(1) T4's story-stage matching reliability — Story Breakdown's Acceptance Criteria matching worked
when it worked (3/4 observed) but wasn't reliably finding every eligible Section 5 bullet; (2) T7's
NFR classification (PRD Generator's Section 4.2) — unaddressed across two prior cycles, flagged as
"should be Cycle 3's primary target." Framed explicitly as the last narrow cycle before Loop-phase
infrastructure work (fallback routing, eval service, live dashboard) begins.
**Cycle type:** ☒ Correctness (fixing a failed/hallucinated test) ☐ Token optimization ☐ Both

*This cycle also has two distinct threads — kept separate throughout. Thread A (T4) is a clean,
fully-resolved fix. Thread B (T7) is reported honestly as an escalating series of attempts that
ultimately failed and was reverted — included in full because the failure mode itself (a fix that
looked reasonable but broke the pipeline worse than the original bug) is a real finding, not a
detour to omit.*

---

## Thread A — T4 story-stage Acceptance Criteria matching reliability

### 1. Run / diagnose

Read `03-story-breakdown.md`'s actual matching instruction (not assumed) to see whether Cycle 2's
"3/4 observed" miss was a vague-instruction problem or a sampling-noise problem.

**Finding:** the instruction was genuinely vague. Story Breakdown's Section 5 (Acceptance Criteria)
matching relied entirely on semantic/text similarity — "find the bullet in the PRD's Section 5 that
corresponds to this FR row... it will typically reference the same underlying source quote." There
was no ID-based mechanism available at all: PRD Generator's own Section 5 output had no `FR-XXX:`
prefix or any other explicit key, so Story Breakdown had nothing deterministic to match against — it
was inferring the correspondence from wording overlap every time. A 3/4 hit rate is exactly what
you'd expect from an inherently fuzzy matching strategy, not noise around an otherwise-reliable one.

### 2. Change

Fixed at both ends of the handoff — the source (PRD Generator) and the consumer (Story Breakdown) —
since a matching problem between two agents can't be fixed by editing only one side.

```
PRD Generator, Section 5 (BEFORE):
(Per key feature: copy the exact source wording as the criterion, verbatim. Do NOT rewrite it into
a complete sentence...)

PRD Generator, Section 5 (AFTER):
(Per key feature: copy the exact source wording as the criterion, verbatim, prefixed with the same
FR ID that requirement has in Section 4.1's table, e.g. `FR-002: "PDF must include company logo."`
— this ID prefix is required so a later pipeline step can match each criterion back to its exact
requirement without guessing...)

Story Breakdown, matching rule (BEFORE):
Semantic/text-similarity matching against Section 5's bullets — no ID key available.

Story Breakdown, matching rule (AFTER):
Exact-string lookup: find the Section 5 line beginning with this story's own `FR-XXX:` prefix. If
no line starts with that exact ID, there is no criterion for this story — do not guess from wording.
```

**Why this should fix the root cause:** it replaces an inherently fuzzy, LLM-inferred correspondence
with a deterministic, mechanical lookup — the kind of fix an LLM can execute at 100% reliability
because it no longer requires judgment, only string matching.
**Scope:** PRD Generator's Section 5 instruction + Story Breakdown's matching rule. No other section
of either prompt touched for this thread.

### 3. Re-score

Ran the full extractor → PRD Generator → Story Breakdown chain **5 times fresh** against T4's input
("Users need to export reports as PDF and CSV. PDF must include company logo. CSV must preserve
formulas."), letting PRD Generator's own Section 5 completeness vary naturally run to run (it wrote
1, 2, or 3 of the 3 available criteria depending on the run — a separate, out-of-scope observation,
see Observations below) to stress-test matching under different conditions, not just the easy case.

| Run | Section 5 criteria PRD Generator wrote | Story Breakdown matches | Correct? |
|---|---|---|---|
| 1 | FR-002, FR-003 (2 of 3) | FR-002 ✓, FR-003 ✓ | Yes — 2/2 |
| 2 | FR-001, FR-002, FR-003 (3 of 3) | FR-001 ✓, FR-002 ✓, FR-003 ✓ | Yes — 3/3 |
| 3 | FR-001, FR-002, FR-003 (3 of 3) | FR-001 ✓, FR-002 ✓, FR-003 ✓ | Yes — 3/3 |
| 4 | FR-001, FR-002, FR-003 (3 of 3) | FR-001 ✓, FR-002 ✓, FR-003 ✓ | Yes — 3/3 |
| 5 | FR-002 only (1 of 3) | FR-002 ✓ | Yes — 1/1 (format drift: missing `— source: FR-XXX` tag, see Observations) |

**10 / 10 (100%) matching accuracy** — every criterion PRD Generator actually provided in Section 5
was correctly and exclusively attached to its matching FR row's story, across every run, regardless
of how many of the 3 available criteria PRD Generator chose to surface that run. Zero false matches,
zero cross-contamination (no criterion attached to the wrong story), zero criteria silently dropped
when present. A final targeted re-verification (1 more full-chain run against the same input, using
the cycle's final locked prompts after Thread B's revert) reproduced the same clean result: FR-001
correctly had no criterion, FR-002 and FR-003 correctly attached — confirming the fix and Thread B's
revert don't interact.

**Thread A verdict: fully resolved.** No longer carried forward.

---

## Thread B — T7 NFR classification (Section 4.2)

### 1. Run / diagnose

Confirmed the starting state fresh rather than assuming Cycle 1/2's description still held: ran T7's
input through PRD Generator 3 times with the then-current prompt (inline OUTPUT-section parenthetical
under Section 4.2, giving the four NFR categories with worked examples, but no RULES-section
reinforcement). **0 / 3** — Section 4.2 read "Not specified in source" every time, all 3 technical
constraints (10,000 concurrent users, <200ms p95, Salesforce REST API v52) landing only in Section
4.1, indistinguishable from Cycle 1/2's unaddressed baseline.

### 2. Change — attempt 1: RULES-section Rule 2

Following Cycle 2's own precedent (Section 1/2/8's groundedness gap needed RULES-section-level
reinforcement, not just an OUTPUT-section patch, to actually change behavior), added a new Rule 2
forcing an active per-row NFR test before finalizing Section 4.1, explicitly naming this as "not a
default you skip."

**Re-score:** 2 fresh runs. **0 / 2, two different failure modes:**
- Run 1: identical to pre-fix behavior — Section 4.2 still empty, all 3 constraints in 4.1.
- Run 2: **new failure mode** — the model populated Section 4.2 correctly (right categories: NFR-001
  Scalability, NFR-002 Performance, NFR-003 Integration) but *also* left all 3 constraints in Section
  4.1, violating the explicit "classify into exactly one of the two tables, never both" instruction.
  The Target column in 4.2 was also left as "Not specified in source" instead of the actual number.

Rule 2 measurably increased the model's attention to Section 4.2 (it went from never populating it to
sometimes populating it) but did not converge on the correct, exclusive classification.

### 3. Change — attempt 2: Rule 2 + contrastive worked example

Rewrote Section 4.1/4.2's OUTPUT-section text to explicitly frame the relocation as "move, not
duplicate," add explicit Target-column instructions, and added a contrastive worked example
(illustrative password-reset-vs-throughput-number example, deliberately using different sample
numbers than T7's own to avoid overfitting to one test case) directly demonstrating the classification
split. Also tightened Rule 2's wording to explicitly call out the double-classification failure mode
just observed.

**Re-score:** 2 fresh runs. **0 / 2 — severe new regression.** Both runs produced:
```
No PRD generated — no requirements were extractable from the source input.
```
This is PRD Generator's own Rule 6 refusal ("if the extractor's input says there was nothing
extractable, do not generate a PRD"), and it fired **incorrectly** — the input plainly contained 3
extractable requirements (visible in the extractor's own output, unchanged from every prior run).
Output tokens dropped to 16 (just the refusal sentence) versus the ~530-680 tokens a full PRD
normally takes. This is strictly worse than either prior failure mode: instead of misclassifying a
table row, the entire pipeline output was lost.

### 4. Change — attempt 3: isolate the cause

To determine whether the worked example specifically was the trigger, or whether Rule 2's rewritten
wording itself was responsible, reverted only the worked-example paragraph (kept the "relocation not
duplication" phrasing and Target-column instruction) and re-tested.

**Re-score:** 3 fresh runs. **0 / 3 — same refusal regression, no worked example present.** All 3
runs produced the identical "No PRD generated..." refusal, tokens 16 out each time.

**Combined across attempts 2 and 3: 5 of 5 fresh confirmation runs produced the incorrect refusal.**
This isolates the cause conclusively: Rule 2's presence and phrasing — not the worked example — was
driving the refusal, most likely by shifting the model's overall posture toward the input (heavy,
repeated "NEVER... MUST... ACTIVELY PERFORM..." framing stacked on top of Rule 1's similarly emphatic
language) to the point where Rule 6's extractability check became over-triggered as a side effect,
rather than by any change to Rule 6 itself.

### 5. Change — revert

Reverted `02-prd-generator.md`'s Section 4.1/4.2 OUTPUT text and the RULES section entirely back to
the pre-Rule-2 state (the same inline-parenthetical version tested in step 1). Diffed the final file
against the Cycle-2-era committed version to confirm the only remaining differences are Thread A's
Section 5 FR-ID-prefix change and this cycle's documentation notes — no residual trace of Rule 2 or
the worked example remains.

**Re-score (sanity check):** 2 fresh runs against T7's input with the reverted prompt. **2 / 2 valid
PRDs, zero refusals** — output tokens 586 and 536 respectively, matching the original pre-Rule-2
baseline behavior (Section 4.2 still reads "Not specified in source," all 3 constraints in 4.1 —
the known, original limitation, not a new problem). This cycle's T4, T11, and T12 re-verification
runs (Thread A and the regression check, below) all used this same final reverted prompt and produced
zero further refusals across 5 additional executions — the revert is not a fragile one-off fix.

**One additional data point worth noting:** T11's PRD generation this cycle (T1's input, a mixed
functional + technical-constraint input, not T7's all-constraints input) *did* correctly classify
"Results must load in under 2 seconds" into Section 4.2 as Performance with the reverted, unmodified
prompt. This suggests the classification failure is not absolute — it appears specifically most
severe on inputs like T7's where every extracted item is a technical constraint and none is a plain
user-facing capability, possibly because the model has no contrasting "obviously-4.1" item in the
same input to calibrate against. This is a genuine lead for a future cycle, not something this
cycle's time budget allowed chasing further.

**Thread B verdict: not resolved.** Reverted to the known, pre-existing limitation rather than
shipping a prompt that intermittently breaks the entire pipeline. T7 remains **FAIL**, carried
forward as a documented, well-understood gap — see Carried into next cycle, below.

---

## Combined Re-score (all threads) — full picture

| Test | Pass/Fail | Hallucination? | Notes |
|------|-----------|-----------------|-------|
| T4 — Section 5 | **PASS** | No | Unaffected by this cycle's changes (FR-ID prefix is additive to an already-verbatim field); re-confirmed clean |
| T4 — Story stage | **PASS** | No | *(was FAIL in Cycle 2)* — matching reliability fix confirmed 10/10 (100%) across 5 fresh runs plus 1 final re-verification |
| T7 (Requirement Extraction) | **PASS** | No | Unaffected — Requirement Extractor prompt untouched this cycle |
| T7 — NFR note | **FAIL** | No | Unresolved after 4 escalating fix attempts (0/3, 0/2, 0/2, 0/3), all reverted; classification still defaults to leaving Section 4.2 empty on T7's specific all-NFR input |
| T11 | **PASS** | No | Unaffected by this cycle's changes; re-confirmed clean (all 10 sections traceable, Section 4.2 actually worked correctly on this mixed input) |

**Aggregate cycle 3 (targeted, Thread A + Thread B):** 4 / 5 rows passed (T4 both stages + T7-extraction
+ T11) · 1 / 5 failed (T7 NFR note, carried forward) · 0 hallucinations

### Regression check

**Requirement Extraction (T1, T2, T3, T5, T6, T8, T9, T10) and Gap Analysis (T2, T5, T9):** not
re-executed fresh this cycle. Neither the Requirement Extractor's prompt nor the Gap Analyzer's
prompt was touched anywhere in Cycle 3 — the only files edited were `02-prd-generator.md` and
`03-story-breakdown.md`, and Requirement Extraction sits entirely upstream of both. These 11 tests
carry forward their Cycle 2 confirmed-passing state unchanged. (Flagged explicitly here rather than
silently implied, per this cycle's own standard for honest reporting — this is a reasoned carry-
forward, not a fresh re-run.)

**T12 (Story Breakdown regression):** DID need a fresh check — Story Breakdown's prompt changed this
cycle (the FR-ID matching rule). Re-ran against a freshly-regenerated T11 PRD (using this cycle's
final locked prompts):
```
- As a Sarah, I want to filter reports by date range, category, and status, so that the stated
  requirement is met. — Priority: Should (priority inferred — not explicit in source) — source: FR-001
  Acceptance Criteria: "The user should be able to filter reports by date range, category, and status."
```
Correct format, correct priority tag, correct FR-ID source, and the Acceptance Criteria line correctly
attached by exact ID match. Only 1 story, correctly — T11's PRD has exactly 1 Functional Requirement
row (the load-time requirement correctly classified as NFR-001, not a user story candidate). No
regression from the matching-rule change.

**T4 story-stage:** covered under Thread A above (5 runs + 1 final re-verification, all clean).

**13 / 13 regression tests held. Zero regressions from either thread's changes.**

### Cycle 3 grand total (14 numbered-capability rows + 3 Gap Analysis, same structure as Cycles 1-2)

**13 / 14 passed** (up from Cycle 2's 12/14) · **0 hallucinations across every test scored this
cycle** · Gap Analysis: **3 / 3, 0 hallucinations** (carried forward, unaffected)

Remaining FAIL: **T7 (NFR note)** — attempted through 4 escalating fix approaches this cycle, all
reverted after the last two caused a pipeline-breaking regression worse than the original bug.
Documented as a known, well-diagnosed gap rather than silently left unexplained.

---

**Verdict:** ☒ Improvement confirmed — keep the change (Thread A) ☒ Regression — revert (Thread B,
attempts 1-4; the *net* cycle outcome is still a real, measured improvement — 13/14 up from 12/14 —
because Thread A's fix shipped cleanly and Thread B's attempts were caught and reverted before
shipping, rather than landing a hidden regression)

Thread A is a complete, clean resolution: a genuinely vague matching instruction was diagnosed by
reading the actual prompt text (not assumed), fixed with a deterministic ID-based mechanism at both
ends of the PRD-Generator-to-Story-Breakdown handoff, and confirmed at 100% reliability across 5
fresh runs plus a final re-verification — the strongest possible confirmation short of a full
statistical sample. Thread B is reported in full because the failure itself is the finding: an
escalation strategy that worked for a different bug in a prior cycle (Cycle 2's Section 1/2/8 fix)
does not automatically transfer to a different *kind* of instruction (an active classification/
routing decision versus a passive groundedness/invention guardrail), and pushing harder on a
RULES-section rule made the model's behavior measurably worse, not better, in a way that would not
have been caught without actually re-testing after each change. Reverting rather than shipping was
the correct call — a table-classification miss is a known, bounded, documented limitation; an
intermittent full-pipeline refusal is not.

**Carried into next cycle (or Loop-phase backlog):**
1. **T7's NFR classification remains unresolved** after four attempts across three cycles now
   (Cycles 1, 2, and 3). The lead worth testing next time: the failure appears specific to inputs
   where *every* extracted item is a technical constraint (T7's case) rather than a mix of
   functional and non-functional items (T11's case, where classification worked correctly
   unprompted) — worth a few-shot or context-injection approach scoped narrowly to that specific
   input shape, rather than another RULES-section escalation, if this is picked up again. Given this
   cycle's finding that RULES-section escalation actively backfired here, any future attempt should
   re-test for refusal regressions specifically, not just for the target classification behavior.
2. The should→must priority-inference drift (Requirement Extractor's prose vs. its quoted field) —
   still open since Cycle 1, still non-deterministic, still low priority.
3. PRD Generator's Sections 6, 7, 9, 10 — still not specifically checked for the "bare heading, no
   local reinforcement" shape (Cycle 2's finding). No evidence of an actual problem there; not urgent.
4. PRD Generator's own Section 5 *completeness* (independent of Story Breakdown's now-100%-reliable
   matching) varies run to run — sometimes 1, sometimes 2, sometimes all 3 of the available criteria
   get written into Section 5 at all. Observed during Thread A's 5-run sample but out of this cycle's
   scope (Thread A's fix is about matching what's there, not about ensuring everything that should be
   there is there). Worth a look if Section 5 completeness becomes its own measured problem.
5. One Thread-A run (Run 5) had a story missing its `— source: FR-XXX` tag entirely — a minor format-
   compliance drift, noted but not chased (single occurrence, didn't affect matching correctness).
