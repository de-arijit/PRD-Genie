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
- **Last completed step**: Loop-phase Task 2 — found and fixed the root cause of Task 1's ~331s
  qwen3.5:4b timeout, but re-testing surfaced a SECOND, distinct blocker. Root cause of the timeout:
  `axios.defaults.timeout = 300000` (300s) hardcoded in `@n8n/backend-network`'s
  `configureGlobalAxiosDefaults()` — n8n's global HTTP client default, set once at process startup,
  **no environment variable override exists** for it in this n8n version. The Ollama node's own
  `apiRequest` sets no per-request timeout, so every call fell through to this global default (the
  observed ~331s = the 300s timeout plus connection/retry overhead). Fixed by patching
  `.../@n8n/backend-network/.../http/axios/config.js` from `300000` to `900000` (15 min) directly in
  the container filesystem (as root, via `docker exec`), applied to **both** the live `n8n-n8n-1`
  container (then `docker restart n8n-n8n-1` — Node.js caches loaded modules, so the file edit alone
  does nothing without a process restart; confirmed back up healthy afterward) **and** a separate
  persistent sibling container used for CLI testing (ephemeral `docker run --rm` sibling containers
  pull fresh from the unpatched base image every time, so they needed their own patch). **This patch
  is a live filesystem edit, NOT part of the Docker image build and NOT tracked in this git repo —
  it will NOT survive a container recreation (`docker compose down && up`, an image pull/upgrade,
  `docker rm`+recreate). Re-apply it (see the patch command in Loop-phase Task 2's own Summary) if
  `n8n-n8n-1` is ever recreated and qwen3.5:4b fallback work resumes.**
  **Re-test result: the timeout itself is fixed** — the same isolated qwen3.5:4b + T1-prompt call
  that failed 3/3 at ~331s in Task 1 now completed with `status: success`, no connection error, after
  926.585s (~15m26s) real generation time, 2902 output tokens. **But the response's final `content`
  field was completely EMPTY** — a new, different failure mode, not a timeout. Working hypothesis
  (not fixed or further investigated this task, per its explicit scope boundary): the Ollama node's
  `think` option defaults to `true` (separates a model's internal reasoning from its final output for
  "thinking"-capable models), and combined with a large, complex system prompt, qwen3.5:4b likely
  spent its entire ~2902-token generation budget on hidden reasoning and never reached the point of
  emitting the actual structured T1 answer. T1 could not be scored positively — an empty response
  trivially fails every PASS criterion.
- **Next step**: Loop-phase Task 3 (or whatever the user names it) needs to resolve the empty-
  content/`think`-mode issue before qwen3.5:4b can be considered a working fallback tier — candidates
  to investigate: setting the Ollama node's `think` option to `false`, and/or raising `num_predict`
  well above the 1024 default so there's room for both reasoning and a final answer if `think` stays
  on. **Do not extend the fallback pattern to PRD Generator, Story Breakdown, Gap Analyzer, or wire
  mistral:7b-instruct until qwen3.5:4b actually produces a real, scoreable T1 response** — two
  consecutive tasks now have found a real blocker on Requirement Extractor alone; multiplying an
  unproven tier across 3 more agents would just multiply unresolved problems. Separately, even once
  content is non-empty, ~15+ minutes for a single fallback call is a serious latency/UX concern worth
  the user's explicit attention regardless of pass/fail — a "fallback that keeps the pipeline usable"
  taking a quarter of an hour may not meet the spirit of why the fallback chain exists.
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
