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

## ADR-005: n8n's OpenAI node — use the `text`/`response` operation, not `message`

**Status:** Accepted
**Context:** This n8n instance runs the `@n8n/n8n-nodes-langchain.openAi` node at v2.3. The node's
own bundled source rejects the `message` operation (valid on v1) as invalid on v2 — only
`resource: text` / `operation: response` (the Responses API) passes validation. Discovered while
wiring the Setup-phase thin-slice proof; not documented anywhere obvious in n8n's own UI error
messaging, so it cost real debugging time.
**Decision:** Every OpenAI node built for this pipeline (all 4 agents) uses `resource: text`,
`operation: response`. Do not reintroduce `message` — it will fail node validation on this n8n
version.
**Alternatives considered:**
- *Pin n8n to an older version supporting `message`* — rejected. No reason to fight the installed
  version; `response` works identically for this project's single-turn, no-conversation-history
  use case.
**Consequences:** None functionally — this is a compatibility note, not a design trade-off. Recorded
here so agents 2–4 (PRD Generator, Story Breakdown, Gap Analyzer) don't rediscover the same node
validation error from scratch during Baseline phase.

## ADR-006: Story Breakdown carries verbatim acceptance criteria in a separate line, not inside the story sentence

**Status:** Accepted
**Context:** The corrected `ground-rules.md` requires T4's Story-Breakdown-stage output to carry
PRD Section 5's acceptance criteria forward "verbatim, unparaphrased." But every story is also
required (T12, and Story Breakdown's own Rule 3) to follow the fixed sentence
"As a [persona], I want [goal], so that [benefit]." — a format that structurally requires
rephrasing a declarative criterion ("PDF must include company logo") into a first-person want
clause ("I want the PDF to include the company logo"). Cycle 1's re-score confirmed this tension is
real, not hypothetical: the story sentence correctly reflected each requirement, but the wording
was paraphrased by construction, failing T4's story-stage verbatim check.
**Decision:** Keep the "As a…, I want…, so that…" sentence exactly as-is — it's allowed to
paraphrase, and should, for readability. Add a second, separate line under each story,
`Acceptance Criteria: "<verbatim text>"`, sourced from PRD Section 5 and copied
character-for-character. The two lines have different jobs: the sentence communicates intent in
natural language, the criteria line preserves the exact source wording for traceability. Story
Breakdown's INPUT section was extended to read PRD Section 5 (previously explicitly excluded), and
a new Rule 4 states the separation directly: the sentence may paraphrase, the criteria line may
not.
**Alternatives considered:**
- *Relax the ground rule's "unparaphrased" wording for the story stage specifically* — rejected.
  The verbatim requirement is a direct extension of T4's own Section-5 rule and the project's core
  hallucination-prevention theme; loosening it because the story format is inconvenient trades away
  the thing this project is scored on, for the sake of prompt convenience.
- *Drop the "As a…, I want…" format for T4-derived stories only, replacing it with the raw
  criterion* — rejected. T12's format rule applies uniformly across all stories; carving out a
  format exception for one requirement's origin (T4 vs. any other PRD) would make Story Breakdown's
  output inconsistent depending on upstream provenance, and downstream consumers (a human reading
  the story list) would see two different story shapes for no visible reason.
- *Embed the verbatim quote inside the "I want" clause itself* (e.g. "I want the PDF to include the
  company logo, exactly as stated: 'PDF must include company logo'") — rejected. This produces an
  awkward, redundant sentence and still risks the model paraphrasing the embedded quote under
  pressure to keep the sentence readable — the whole point of a separate line is that it isn't
  competing with prose-quality pressure.
**Consequences:** Every story's output is one line longer when a Section 5 criterion exists for its
requirement (most will; T9/T2/T5-style insufficient inputs that never reach Story Breakdown are
unaffected). Slightly more output tokens per story — acceptable given this project's cost priority
is Section-5-template-injection-driven PRD Generator cost (see `ground-rules.md`'s token-estimate
section), not Story Breakdown's. Re-scored once against T4's story stage after the change — see
`evidence/cycles/cycle-02.md` for the confirming result.

---

*Add ADR-007+ here for any decision made during the Loop phase that a future reader would otherwise have to reverse-engineer from prompt diffs alone.*
