# Evaluation Cycle Log — Cycle #2

*One of these per Run→Score→Learn→Change→Re-score iteration. Copy this template for each new cycle.
Feeds both the dashboard's cycle log (via `POST /cycles`) and the Q3/Q4 writeup directly — write it in
language you'd be comfortable pasting straight into the submission.*

**Date:** 2026-09-02
**Trigger for this cycle:** Two separate items carried forward from Cycle 1: (1) a *pattern*, not
yet a confirmed bug — 2 of 3 post-Cycle-1-fix PRD Generator runs invented ungrounded content in
Sections 1, 2, or 8, sections Cycle 1 never touched, needing a real sample before acting; (2) a
*design decision* — the corrected `ground-rules.md`'s T4 story-stage row requires verbatim
acceptance criteria in a format ("As a…, I want…") that structurally requires paraphrasing,
needing a resolution and an ADR before it could be re-tested.
**Cycle type:** ☒ Correctness (fixing a failed/hallucinated test) ☐ Token optimization ☐ Both

*This cycle has two distinct threads — the Section 1/2/8 measurement and the T4 story-stage ADR
decision — kept separate throughout rather than blended into one narrative.*

---

## Thread A — Section 1/2/8 groundedness measurement

### 1. Run

- Which test(s): T1 → Requirement Extractor → PRD Generator, run **8 additional times** with
  identical T1 input, for a combined sample of **9** (the 8 new runs plus Cycle 1's T11-after run,
  which also used T1's exact input — Cycle 1's other two post-fix runs, T4-after and T7-after, used
  T4's and T7's input respectively, so they're reported separately, not folded into this specific
  9-sample tally).
- Which model tier served each: OpenAI (`gpt-4o-mini`) for all 9.
- Pipeline version / prompt version tested: Requirement Extractor unchanged (byte-identical across
  all 9 runs). PRD Generator: the Cycle-1-fixed prompt (Section 5 verbatim instruction + Persona
  Key Need grounding), *before* this cycle's Rule 1 change.

### 2. Score

| Run | Section 1 | Section 2 | Section 8 | Notes |
|---|---|---|---|---|
| New run 1 | **Invented** (Product Name, Document Version) | clean | clean | |
| New run 2 | clean | clean | clean | |
| New run 3 | clean | clean | clean | |
| New run 4 | **Invented** (Product Name, Document Version) | clean | clean | |
| New run 5 | clean | clean | clean | |
| New run 6 | clean | clean | clean | |
| New run 7 | clean | clean | clean | |
| New run 8 | clean | clean | clean | |
| Cycle 1's T11-after | **Invented** (Product Name, Version, Status) | **Invented** (Success Metrics) | **Invented** (Assumptions) | The one run where all 3 sections failed together |

**Aggregate this cycle (measurement only):** 3 / 9 runs had invention in at least one of Sections
1/2/8 · hallucination rate (this specific pattern) 33% · fallback rate 0% (OpenAI only)

**Which section is worst, and is it consistent or scattershot?** Section 1 is the dominant failure
— invented in all 3 of the 3 failing runs (3/9 overall), always the same *kind* of claim (a
made-up product name, a defaulted "Document Version: 1.0," occasionally a defaulted "Status").
Sections 2 and 8 never failed independently — both only appeared in the single run where Section 1
also failed. This is **consistent invention** (the same category of claim every time), not
scattershot noise — a real, specific prompt gap, not generic model variance. Meets the task's own
"3+/9" bar for acting.

### 3. Learn

Read `02-prd-generator.md`'s actual current text for Sections 1, 2, and 8, rather than assuming:

- **Section 1** (`## 1. Product Overview`): five bare field labels (`- **Product Name:**` etc.) with
  *zero* attached instructions.
- **Section 2** (`## 2. Goals and Objectives`): three bare field labels, same — zero attached
  instructions.
- **Section 8** (`## 8. Assumptions`): a bare heading with *nothing* under it at all — no labels, no
  example, no guidance.

