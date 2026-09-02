# Agent prompts

The 4 system prompts for PRD Genie's agents, one file each:

- [x] `01-requirement-extractor.md` — Day 3 (thin-slice proof), passed against T1
- [ ] `02-prd-generator.md` — Baseline phase
- [ ] `03-story-breakdown.md` — Baseline phase
- [ ] `04-gap-analyzer.md` — Baseline phase (extended capability)

Each follows the ROLE / INPUT / OUTPUT / RULES structure from the course playbook, with the
hallucination guardrail (UNKNOWN-over-invention) written to hold on the qwen3.5:4b and
mistral:7b-instruct fallback tiers specifically, not just the OpenAI primary tier — see
`../ground-rules.md` and `../roadmap.html`'s Tool Stack section for why that distinction matters
here.

Every prompt revision that survives a Run→Score→Learn→Change→Re-score cycle should be committed
here as the new version, with the before/after diff preserved in that cycle's log under
`../../evidence/cycles/` rather than only living in git history — the cycle log is what a reader
checks to understand *why* a prompt changed, not just *that* it did.

## Wiring these into n8n

Each prompt is the system message on that agent's OpenAI node. This n8n instance runs the
`@n8n/n8n-nodes-langchain.openAi` node at v2.3, which requires `resource: text` /
`operation: response` — the older `message` operation (valid on v1) fails validation here. See
`docs/02-architecture-decision-record.md`'s ADR-005 for the full note. All 4 agents' nodes should
use the same operation for consistency.
