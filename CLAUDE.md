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
- **Last completed step**: Loop-phase Task 1 — built and proved OpenAI→qwen3.5:4b fallback routing
  on Requirement Extractor only (not the other 3 agents — that's explicitly gated on this task
  proving the mechanism cleanly). In the live n8n workflow ("PRD Genie - Full Pipeline", id
  `0da58f21-edcb-4cb6-b2a4-c5a7261f51c4`): added `onError: 'continueErrorOutput'` to the
  Requirement Extractor OpenAI node, added a new `@n8n/n8n-nodes-langchain.ollama` node
  (`qwen3.5:4b`, same system prompt, existing `Ollama account` credential) wired to its error
  output, and two `Set` nodes tagging output with `{text, model_tier}` so PRD Generator/Gap
  Analyzer can consume either tier's output uniformly (`{{ $json.text }}`). **Wiring is confirmed
  correct**: a forced OpenAI failure (invalid model name, credential untouched) correctly routed to
  the qwen3.5:4b branch every time, and a normal run correctly tagged `openai/gpt-4o-mini` with
  zero qwen involvement. **The qwen3.5:4b leg itself is NOT reliable at the Requirement Extractor's
  actual prompt size**: 3 independent attempts (2 through the full pipeline's error-routing branch,
  1 an isolated direct call bypassing the pipeline entirely, all with the identical system prompt +
  T1 input) **all failed** with "the connection was aborted" at 331.783s, 330.965s, and 331.920s —
  three timings within 1 second of each other, conclusively a fixed timeout being hit (not host-load
  variance, not pipeline overhead — the isolated direct call failed identically). The Ollama node
  exposes no timeout override in its own options; the ~331s ceiling is coming from a global n8n or
  Docker-level HTTP default, out of this task's scope to change unilaterally. T1 could therefore
  NOT be scored against the qwen3.5:4b tier — every attempt to get a real completed response timed
  out. The standalone connectivity check (a trivial "reply OK" prompt) succeeded in ~2.5-3 min,
  proving the node/credential/model themselves work — this is specifically a large-system-prompt
  generation time problem, not a broken connection.
- **Next step**: Loop-phase Task 2 is **blocked** — do not extend this fallback pattern to PRD
  Generator, Story Breakdown, or Gap Analyzer (or wire mistral:7b-instruct) until the qwen3.5:4b
  timeout is actually fixed (raise whatever default HTTP/request timeout is being hit — candidate:
  n8n's own default request timeout — and/or investigate why a 4B model needs >331s to process a
  ~2,500-word system prompt on this host, and/or reduce host load — 28+ containers were running
  throughout this task). Extending to 3 more agents on top of an unproven fallback tier would just
  multiply an unresolved problem.
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
