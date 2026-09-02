# PRD Genie

AI-powered product documentation assistant for NeuronForge Technologies — ingests meeting
transcripts and stakeholder notes, extracts requirements (separating stated fact from assumption),
generates a structured PRD, breaks it into epics and user stories, and flags gaps for
clarification. Built for the *Applied Agentic AI for PMs/TPMs* capstone.

This repo is organized as an artefact tree, not a code folder — a 14-day engagement produces more
than a workflow file, and each category below answers a different question a reader might have.

## Repository map

| Folder | Answers | Contents |
|---|---|---|
| [`docs/`](docs/) | What is this and why does it exist? | Problem & charter brief, architecture decision records, runbook & handover |
| [`architecture/`](architecture/) | How is it designed? | Ground rules (scoring spec + token estimates), the 14-day roadmap, agent prompts *(added Day 3–8)*, orchestration diagram *(added Day 13)* |
| [`evidence/`](evidence/) | Does it actually work? | Environment readiness, baseline test report, per-cycle logs, the live eval dashboard, screenshots |
| [`eval-service/`](eval-service/) | How is it evaluated, live? | FastAPI + SQLite service bridging n8n → the dashboard, scoring against `architecture/ground-rules.md` |
| [`n8n/`](n8n/) | What actually runs? | Exported workflow JSON *(added Day 3–9)* |

No folder structure is prescribed by the course — this shape is a choice, documented so a TA (or
future-me) doesn't have to guess at the reasoning. See `docs/02-architecture-decision-record.md` if
you want the same treatment applied to the design decisions themselves.

## What's here vs. what lands as the build proceeds

This repo is committed early, on Day 2, so the structure exists before the content does — several
files below are placeholders with a stub explaining what will fill them and on which day (see the
project roadmap: `architecture/roadmap.html`). Checked items are already real:

- [x] `docs/01-problem-charter-brief.md` — Discovery phase (Days 1–2)
- [x] `docs/02-architecture-decision-record.md` — decisions made through Day 9, updated Day 13
- [x] `docs/03-runbook-handover.md` — filled fully on Day 13
- [x] `architecture/ground-rules.md` — scoring spec + token estimates, written Day 2
- [x] `architecture/roadmap.html` — the 14-day plan
- [ ] `architecture/agent-prompts/` — the 4 agents' system prompts, added Days 3–8 as each is built
- [ ] `architecture/diagram.*` — architecture diagram, added Day 13 (a text-form version already
      exists in `architecture/roadmap.html`'s Agent Architecture section)
- [x] `evidence/environment-readiness.md` — Setup phase (Day 3)
- [ ] `evidence/baseline-runs/baseline-test-report.md` — table exists, rows filled Days 4–9
- [ ] `evidence/cycles/` — cycle logs added as each Run→Score→Learn→Change→Re-score iteration
      completes (Days 8–11 minimum, more if time allows)
- [x] `evidence/dashboard.html` — live from Day 10 onward, regenerated on each scheduled refresh
- [ ] `evidence/screenshots/` — workflow canvas, pipeline in action, Langfuse traces — Day 14
- [x] `eval-service/` — built and running from Day 9
- [ ] `n8n/prd-genie-pipeline.json` — exported once the full pipeline is stable, Day 9 onward,
      re-exported on Day 14 as the final submitted version

## Running it

See `eval-service/README.md` for the eval service, and `docs/03-runbook-handover.md` for how to run
the pipeline itself once `n8n/prd-genie-pipeline.json` exists.

## Submission pack

What ships in this public repo, split the way the course itself splits it — items every capstone
project needs, and items specific to PRD Genie.

**Always required** (all 4 capstone projects):
workflow JSON + README · architecture diagram · baseline test results (all 12 inputs, with outputs)
· screenshots (canvas, system in action, observability traces) · 1–2 page architecture write-up ·
cost analysis + 3 production metrics · Q1–Q4 answered · slide deck.

**Project-specific to PRD Genie:**
- **A 5-minute demo video.** Only PRD Genie and SalesGenie require one — CalendarMate and Mira
  don't. This is the single most commonly missed submission item across all four projects; record
  it once the pipeline is stable (roadmap Day 12–13), not as a last-minute Day-14 addition.
- **A stated rollout number, used consistently.** The other three projects each commit to a figure
  in their Q4 reflection (CalendarMate: 50 PMs/TPMs, SalesGenie: 20 reps, Mira: 10 PMs/TPMs). PRD
  Genie's `docs/01-problem-charter-brief.md` states its own figure — reuse that exact number in the
  Q4 reflection rather than restating a different guess from memory.

Full day-by-day mapping of when each piece lands: `architecture/roadmap.html`.

## Mandatory housekeeping

- `.gitignore` excludes `eval-service/eval_results.db`, `.env`, and anything matching `*secret*` or
  `*_key*` — **verify no API key or the eval-service shared secret is committed** before making this
  repo public. `eval-service/main.py`'s `API_KEY` placeholder must be changed and kept out of git,
  not just renamed in place.
- This capstone lives in its own repository, separate from weekly course assignments.
