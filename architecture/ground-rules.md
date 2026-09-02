# PRD Genie — Ground Rules (Scoring Spec)

This is the "ground truth" step 2 of the evaluation loop scores against. It is the 12 baseline tests
from `eval_prdgenie_inputs.txt`, expanded into pass/fail criteria precise enough to log automatically
(see `eval-service/main.py`'s `ResultIn` schema) and to fill the cycle-documentation template.

Every rule is written as: **PASS if**, **FAIL if**. A test can pass on required content and still be
logged as a hallucination if it also adds anything not in the source — these are tracked as two
separate fields (`passed`, `hallucination_detected`), not folded into one bit.

---

## Capability: Requirement Extraction

| Test | Input type | PASS if output contains | FAIL if output |
|------|-----------|--------------------------|-----------------|
| T1 | Detailed | filter by date/category/status; 2-second load time; PM Sarah; Q3 deadline | omits any of the four, OR adds a requirement not in the transcript |
| T2 | Vague | flags as ambiguous; lists missing info (metrics, format, users) | invents specific requirements not stated |
| T3 | Contradictory | identifies the refresh-rate vs. minimize-API-calls conflict; flags for stakeholder resolution | silently resolves the conflict in favor of one side |
| T5 | Incomplete | flags as insufficient; lists what's missing (dashboard scope, "real-time" definition, budget) | fills gaps with assumptions |
| T6 | Multi-stakeholder | captures all 3 viewpoints (Eng/microservices, Design/SPA, PM/March); identifies the tension | favors or drops any one viewpoint |
| T7 | Technical | exact numbers preserved: 10,000 users, 200ms p95, Salesforce REST API v52; classified as NFRs | rounds or modifies any number |
| T8 | Persona-heavy | 3 distinct personas (Admin, End User, Auditor), separate stories per persona | merges personas into generic "users" |
| T9 | Empty | flags "no requirements extractable"; produces no PRD content | generates any PRD-shaped content from nothing |
| T10 | Dependency | extracts SSO feature; flags dependency on Team Alpha's auth service; flags unknown ETA as a risk | omits the dependency or invents an ETA |

## Capability: PRD Generation

| Test | PASS if | FAIL if |
|------|---------|---------|
| T4 | Acceptance criteria extracted verbatim (PDF logo, CSV formula preservation); User Stories generated with exactly these criteria | adds criteria not stated, or paraphrases the AC away from verbatim |
| T11 | Follows `prd_template.md` structure section-for-section; every section traceable to T1's extraction; no section invented from general PRD "best practice" | skips a template section, reorders it, or fills a section with generic filler |

## Capability: Epic / User Story Breakdown

| Test | PASS if | FAIL if |
|------|---------|---------|
| T12 | Epics/User Stories in "As a [persona], I want…" format; each carries a Must/Should/Nice priority | missing priority tags, or format drifts from the "As a…" pattern |

## Capability: Gap Analysis (extended)

Not in the numbered baseline set — score against the same transcripts using this rule instead:

**PASS if**, for any input the Requirement Extractor flagged as ambiguous/incomplete (T2, T5, T9), the
Gap Analyzer produces a numbered list of concrete, stakeholder-answerable questions.
**FAIL if** it produces vague prompts ("please clarify requirements") instead of specific questions, or
invents an answer instead of asking.

---

## Cross-cutting rules (apply to every test)

1. **Groundedness**: every claim in the output must be traceable to a specific line in the source input.
   If it can't be pointed to, it's a hallucination — log `hallucination_detected: true` even if the test
   otherwise passes its keyword check.
2. **UNKNOWN over invention**: when a field can't be determined, the correct output is an explicit
   `UNKNOWN` / open-question marker — never a plausible-sounding guess.
3. **Model-tier neutrality**: the rule is identical regardless of which tier served the response
   (OpenAI primary, qwen3.5:4b fallback, mistral:7b-instruct second fallback). Record which tier served
   each result — a pass/fail split by tier is itself a metric, not just a routing detail.

---

## Production metrics (derived from these rules, tracked over time)

Split into two tiers deliberately — a TA or a stakeholder reading the dashboard should be able to
answer "does it work" (AI/technical) and "is it worth it" (business) from two different sections,
not have to derive the second from the first.

### AI / Technical metrics

- **Extraction completeness** — passed / total, Requirement Extraction tests only
- **Hallucination rate** — hallucination_detected / total, across all tests. The single most
  important number in this project — it's the explicit risk named in the problem statement.
- **Format compliance** — passed / total, PRD Generation + Story Breakdown tests only
- **Fallback rate** — % of runs served by qwen3.5:4b or mistral rather than OpenAI primary
- **Latency (p50 / p95)** — per agent and end-to-end, from `latency_ms` in each result
- **Trend across cycles** — is hallucination rate actually decreasing cycle over cycle, or plateauing

### Business metrics

- **Time saved per PRD** — baseline manual time (hours, per stakeholder notes: "hours translating
  transcripts to PRDs") vs. pipeline time-to-draft-PRD (from trace latency). Report as an estimated
  hours-saved figure, not just a ratio — this is the number a PM/stakeholder actually cares about.
- **Cost per PRD generated** — OpenAI token cost for the run, plus $0 marginal noted separately for
  any run served by the local fallback tier (see roadmap Day 13 cost analysis for the full model).
- **Draft-readiness rate** — % of generated PRDs that passed with zero hallucinations and needed no
  human correction before being usable as a first draft (i.e., "must a PM re-read this line by line,
  or can they trust the draft and just refine it"). This is extraction completeness reframed for a
  non-technical reader, deliberately in plain language.
- **Requirement-loss prevented** — for inputs like T5/T9 (incomplete/empty notes), did the system
  correctly surface the gap instead of silently losing it — this is the literal problem NeuronForge
  named ("meeting transcripts and notes sit in documents nobody revisits — valuable requirements get
  lost"), so a correct "insufficient input" flag counts as a business win, not just a technical pass.
- **Adoption signal (directional, not measurable from the baseline set alone)** — note in the writeup
  as a forward-looking metric you'd track in real production: % of AI-generated PRDs a PM ships with
  only minor edits vs. rewrites from scratch.

---

## Token estimates (set before build — Day 2, revised only with a reason)

*Optimization needs a target set in advance, not a number derived after the fact from whatever the
first run happened to cost. These are written now, grounded in the actual input files' measured size
(chars/4 ≈ tokens, English average), before Day 3's build starts. The dashboard tracks actual usage
against these — see "Estimate vs. actual" below — and Day 11+ optimization work targets whichever
agent shows the largest variance, not a guess.*

### Input sizes, measured (not estimated) from the provided files

| Source | Chars | Est. tokens (÷4) |
|---|---|---|
| Baseline test inputs (T1–T10), average | ~93 | ~23 |
| Baseline test inputs (T1–T10), range | 30–141 | 7–35 |
| `sample_meeting_transcripts.txt`, per transcript, average | ~722 | ~180 |
| `sample_product_brief.txt` (full) | 1,603 | ~400 |
| `stakeholder_notes.txt` (full) | 2,415 | ~603 |
| `prd_template.md` (full — injected into PRD Generator's prompt every call) | 1,522 | ~380 |

The raw test inputs are small. **System-prompt overhead — the ROLE/INPUT/OUTPUT/RULES guardrail text
plus, for PRD Generator, the full template — dominates per-call cost far more than the input data
does.** This is the first optimization target: a verbose guardrail prompt costs tokens on every single
call, 12+ times per baseline pass, regardless of how small the actual input is.

### Per-agent estimate (single baseline-test call, T1-sized input)

| Agent | Est. system prompt | Est. input | Est. output | Est. total/call |
|---|---|---|---|---|
| Requirement Extractor | ~350 | ~30 (test input) or ~180 (full transcript) | ~250 (extracted fields + stated/ambiguous split) | ~630–780 |
| PRD Generator | ~300 + **380 (template)** = ~680 | ~250 (extractor output) | ~700 (full PRD, 10 sections) | ~1,630 |
| Story Breakdown | ~300 | ~700 (PRD input) | ~450 (epics + stories + priorities) | ~1,450 |
| Gap Analyzer | ~300 | ~250 (ambiguous-items list) | ~150 (clarification questions) | ~700 |

**Estimated full pipeline (4 agents, one input, one pass):** ~4,400–4,600 tokens
**Estimated full 12-test baseline pass (all agents, all applicable tests):** ~35,000–45,000 tokens
*(not 12× the single-call estimate — T2/T5/T9 short-circuit early since they correctly stop at
"insufficient," so PRD Generator/Story Breakdown never run on those 3 inputs)*

### Where to optimize, in priority order

1. **PRD Generator's system prompt** — largest single-call cost (template injection alone is ~380
   tokens, repeated on every call including reruns). First candidate: cache/shorten the template
   representation, or reference sections by ID instead of re-sending full section text.
2. **Guardrail verbosity across all 4 agents** — the UNKNOWN/hallucination rules need to be strict
   (see Tool Stack note on fallback-tier guardrail strength) but strict doesn't require long; a
   tightened prompt that's equally firm costs less per call, times every call.
3. **Story Breakdown's input** — currently the full PRD text; consider whether it needs the entire
   PRD or just the Feature Requirements section.

### Estimate vs. actual (tracked live on the dashboard)

Each eval-service result logs actual `input_tokens`/`output_tokens` per agent per run (all 3 model
tiers, not just OpenAI — see `eval-service/main.py`). The dashboard shows estimate vs. actual per
agent with a variance %, so the Day 11+ optimization cycles target the agent with the largest gap,
not the one that's easiest to change. A large positive variance (actual >> estimate) on a specific
agent is itself a Learn-stage finding worth writing into that cycle's log.
