# Runbook & Handover — PRD Genie

*Phase: Hardening/Handover (Days 12–14). Maps to reference "Sprint 6: Hardening, handover." The
document a TPM leaves so someone else — a TA grading cold, or you in six months — can run the system
and understand its state without re-deriving it from the prompt files.*

---

## How to run the pipeline

1. Confirm environment readiness — see `02-environment-readiness.md`. All boxes checked.
2. Start Ollama (`ollama serve`, if not already running as a service).
3. Open the n8n workflow (`prd-genie-pipeline.json`, exported per submission checklist).
4. Trigger with a test input (file upload node) — one of the 5 provided sample files, or a new transcript.
5. Pipeline runs: Requirement Extractor → PRD Generator → Story Breakdown → Gap Analyzer.
6. Output: structured PRD (Markdown, matches `prd_template.md`) + user stories + clarification questions if any input was flagged ambiguous.

## How to run an evaluation cycle

1. Copy `cycle-template.md` → name it `cycle-0X.md`.
2. Run the 12 baseline tests (or a targeted subset) through the pipeline.
3. Score each against `ground-rules.md`.
4. Fill Run/Score/Learn/Change/Re-score in the copied template as you go.
5. If the eval service is running: POST each result to `/results`, and the cycle summary to `/cycles`
   (see `eval-service/README.md` for the exact payloads — not yet created; lands with the eval
   service build, Day 9).
6. On next scheduled refresh (or manual trigger), `dashboard.html` regenerates with the new numbers.

## Known limitations (state this plainly — a real handover doesn't hide these)

- Local fallback tier (qwen3.5:4b) has a higher hallucination rate than OpenAI primary on vague/empty
  inputs (T2, T9-style) — see Cycle 1–3 logs for the specific failure mode and partial fix.
- Eval service requires the Cloudflare Tunnel to stay up; if it drops, `/summary` becomes unreachable
  and the dashboard stops refreshing (it does NOT silently show stale data as if live — check the
  "last refreshed" timestamp on the dashboard before trusting a number).
- OpenTelemetry is documented as a future step, not implemented (see ADR-003) — current tracing is
  Langfuse-SDK-specific.
- Business metrics (cost/PRD, time saved) use approximate constants (gpt-4o-mini list pricing, a
  3-hour manual-PRD baseline) — real production numbers would need actual usage data to replace these.

## If something breaks

| Symptom | Likely cause | Fix |
|---|---|---|
| Pipeline produces a PRD from an empty/vague input | Guardrail regression, possibly after a prompt edit | Check which agent/tier produced it in Langfuse trace; compare against the Cycle 1 fix in cycle logs |
| Eval service `/summary` returns 401 | API key mismatch between n8n and the service | Confirm `X-API-Key` header matches `main.py`'s `API_KEY` |
| Dashboard numbers look stale | Tunnel down, or scheduled task hasn't fired | Check tunnel status; manually POST/GET to confirm service is reachable |
| A specific baseline test regresses after a prompt change | Guardrail change was too broad | Re-run the full 12, not just the target test, after every prompt edit — see Loop-phase cycle discipline |

## Ownership at handover

| Component | Owner | Notes |
|---|---|---|
| n8n workflow | You | Exported as JSON in repo |
| Eval service | You | Not meant to run unattended long-term — a demo/grading-window service |
| Dashboard | You | Static file, regenerated on refresh — not a permanently hosted page |

## What a future iteration should do next

1. Fine-tune the qwen3.5:4b fallback tier specifically on stated-vs-assumed examples (see roadmap Day 12).
2. Consider Version Comparison as a second extended capability.
3. If this moves toward real production use: replace approximate cost/time constants with measured data, and revisit ADR-003 (OTel) if instrumentation needs to become vendor-neutral.
