# Problem & Charter Brief — PRD Genie

*Phase: Discovery (Days 1–2). Merges the course's Q1 (Ideation) and Q2 (Program Charter) into the single
document a TPM would actually produce before kickoff. Fill this before touching the workflow platform.*

---

## 1. Problem summary (3 sentences)

> *(from playbook Step 1 checkpoint)*

- Who: ___________
- What's wasted: ___________
- Biggest risk: ___________

## 2. Pain points → agent solutions (Q1: Ideation, 15 pts)

| # | Manual step today | Agent solution | Input → Output | Risk if AI gets it wrong |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |

## 3. Vision & objectives (Q2: Program Charter, 15 pts)

- **Vision:** ___________
- **Objectives:** ___________
- **In scope:** ___________
- **Out of scope:** ___________
- **Success criteria:** ___________ (e.g., "0% invented requirements on the 12-test baseline set")

## 4. Timeline (compressed engagement shape)

| Phase | Reference shape | Compressed to | Days |
|---|---|---|---|
| Discovery | 4–6 weeks | Problem framing + charter | 1–2 |
| Setup | Sprint 1 — access, setup, thin slice | Tooling + first agent, single input | 3 |
| Baseline | Sprint 2 — ground truth, first baseline | 3 core agents built, full 12-test baseline run | 4–7 |
| The Loop | Sprints 3–5 — most of the value | Extended capability + eval service + 3 documented cycles | 8–11 |
| Hardening / Handover | Sprint 6 | Docs, ADR, runbook, submission package | 12–14 |

## 5. Risks (RAID — Risks only, at charter stage)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Local fallback tier (qwen3.5:4b) hallucinates more readily than OpenAI primary | High | High | Guardrail wording hardened specifically for fallback tier; tracked per model_tier in eval service |
| 14-day window compresses a normally 10–12 week engagement shape | High | Medium | Phases compressed, not skipped — see timeline above; Loop phase still gets the largest time share |
| Eval service (new infra) becomes a bottleneck if it breaks mid-build | Medium | Medium | Keep manual baseline documentation (Day 4–7) independent of the service — it's a value-add layer, not a dependency for core rubric points |

## 6. Stakeholders

| Stakeholder | Role | Interest |
|---|---|---|
| PM/TPM (you) | Owner | Ship a working, evaluated system in 14 days |
| Course TA | Grader | Rubric compliance, evidence of the evaluation loop |
| (Fictional) NeuronForge Eng | Downstream consumer | PRDs must be trustworthy enough to scope from |

## 7. Rollout plan

- Day 14: submission package complete, GitHub repo public
- Post-submission (optional): fine-tune the fallback tier, extend to Version Comparison capability
