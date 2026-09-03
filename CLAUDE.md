# CLAUDE.md — PRD Genie Capstone

This file is for Claude Code, running locally in this repo. Read it first, every session. It is
NOT the project's public-facing explanation — that's `README.md`. This file is the working state
tracker and the operating rules for how we build this repo, step by step, gated.

## What this project is

PRD Genie: an AI-powered product documentation assistant for NeuronForge Technologies. A
sequential 4-agent pipeline (Requirement Extractor → PRD Generator → Story Breakdown → Gap
Analyzer), orchestrated in n8n, built for the *Applied Agentic AI for PMs/TPMs* capstone (14-day
deadline). Full detail lives in `architecture/roadmap.html` — this file does not restate it, it
points at it.

## Operating rules (do not deviate without asking)

1. **One step at a time.** Work only on the task you were given in this session. Do not start the
   next roadmap day/phase step on your own initiative, even if it seems like the obvious next
   thing — stop and report back instead.
2. **Every task ends with a `## Summary`** section: Execution, Results, Key Findings,
   Observations, Suggestions. Never skip this, even if the task failed or was stopped early —
   report what happened instead.
3. **Stop on unexpected friction, don't route around it.** If something in a task's instructions
   conflicts with what you find in the repo (a file that shouldn't exist yet does, a credential
   isn't set, a tool isn't installed), stop and report it in the Summary rather than improvising a
   fix — unless the task explicitly authorizes a specific kind of deviation (e.g. Task 1
   authorized fixing the `.gitignore`/`.env.example` conflict).
4. **Update this file's State section** (below) before ending any task that changes it — new
   files landed, a phase completed, a decision made that future sessions need to know about.
5. **Never commit secrets.** Real API keys, the eval-service shared secret, and `.env` itself stay
   local only. `.env.example` is the committed template — real values never replace its blanks in
   git history.
6. **Hallucination guardrail is non-negotiable across all 3 model tiers**: UNKNOWN/explicit gap
   over invented content, always — see `architecture/ground-rules.md`. This applies exactly as
   strictly to the qwen3.5:4b and mistral:7b-instruct fallback tiers as to OpenAI primary.

## Key references (read these, don't duplicate their content here)

- `architecture/roadmap.html` — the 14-day plan, restructured into 6 phases (Discovery → Setup →
  Baseline → The Loop → Hardening/Handover). Source of truth for sequencing.
- `architecture/ground-rules.md` — scoring spec (12 baseline tests, PASS/FAIL criteria), the
  AI/technical + business metrics definitions, and the token estimates used for optimization.
- `architecture/cycle-template.md` — Run→Score→Learn→Change→Re-score template, filled once per
  evaluation cycle in `evidence/cycles/`.
- `docs/01-problem-charter-brief.md`, `docs/02-architecture-decision-record.md`,
  `docs/03-runbook-handover.md` — TPM engagement artifacts (Discovery, decisions-as-made,
  final handover).
- `README.md` — the human-facing repo explanation and submission-pack checklist.

## LLM stack (do not change without explicit instruction)

- **Primary**: OpenAI (IK-provided key), `gpt-4o-mini` default, step up to `gpt-4o` only if a
  specific step demonstrably needs it.
- **First fallback**: `qwen3.5:4b` via Ollama, local.
- **Second fallback**: `mistral:7b-instruct-q4_K_M` via Ollama, local — PRD Generator step only.
- Fallback triggers on primary failure/timeout, not on cost. Every result logs which tier served
  it (`model_tier` field in the eval service) — this is a tracked metric, not just routing detail.

## State (update this section every task)

- **Current phase**: Baseline phase complete (3 evaluation cycles, 13/14 pass rate) → in The Loop
  phase, building fallback infrastructure one agent at a time.
