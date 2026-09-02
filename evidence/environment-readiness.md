# Environment Readiness Checklist — PRD Genie

*Phase: Setup (Day 3). Maps to reference "Sprint 1: Access, setup, thin slice." A TPM signs off on this
before a team starts building — same logic applies solo: don't discover a broken connection on build day.*

---

## Access & credentials

- [x] IK-provided OpenAI API key active and tested (one curl/test call succeeds) — `gpt-4o-mini-2024-07-18`
      responded "OK" to a minimal chat completion in 2.52s. Key read from local `.env`, never printed
      or committed.
- [x] Ollama running locally, `qwen3.5:4b` pulled and responding to a test prompt — tested via
      `ollama run` CLI, responded "OK" (with a visible thinking/reasoning trace first) in ~89s.
- [x] `mistral:7b-instruct-q4_K_M` pulled (fallback tier 2, PRD Generator only) — already present
      locally (no download needed), the exact tag requested. Tested via `ollama run` CLI, responded
      "OK" in ~26s.
- [ ] n8n instance running, reachable at its local URL — **BLOCKED.** See "n8n startup failure" below.
- [ ] Langfuse account created, API keys generated, connected to n8n — not done, scheduled for a
      later day per the roadmap, not part of Day 3's minimum bar. Not a gap.

### n8n startup failure (details)

n8n is installed (global `n8n@0.126.1` was already present via npm), but **`n8n start` fails** with
`Error: There was an error: glob is not a function` and exits immediately — the editor UI at
`http://localhost:5678` never becomes reachable. Diagnosed as follows, not worked around:

- Confirmed this isn't a stale/broken global install specifically: ran `npx n8n@latest start`
  (fresh install, n8n 2.37.7, isolated from the global install) — **same exact error**, same exit
  code.
- `npx n8n@latest --version` succeeds fine (prints `2.37.7`) — so the CLI entry point and basic
  module loading work; the failure is specifically in code paths `start` exercises.
- Root cause looks like a Node.js/`glob` package incompatibility in this environment: Node is
  v24.14.0 (very new), and `npm ls -g glob` shows the global npm tree has both a modern
  `glob@13.0.6` (class-based API, no longer a callable function) hoisted at the top level and
  several old `glob@7.x`/`glob@10.x` copies nested under n8n's own dependencies (`sqlite3`,
  `typeorm`, `mqtt` etc., which still expect the old callback-style `glob(pattern, cb)` function
  export). Something in this resolution mix is handing n8n's old-style caller the new
  class-based export.
- Did not attempt to fix this (e.g., pinning/removing global `glob`, downgrading Node system-wide)
  — that's a nontrivial environment change with effects beyond this repo, out of scope for a
  verify-only readiness task, and explicitly the kind of thing this task said to stop and report
  rather than route around.

## Workflow platform

- [ ] n8n OpenAI node configured and tested with a trivial prompt — not done, out of scope for this
      task (pipeline-building work for a later task). Also currently blocked transitively by the n8n
      startup failure above.
- [ ] n8n Ollama node configured and tested against `qwen3.5:4b` — not done, same reason.
- [ ] Fallback logic sketched — not done, same reason.

## Data

- [ ] All 5 input files loaded into n8n (or confirmed accessible) — **none of the 5 files are present
      anywhere accessible**: `sample_meeting_transcripts.txt`, `sample_product_brief.txt`,
      `stakeholder_notes.txt`, `prd_template.md`, `eval_prdgenie_inputs.txt`. Searched the full repo
      (`Glob` across the tree) and the obvious course-material locations on this machine (Documents,
      Desktop, Downloads, OneDrive, and a filesystem-wide search) — no matches anywhere. Not
      fabricated as placeholders per instruction.
- [ ] `prd_template.md` structure confirmed — **cannot confirm, file doesn't exist yet** (see above).

## Thin-slice proof (Sprint 1's actual deliverable)

- [ ] One agent (Requirement Extractor) built — not started, this task's explicit scope boundary
      (next task's work, gated on this one).
- [ ] Run against T1 only — not started, same reason.
- [ ] Output manually verified — not started, same reason.
- [ ] **Sign-off:** thin slice works end-to-end — not reached; gated on n8n readiness and the 5 data
      files landing first.

## Eval service pre-check (used starting Day 9, confirmed reachable now to avoid a Day-9 surprise)

- [x] `eval-service/main.py` dependencies installable (`fastapi`, `uvicorn`) — created a throwaway
      venv, installed both, confirmed both import successfully (`fastapi` 0.141.1, `uvicorn` 0.52.4),
      then removed the venv. Not committed (was created outside the repo, in system temp, so never a
      risk either way).
- [x] Cloudflare Tunnel CLI installed and tested — was not previously installed; installed via
      `winget install Cloudflare.cloudflared` (version 2026.8.3). `cloudflared --version` confirmed
      working. No actual tunnel stood up, per this task's scope (needed only once the eval service
      itself exists, Day 9+).

---

**Readiness verdict:** ☐ Green — proceed to Baseline phase ☐ Yellow — proceed with noted gaps: ___________ ☒ Red — blocked on: **n8n cannot start on this machine (`glob is not a function`, reproduced on
both the pre-existing global install and a fresh `npx n8n@latest`) — the editor UI is unreachable,
which blocks the Workflow platform and Thin-slice proof sections entirely. Separately, none of the 5
required input data files exist anywhere on this machine yet — needed before the thin-slice proof can
run even once n8n is fixed.**
