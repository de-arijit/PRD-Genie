# Architecture Decision Record (ADR) — PRD Genie

*Phase: Hardening/Handover (Days 12–14). An ADR captures a decision, its alternatives, and why the
alternatives were rejected — the format a TPM leaves behind so the next person doesn't silently re-litigate
a settled choice. One entry per significant decision; add more as needed.*

---

## ADR-001: Sequential Pipeline orchestration

**Status:** Accepted
**Context:** Every PRD Genie input follows the identical route — transcript → requirements → PRD → stories.
**Decision:** Sequential Pipeline, not Router/Dispatcher or Hierarchical.
**Alternatives considered:**
- *Router/Dispatcher* — rejected. There's no varied input-type routing need; every input takes the same path. Router pattern fits CalendarMate/Mira, not this project.
- *Hierarchical* — rejected. No single request decomposes into parallel subtasks that need coordinating; the 4 agents are already a strict linear dependency chain.
**Consequences:** Simple to reason about and debug (one agent's output is the next agent's whole input). Trade-off: no parallelism, so latency is additive across all 4 agents.

## ADR-002: OpenAI primary, local (Ollama) fallback chain

**Status:** Accepted
**Context:** IK provided an OpenAI key; local models (qwen3.5:4b, mistral:7b-instruct) are already running via Ollama from prior projects.
**Decision:** OpenAI (gpt-4o-mini default) as primary for all 4 agents; qwen3.5:4b as first fallback; mistral:7b-instruct as second fallback, PRD Generator only.
**Alternatives considered:**
- *Local-only* — rejected as primary. Smaller local models are measurably more prone to filling gaps instead of saying UNKNOWN (see Cycle 1–3 logs) — a worse fit for a project whose core risk IS hallucination.
- *OpenAI-only, no fallback* — rejected. No resilience if the key is rate-limited/unavailable; local fallback keeps the pipeline runnable offline.
**Consequences:** Two-tier cost model (token cost + $0 marginal on fallback runs) instead of one. Requires tracking which tier served each run — added as a first-class field in the eval service, not an afterthought.

## ADR-003: OpenTelemetry — documented, not implemented

**Status:** Accepted (deferred)
**Context:** OTel was raised as a desired direction for the eval/observability layer.
**Decision:** Stay on Langfuse's native SDK for this submission; document the OTel upgrade path rather than building it.
**Alternatives considered:**
- *Full OTel Collector → Langfuse (OTLp ingestion)* — rejected for this cycle on time budget alone, not on merit. Real value (vendor-neutral instrumentation) but not worth the build-day cost against a 14-day deadline.
**Consequences:** Current instrumentation is Langfuse-specific (SDK, not OTel spec) — a future migration means re-instrumenting call sites, not just swapping a backend.

## ADR-004: Eval service as a separate FastAPI + SQLite bridge, not built into n8n/Langfuse directly

**Status:** Accepted
**Context:** Needed ground-truth scoring, a cycle log, and both technical and business metrics — none of which Langfuse computes natively.
**Decision:** Small standalone FastAPI service, SQLite-backed, exposed via Cloudflare Tunnel; n8n posts to it, a scheduled task reads its `/summary`.
**Alternatives considered:**
- *Build metrics directly into Langfuse's scoring feature* — rejected. Ground-truth scoring rules here are project-specific (12 named tests + a Gap Analysis rubric); a purpose-built endpoint is simpler than fighting a general-purpose tool's scoring model into this shape.
- *Managed backend (Supabase)* — rejected in favor of self-hosting, to stay consistent with the local-first/privacy-first principle already applied elsewhere in the stack.
**Consequences:** New infrastructure to run and keep alive (tunnel uptime is a real dependency); documented explicitly in the Environment Readiness checklist so it isn't discovered broken late.

---

*Add ADR-005+ here for any decision made during the Loop phase that a future reader would otherwise have to reverse-engineer from prompt diffs alone.*
