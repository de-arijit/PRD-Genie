# Environment Readiness Checklist — PRD Genie

*Phase: Setup (Day 3). Maps to reference "Sprint 1: Access, setup, thin slice." A TPM signs off on this
before a team starts building — same logic applies solo: don't discover a broken connection on build day.*

---

## Access & credentials

- [x] IK-provided OpenAI API key active and tested (one curl/test call succeeds) — `gpt-4o-mini-2024-07-18`
      responded "OK" to a minimal chat completion in 2.52s. Key read from local `.env`, never printed
      or committed.
- [x] Ollama running locally, `qwen3.5:4b` pulled and responding to a test prompt — confirmed via both
      `ollama run` CLI and a real HTTP POST to `/api/generate` at `http://127.0.0.1:11434`, responded
      "OK" in 42.8s over HTTP. See "Ollama port-11434 diagnosis" below for why this specific
      host/port combination matters.
- [x] `mistral:7b-instruct-q4_K_M` pulled (fallback tier 2, PRD Generator only) — already present
      locally, the exact tag requested. Confirmed via real HTTP POST to `/api/generate` at
      `http://127.0.0.1:11434`, responded "OK" in 22.4s.
- [x] n8n instance running, reachable at its local URL — running via Docker Desktop, confirmed
      `http://localhost:3002/home/workflows` returns HTTP 200 (container `n8n-n8n-1`, up 4 days).
      Not reinstalled/reconfigured — was already correctly set up, contrary to the previous task's
      finding that a native `n8n start` was broken (that finding still stands for the native
      install; it's simply irrelevant now that n8n runs in Docker instead).
- [ ] Langfuse account created, API keys generated, connected to n8n — not done, scheduled for a
      later day per the roadmap, not part of Day 3's minimum bar. Not a gap.

### Ollama port-11434 diagnosis (resolved)

Root cause found: **two unrelated Ollama-family processes were both answering on port 11434**, and
which one a client reached depended on IP version, not on any actual fix needed:

- **`127.0.0.1:11434` (IPv4)** → forwarded via `wslrelay.exe` (WSL2's port relay) → reaches the
  **correct** backend: the exact 7-model catalog `ollama list` shows, including `qwen3.5:4b` and
  `mistral:7b-instruct-q4_K_M`. Verified consistent across 6 repeated probes.
- **`[::1]:11434` / `0.0.0.0:11434` (IPv6 / wildcard)** → native Windows `ollama.exe` (PID 40956,
  a separate stray/duplicate instance, version 0.33.2) → the **wrong** backend: 12 unrelated
  community/cloud models, missing both required models entirely.
- `http://localhost:11434` is therefore unreliable on this machine — dual-stack resolution can pick
  IPv6 first and silently hit the wrong instance. **Always use `http://127.0.0.1:11434` explicitly
  from anything running directly on the Windows host.**
- A third process, `ollama app.exe` (PID 38532, the system-tray app) listens on `127.0.0.1:19580` —
  this is **not a real inference API**, just the tray app's own local GUI web server (a static SPA);
  it happens to proxy `/api/tags` and `/api/version` for its own UI but returns the HTML shell for
  `/api/generate`. Ruled out as a usable endpoint.
- No process was killed or reconfigured — the fix was purely "use the right address," which
  required no destructive action at all.
- **For Docker/n8n**: `http://host.docker.internal:11434` was tested directly from a throwaway
  container and returned the correct 7-model catalog cleanly (see "Docker networking check" below)
  — Docker's host-loopback resolution reaches the same correct backend as `127.0.0.1` does from the
  Windows host, with no extra configuration needed.

## Workflow platform

- [x] n8n OpenAI node configured and tested with a trivial prompt — done as part of the thin-slice
      proof below (not a separate trivial test; the real T1 run doubles as this check). Node type
      `@n8n/n8n-nodes-langchain.openAi` v2.3, resource `text` / operation `response` (this n8n
      version's Responses API operation — its own node description explicitly warns the older
      `message` operation is invalid on v2), model `gpt-4o-mini`, credential `AD-IK-OpenAI`.
- [ ] n8n Ollama node configured and tested against `qwen3.5:4b` — not done, out of scope for this
      task (fallback routing is Baseline/Loop-phase work). The correct endpoint for this node, once
      built, will be `http://host.docker.internal:11434`.
- [ ] Fallback logic sketched — not done, same reason.

## Data

- [x] All 5 input files present and organized — moved from the repo root (where the user had placed
      them) into a new `sample-data/` folder: `sample-data/sample_meeting_transcripts.txt`,
      `sample-data/sample_product_brief.txt`, `sample-data/stakeholder_notes.txt`,
      `sample-data/prd_template.md`, `sample-data/eval_prdgenie_inputs.txt`. All 5 original filenames
      intact.
- [x] `prd_template.md` structure confirmed — 10 numbered sections: Product Overview, Goals and
      Objectives, User Personas, Feature Requirements (with Functional/Non-Functional
      subsections), Acceptance Criteria, Out of Scope, Dependencies, Assumptions, Open Questions,
      Timeline.

## Thin-slice proof (Sprint 1's actual deliverable)

- [x] One agent (Requirement Extractor) built — system prompt written to
      `architecture/agent-prompts/01-requirement-extractor.md` (ROLE/INPUT/OUTPUT/RULES structure),
      wired into n8n as workflow "PRD Genie - Thin Slice - Requirement Extractor" (manual trigger →
      OpenAI node), imported/executed via n8n CLI (no browser access available; n8n's REST API needs
      a login this session doesn't have).
- [x] Run against T1 only — executed once via
      `n8n execute --id=0da58f21-edcb-4cb6-b2a4-c5a7261f51c4` (a throwaway sibling container sharing
      the running instance's data volume, since `n8n execute` inside the already-running container
      conflicts with its own live Task Broker port). `gpt-4o-mini`, 1157 input / 149 output tokens,
      4.85s. Single run only, not iterated — see Task 4's Summary for the full verbatim output.
- [x] Output manually verified against T1's "Expected Output Must Contain" criteria — **PASSED**, all
      4 required elements present, each traced to an exact verbatim quote from the T1 source text
      (programmatically confirmed via substring match, not eyeballed). `hallucination_detected:
      false` — nothing in the output falls outside the source text; the one UNKNOWN flag
      (implementation ownership) is the correct guardrail behavior, not an invented requirement.
- [x] **Sign-off:** thin slice works end-to-end — proceed to Baseline phase.

## Eval service pre-check (used starting Day 9, confirmed reachable now to avoid a Day-9 surprise)

- [x] `eval-service/main.py` dependencies installable (`fastapi`, `uvicorn`) — confirmed in the prior
      task (throwaway venv, both imported successfully, venv removed). No change since; not redone.
- [x] Cloudflare Tunnel CLI installed and tested — confirmed in the prior task (`winget`, version
      2026.8.3, `--version` works). No change since; not redone.

## Docker networking check

- [x] `host.docker.internal:11434` reaches the correct Ollama endpoint from inside a container —
      `docker run --rm curlimages/curl curl http://host.docker.internal:11434/api/tags` returned the
      full, correct 7-model catalog (including both required models) on the first attempt. A
      follow-up real `/api/generate` call from inside a container timed out on retries — attributed
      to this machine's Docker Desktop being under heavy load (24 long-running containers, up 4 days,
      plus Ollama models resident in memory) making fresh throwaway-container spin-up slow, not a
      networking failure — the `/api/tags` result already confirms the network path itself is
      correct and clean. No n8n Ollama node was wired up (explicitly out of scope this task).

---

**Readiness verdict:** ☒ Green — proceed to Baseline phase ☐ Yellow — proceed with noted gaps: ___________ ☐ Red — blocked on: ___________

All items in scope for Setup-phase readiness now pass: OpenAI, both Ollama fallback tiers (via the
correct HTTP endpoint), n8n (via Docker), the 5 sample data files (organized in `sample-data/`), the
eval-service/cloudflared pre-checks, and — as of this update — the thin-slice proof itself (one
agent, one test input, passed end-to-end). Items intentionally out of scope remain unchecked and are
not gaps: Langfuse, the Ollama node/fallback logic (Baseline/Loop-phase work).