The only instruction covering these three sections was the general paragraph positioned *after all
10 section headers* ("For every field or table row above: fill it ONLY if..."). This confirms the
hypothesis: Sections 3 (Key Need) and 5 (Acceptance Criteria) each got an explicit, localized
reminder with a worked example directly at their point in the prompt during Cycle 1; Sections 1, 2,
8 never got the same treatment and were relying entirely on a distant, general instruction to be
recalled and correctly applied to a completely different kind of field.

A second, non-obvious insight from reading the actual failures: Section 1's invented fields
(Product Name, Document Version, Status) are *administrative metadata*, not "extracted content" in
the way Section 3/5's fields obviously are. The existing Rule 1 ("never invent content for any
section") is framed throughout the rest of the prompt in requirement-extraction language — the
model may not have been connecting "a product needs *some* name and version, so I'll supply a
reasonable one, the way a human technical writer filling out a template would" to the same
invention rule that clearly applies to a fabricated requirement. This is why the fix (Section 4
below) explicitly names Sections 1/2/8 and explicitly rejects the "but every PRD needs a name"
framing, rather than just repeating "don't invent" more emphatically.

### 4. Change

Per the task's explicit instruction: since the pattern was confirmed at meaningful frequency
(3/9), extend the *proven* "Not specified in source" fallback to Sections 1/2/8 — but as **one
consolidated rule**, not three more per-section patches (deliberately different from Cycle 1's
approach, per instruction).

```
BEFORE:
1. NEVER INVENT CONTENT FOR ANY SECTION. If the input doesn't support a field, write "Not
   specified in source — flagged for stakeholder input" in that exact wording. It is always wrong
   to fill a gap with something that sounds plausible for a "typical" PRD. It is never wrong to
   mark a field as not specified.

AFTER:
1. NEVER INVENT CONTENT FOR ANY SECTION — INCLUDING SECTIONS THAT FEEL LIKE ADMINISTRATIVE
   BOILERPLATE. If the input doesn't support a field, write "Not specified in source — flagged for
   stakeholder input" in that exact wording. This applies with EQUAL force to every section in this
   document, not just the ones that obviously involve extracting a requirement — Section 1's
   Product Name / Document Version / Author / Date / Status, Section 2's Business Goal / User Goal
   / Success Metrics, and Section 8's Assumptions are exactly as bound by this rule as Section 3's
   Key Need or Section 5's Acceptance Criteria. A generic placeholder that "every PRD needs" — a
   made-up product name, defaulting to "Version 1.0," defaulting Status to "Draft," a success
   metric that merely sounds reasonable for this kind of feature — is exactly as much an invention
   as a fabricated requirement, even though it feels more like harmless boilerplate than a factual
   claim. It is always wrong to fill any field, anywhere in this document, with something that
   sounds plausible. It is never wrong to mark a field as not specified.
```

**Why this should fix the root cause:** it explicitly names all three previously-unguarded
sections at the most prominent point in the prompt (Rule 1, the first rule in the section the
prompt itself says to "read more than once"), and directly counters the specific reasoning
shortcut identified in Learn — that administrative-feeling fields are somehow exempt.
**Scope:** PRD Generator's prompt only, one rule strengthened, no other agent touched.

### 5. Re-score

Re-ran T4→Section 5, T11, and T7 (fresh, against the new Rule 1) — see Thread A+B combined re-score
below, since these same three runs also carry Thread B's evidence. Section 1/2/8 results:

| Test | Section 1 | Section 2 | Section 8 |
|---|---|---|---|
| T4 (Section 5 re-score) | clean | clean | clean (correctly grounded: "ownership... unclear," traces to a real extractor gap) |
| T11 (re-score) | clean | clean | clean (correctly grounded: "Ownership details: UNKNOWN," traces to a real extractor gap) |
| T7 (re-score) | clean | clean | clean (correctly grounded, same pattern) |

