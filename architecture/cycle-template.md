# Evaluation Cycle Log — Cycle #___

*One of these per Run→Score→Learn→Change→Re-score iteration. Copy this template for each new cycle.
Feeds both the dashboard's cycle log (via `POST /cycles`) and the Q3/Q4 writeup directly — write it in
language you'd be comfortable pasting straight into the submission.*

**Date:** ___________
**Trigger for this cycle:** (why are you running this — first pass? a known failure from last cycle? a prompt change to verify? a token-cost variance flagged on the dashboard?)
**Cycle type:** ☐ Correctness (fixing a failed/hallucinated test) ☐ Token optimization (reducing cost/size without changing correctness) ☐ Both

---

## 1. Run

- Which test(s): ___________ (e.g. all 12, or just T9/T2 if targeted)
- Which model tier served each: ___________
- Pipeline version / prompt version tested: ___________

## 2. Score

Compare against `ground-rules.md`. Fill one row per test run this cycle.

| Test | Pass/Fail | Hallucination? | Notes |
|------|-----------|-----------------|-------|
| | | | |

**Aggregate this cycle:** ___ / ___ passed · hallucination rate ___% · fallback rate ___%

## 3. Learn

For every FAIL or hallucination flagged above — root cause, not just symptom:

- What specifically went wrong: ___________
- Which agent produced it (check the Langfuse trace, not just the final output): ___________
- Why: was it the prompt, the model tier, the input itself, or the orchestration? ___________

*For a token-optimization cycle, fill this instead:*

- Which agent showed the largest estimate-vs-actual variance on the dashboard: ___________
- Estimated vs. actual (from `/summary`'s `token_estimate_vs_actual`): est. ___ / actual ___ (___%)
- Suspected cause (verbose system prompt? full template re-sent vs. referenced? oversized input passed
  through unnecessarily?): ___________

## 4. Change

- Exact change made (paste the before/after prompt diff, not a paraphrase): 

  ```
  BEFORE:
  ...

  AFTER:
  ...
  ```

- Why this specific change should fix the root cause identified above: ___________
- Scope of the change: one agent's prompt only, or does it touch the pipeline structure? ___________

## 5. Re-score

- Same test(s), same input(s), re-run against the changed prompt: ___________
- Result: pass/fail now vs. before — ___________
- Did the fix regress anything that was previously passing? Re-run the full 12, not just the fixed test: ___________

*For a token-optimization cycle, also confirm:*

- New actual token count vs. estimate: ___ (was ___, target ___) — variance now ___%
- **Correctness held constant?** Same pass/fail results as before the change — a token reduction
  that also drops a hallucination-guardrail line is not a win. Re-run T9/T2 specifically, since
  guardrail wording is exactly what a "shorten the prompt" edit risks cutting.

---

**Verdict:** ☐ Improvement confirmed — keep the change ☐ No change / inconclusive ☐ Regression — revert

**Carried into next cycle:** ___________
