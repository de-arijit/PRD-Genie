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

## Baseline run — results

*One row per test. This is the literal table the playbook's Step 5 requires ("5 points specifically for
baseline dataset tested with documented results") — TAs check each row against the expected criteria.*

| Test ID | Input (summary) | Output (summary/excerpt) | Expected keywords present? | Hallucination detected? | Pass/Fail |
|---|---|---|---|---|---|
| T1 | | | | | |
| T2 | | | | | |
| T3 | | | | | |
| T4 | | | | | |
| T5 | | | | | |
| T6 | | | | | |
| T7 | | | | | |
| T8 | | | | | |
| T9 | | | | | |
| T10 | | | | | |
| T11 | | | | | |
| T12 | | | | | |

**Baseline pass rate:** ___ / 12
**Baseline hallucination rate:** ___%
**Model tier that served each run:** OpenAI ___ / qwen3.5:4b ___ / mistral ___

## Notable findings

- Strongest capability: ___________
- Weakest capability: ___________
- Which test(s) needed a second look / manual verification: ___________

## Handoff to The Loop phase

This baseline is the "before" state for Cycle 1 in `cycle-template.md`. Carry the failing tests and
their root causes forward as that cycle's starting point — don't re-diagnose from scratch on Day 8.

**Open items carried forward:** ___________