**3 / 3 clean this round — zero new invention across all three re-scored PRD Generator runs.**
Combined with the fact that Section 8 now actively surfaces real, grounded uncertainty (rather than
either inventing or going silent), this is a strong, direct confirmation of the fix. (A 3-sample
re-confirmation isn't a full second 9-run measurement — the task asked for one re-score pass, not a
repeat study — but it's a clean result with no counter-evidence.)

---

## Thread B — T4 story-stage format tension (ADR-006)

### Design decision

See `docs/02-architecture-decision-record.md`'s **ADR-006** for the full context, decision, and
alternatives considered. Summary: added a separate `Acceptance Criteria: "<verbatim text>"` line
under each story, kept deliberately apart from the "As a…, I want…, so that…" sentence (which stays
free to paraphrase). `03-story-breakdown.md`'s INPUT section was extended to read PRD Section 5
(previously explicitly excluded), and a new Rule 4 states the separation directly. Full diff:

```
BEFORE (INPUT section):
You will receive one full PRD document, in the PRD Generator's 10-section format. Your job only
uses two parts of it:
- Section 3 (User Personas) — for the persona names to write stories from.
- Section 4.1 (Functional Requirements table) — the actual features to break into stories.
Ignore Sections 1-2 and 5-10 entirely...

AFTER (INPUT section):
You will receive one full PRD document, in the PRD Generator's 10-section format. Your job uses
three parts of it:
- Section 3 (User Personas) — for the persona names to write stories from.
- Section 4.1 (Functional Requirements table) — the actual features to break into stories.
- Section 5 (Acceptance Criteria) — the verbatim criteria to attach to a story, when one exists for
  that requirement.
Ignore Sections 1-2 and 6-10 entirely...

BEFORE (story format):
- As a [persona], I want [goal], so that [benefit]. — Priority: [Must/Should/Nice] — source: [FR ID]

AFTER (story format):
- As a [persona], I want [goal], so that [benefit]. — Priority: [Must/Should/Nice] — source: [FR ID]
  Acceptance Criteria: "[verbatim criterion text, only if one exists for this row]"

NEW Rule 4 (renumbering old 4-7 to 5-8):
4. THE STORY SENTENCE AND THE ACCEPTANCE CRITERIA LINE ARE NOT THE SAME THING — DO NOT BLEND THEM.
   The "As a…, I want…, so that…" sentence is allowed, expected even, to paraphrase the requirement
   into natural, readable language. The "Acceptance Criteria:" line underneath it is not a
   paraphrase target — it is an exact quote, copied character-for-character from PRD Section 5.
   Keep them separate...
```

### Re-run to confirm (once, per instruction)

**First re-run (T4's own story stage, Phase A):**
```
- As a user, I want to export reports as PDF and CSV, so that the stated requirement is met. — Priority: Should (priority inferred — not explicit in source) — source: FR-001
- As a user, I want the PDF to include the company logo, so that the stated requirement is met. — Priority: Must — source: FR-002
- As a user, I want the CSV to preserve formulas, so that the stated requirement is met. — Priority: Must — source: FR-003
  Acceptance Criteria: "CSV must preserve formulas."
```
**Result: partial.** FR-003 correctly got its verbatim `Acceptance Criteria:` line — proving the
design works when it triggers: the sentence paraphrases naturally, the criteria line stays exact.
FR-001 correctly had no line (PRD Generator's own Section 5 had no matching bullet for it that run
— nothing to attach, correctly omitted). But **FR-002 had a matching Section 5 criterion
available** ("PDF must include company logo.") **and it wasn't attached** — a matching-reliability
miss, not a design flaw.

**Second data point (T12, Phase B regression, against a different PRD — T11's):**
```
- As a Sarah, I want to filter reports by date range, category, and status, so that the stated requirement is met. — Priority: Must — source: FR-001
  Acceptance Criteria: "The user should be able to filter reports by date range, category, and status."
- As a user, I want the results of the reports to load in under 2 seconds, so that the stated requirement is met. — Priority: Must — source: FR-002
  Acceptance Criteria: "Results must load in under 2 seconds."
```
**Result: 2/2, clean.** Both eligible criteria correctly attached this time.

**Combined: 3 of 4 eligible criteria correctly attached across the two observed runs (75%).** The
architectural resolution is confirmed sound — the "As a…" sentence and the verbatim criteria line
never blend into each other in any observed run, satisfying ADR-006's actual goal. But
per-row matching isn't 100% reliable yet (FR-002's miss). Per `ground-rules.md`'s literal T4
story-stage FAIL trigger ("...or omits either criterion"), **T4's own story-stage re-run is scored
FAIL** — the criterion for FR-002 was omitted from that specific run — even though the underlying
design problem this cycle set out to fix (verbatim vs. paraphrase-format tension) is resolved.
Reported precisely rather than rounded up to a clean pass.

---

## Combined Re-score (all threads) — full picture

| Test | Pass/Fail | Hallucination? | Notes |
|------|-----------|-----------------|-------|
| T4 — Section 5 | **PASS** | No | Verbatim criteria confirmed again; Section 1/2/8 clean (Thread A confirmation) |
| T4 — Story stage | **FAIL** | No | ADR-006 design confirmed sound (no blending), but FR-002's available criterion was omitted this run — a matching-reliability gap, not the architectural tension |
| T11 | **PASS** | No | Section 5 verbatim, Key Need grounded, Section 1/2/8 all clean — every issue found across both cycles is now resolved for this test |
| T7 | **FAIL** | No | NFR classification still unaddressed, correctly out of scope this cycle; Section 1/2/8 clean |

**Aggregate cycle 2 (targeted 4):** 2 / 4 passed · hallucination rate 0% (down from Cycle 1's 2/4)

### Regression check (same 13 as Cycle 1, re-run in full)

All 9 Requirement Extraction tests (T1, T2, T3, T5, T6, T7-extraction, T8, T9, T10), T12 (against
the new T11 PRD — the most likely regression point), and all 3 Gap Analysis sub-tests (T2, T5, T9)
were re-run. **13 / 13 passed. Zero regressions.** T12 in particular is worth noting: run against
Cycle 2's freshly-regenerated T11 PRD output, it not only held its own format/priority correctness
but also picked up the new Acceptance Criteria lines cleanly (2/2) — the strongest evidence this
cycle that ADR-006's design generalizes beyond the one input it was designed against.

### Cycle 2 grand total (14 numbered-capability rows + 3 Gap Analysis, same structure as Cycle 1)

**12 / 14 passed** (up from Cycle 1's 11/14) · **0 / 4 hallucinations in this cycle's re-scored
items** (down from Cycle 1's 2/4) · Gap Analysis: **3 / 3, 0 hallucinations**

Remaining FAILs: **T4 (story stage)** — matching-reliability gap, architecture resolved; **T7 (NFR
note)** — correctly untouched, out of scope this cycle.

---

**Verdict:** ☒ Improvement confirmed — keep the change ☐ No change / inconclusive ☐ Regression — revert

Both threads produced real, measured improvement. Thread A: a properly-sampled 9-run measurement
confirmed a real 3/9 pattern, root-caused by direct prompt inspection (not assumption), fixed with
one consolidated rule per instruction, and confirmed clean on all 3 re-scored runs plus zero
regressions across 13 previously-passing tests. Thread B: a genuine architectural tension in the
corrected spec was identified, resolved with a documented design decision (ADR-006), and confirmed
to work correctly (no sentence/quote blending in any observed run) even though per-row matching
isn't yet 100% reliable. T11 — failing in both the original baseline and Cycle 1 — passes cleanly
for the first time this cycle, with every previously-found issue resolved.

**Carried into next cycle:**
1. T4's story-stage matching reliability: Story Breakdown's Acceptance Criteria matching logic
   works when it works (3/4 observed), but isn't reliably finding every eligible Section 5 bullet.
   Worth a small sample (3-5 runs) to see if this is systematic (e.g. always misses a specific
   position in the list) or genuine sampling noise before deciding whether it needs its own fix.
2. T7's NFR classification (Section 4.2) — still unaddressed, correctly out of scope across two
   cycles now. Should be Cycle 3's primary target if Loop-phase work doesn't take priority first.
3. The should→must priority-inference drift (Requirement Extractor's prose vs. its quoted field) —
   still open, still non-deterministic, still low priority relative to the above.
4. Now that Sections 1/2/8's fix is confirmed, worth a wider check (not urgent) of whether any
   *other* PRD Generator section has the same "bare heading, no local reinforcement" shape this
   cycle's Learn step identified as the actual root cause — Sections 6, 7, 9, 10 weren't part of
   this cycle's measurement and haven't been specifically checked.
