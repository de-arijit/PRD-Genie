# Environment Readiness Checklist — PRD Genie

*Phase: Setup (Day 3). Maps to reference "Sprint 1: Access, setup, thin slice." A TPM signs off on this
before a team starts building — same logic applies solo: don't discover a broken connection on build day.*

---

## Access & credentials

- [ ] IK-provided OpenAI API key active and tested (one curl/test call succeeds)
- [ ] Ollama running locally, `qwen3.5:4b` pulled and responding to a test prompt
- [ ] `mistral:7b-instruct-q4_K_M` pulled (fallback tier 2, PRD Generator only)
- [ ] n8n instance running, reachable at its local URL
- [ ] Langfuse account created, API keys generated, connected to n8n

## Workflow platform

- [ ] n8n OpenAI node configured and tested with a trivial prompt
- [ ] n8n Ollama node configured and tested against `qwen3.5:4b`
- [ ] Fallback logic sketched: which node/condition routes OpenAI failure → qwen → mistral

## Data

- [ ] All 5 input files loaded into n8n (or confirmed accessible): `sample_meeting_transcripts.txt`,
      `sample_product_brief.txt`, `stakeholder_notes.txt`, `prd_template.md`, `eval_prdgenie_inputs.txt`
- [ ] `prd_template.md` structure confirmed as the exact schema for PRD Generator's system prompt

## Thin-slice proof (Sprint 1's actual deliverable)

- [ ] One agent (Requirement Extractor) built
- [ ] Run against T1 only
- [ ] Output manually verified against T1's "Expected Output Must Contain" criteria
- [ ] **Sign-off:** thin slice works end-to-end — proceed to Baseline phase

## Eval service pre-check (used starting Day 9, confirmed reachable now to avoid a Day-9 surprise)

- [ ] `eval-service/main.py` dependencies installable (`fastapi`, `uvicorn`) — test now, don't wait
- [ ] Cloudflare Tunnel CLI installed and tested with a throwaway local server

---

**Readiness verdict:** ☐ Green — proceed to Baseline phase ☐ Yellow — proceed with noted gaps: ___________ ☐ Red — blocked on: ___________
