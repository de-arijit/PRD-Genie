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

- **Current phase**: Discovery (Day 1–2 equivalent)
- **Last completed step**: Task 1 — repository scaffolding, initial commit, GitHub repo created
  and pushed (`de-arijit/PRD-Genie`, commit `25d9cee`, branch `main`).
- **Next step**: Task 2 — populate `README.md`, this `CLAUDE.md` (already done),
  `architecture/ground-rules.md`, `architecture/cycle-template.md`, `architecture/roadmap.html`,
  and the five `docs/`/`evidence/` artifact files with their real content (currently only
  `.gitkeep` placeholders exist in most folders).
- **Known deviations from spec, intentionally kept**:
  - `.gitignore` includes `!*.example` / `!.env.example` exemptions not in the original literal
    spec text, added because `.env.*` would otherwise silently exclude `.env.example` from every
    commit. Keep this exemption for any future `*.example` files too.
- **Not yet started**: agent prompts (`architecture/agent-prompts/`), n8n workflow, eval service
  code, baseline test runs, dashboard, cycle logs, screenshots, demo video.