- **Last completed step**: Loop-phase Task 3 — fixed Task 2's empty-content problem. Root cause:
  the Ollama node's `think` option (unset in the workflow, so defaulting to `true`) makes
  qwen3.5:4b spend its generation budget on hidden reasoning separated from the final answer;
  against Requirement Extractor's long system prompt, it never reached the final answer at all
  (Task 2: 926s, 2902 output tokens, empty `content`). Setting `think: false` on the
  `Requirement Extractor - qwen3.5:4b Fallback` node in the live "PRD Genie - Full Pipeline"
  workflow fixed both problems at once: an isolated re-test (same system prompt + T1 input)
  completed in **80.6s** (vs. 926s) with **real, non-empty content** (141 output tokens). A
  second sample via the actual forced-OpenAI-failure full-pipeline path (same method as Task 1)
  confirmed the fix works end-to-end — OpenAI error → qwen3.5:4b fallback (51.1s) → `Tag:
  qwen3.5:4b` correctly shows `model_tier: "qwen3.5:4b"` → PRD Generator/Story Breakdown/Gap
  Analyzer all ran successfully downstream on the fallback-tier output. **Live pipeline restored
  to its correct state afterward**: OpenAI node back on `gpt-4o-mini`, fallback node's `think:
  false` kept, verified via a final export (8 nodes, no duplicates).
  **T1 scoring — both samples FAIL, consistently, in the same way.** Isolated test: filter-by-
  date/category/status present but the paraphrase narrowed to "date range" only; 2-second load
  time correct; **PM Sarah demoted from a stated fact into an "Owner: UNKNOWN" hedge that
  mischaracterizes a clearly stated fact as ambiguous**; **Q3 deadline completely absent, not
  mentioned anywhere.** Pipeline-path sample: broadly similar — PM Sarah and Q3 both technically
  present as words, but each wrapped in confused, self-contradictory reasoning that argues a
  clearly-stated fact might be "UNKNOWN" before conceding it's actually stated. Neither sample
  invents anything false (`hallucination_detected: No` both times) — the failure mode is
  **under-extraction / miscategorizing stated facts as unknown**, not fabrication. This is a real,
  repeatable quality gap, not a one-off sampling fluke.
- **Next step**: qwen3.5:4b is now **fast and non-empty but not yet trustworthy** — extending the
  fallback pattern to PRD Generator, Story Breakdown, or Gap Analyzer (or wiring mistral:7b as
  PRD Generator's second fallback) should wait until this extraction-quality gap is addressed,
  since it would otherwise just propagate silently-degraded requirement extractions downstream
  into every other agent. Two candidate directions for whoever picks this up: (a) a shorter,
  fallback-tier-specific system prompt (the current one is written for a strong instruction-
  following model like `gpt-4o-mini`; qwen3.5:4b may need the guardrail stated more simply/
  directly rather than at the same length/nuance), or (b) explicit few-shot examples in the
  fallback prompt showing that a directly-labeled fact ("PM: Sarah.") is never "UNKNOWN." Neither
  was attempted this task (out of scope — this task was specifically about the `think`/timeout
  mechanics, not prompt-quality tuning).
  **Timeout patch caveat still applies and is unchanged**: `axios.defaults.timeout` is still
  patched to `900000` in `n8n-n8n-1`'s live filesystem only (not the Docker image, not any
  git-tracked file) — will not survive a container recreation. Re-apply per Task 2's Summary if
  that ever happens.
- **Known deviations from spec, intentionally kept**:
  - `.gitignore` includes `!*.example` / `!.env.example` exemptions not in the original literal
    spec text, added because `.env.*` would otherwise silently exclude `.env.example` from every
    commit. This exemption covers any future `*.example` files too.
  - Ollama is reached from n8n's Docker container via `http://host.docker.internal:11434`, not
    `localhost` — see `.env.example` and `evidence/environment-readiness.md`'s "Ollama port-11434
    diagnosis" for the full root-cause (two unrelated Ollama-family processes on the same host both
    listening on port 11434, resolved by IP version, not by any config change).
- **Not yet started**: PRD Generator/Story Breakdown/Gap Analyzer fallback routing, mistral:7b
  second fallback, eval service code, dashboard, `n8n/prd-genie-pipeline.json` export (README.md's
  own checklist scopes this to "Day 9 onward... final on Day 14" — noted, not started prematurely),
  screenshots, demo video.
