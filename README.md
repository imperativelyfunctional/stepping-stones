# Stepping Stones

**Post-call coaching for contact-centre agents on templated calls.**

After each call, Stepping Stones shows which required steps the agent completed, flags the ones they missed, scores the call, and tracks improvement over time.

Team **TOo BAD** · UofT Focus in Technology Leadership (FTL) Cohort 9 × Genesys (with CSC207 Software Design)

[Figma UI mockup](https://www.figma.com/design/tgQYPY631Tg5njDsoBYGNJ/Stepping-Stones-%E2%80%94-UI-Mockup?node-id=12-99&p=f)

---

## Overview

### The problem

Feedback to contact-centre agents today is:

- **Late**: it arrives long after the call.
- **Vague**: "be more empathetic".
- **Sampled**: QA listens to only a few calls.

A call can sound polite, and even be rated 10/10 by the customer, and still skip required steps: never confirming the outcome, skipping a check, or logging the wrong details.

> Real example (pay bill): the customer rated the call 10/10. The agent never verified identity, never read the amount back, and logged $422 when the customer said $122.

### Who it helps

- **New hires**: learn call structure faster.
- **Current agents**: stay solid on uncommon or easy-to-skip steps.
- **Coaches / QA**: measurable step completion, not just soft ratings.

### Where it fits

The Quality Management & Coaching stage: **after** the transcript exists and **before** the agent's next shift. It is not live during the call.

---

## Goal

From the Genesys brief:

> Using transcripts, metadata, and performance metrics from contact-centre interactions, create a digital coaching tool that helps an organization's customer service agents independently improve their own performance against configurable quality expectations, and verify that improvement over time.

After each call the agent gets:

1. Which **required steps** they completed
2. Which important steps they **missed** (flags)
3. A **score** based on those steps
4. Over time, whether their **step-completion rate is improving**

Of Genesys's five axes of "performing well" (Compliance, Process, Communication, Efficiency, Outcome), we are strongest on **Process**, and also cover Compliance and Outcome.

---

## Features

### MVP

- Templates for 1–3 common task types (linear, no branching)
- Load an agent's calls (HVB JSON)
- Post-call summary: completed / missed steps, score, flags
- Timeline of an agent's calls with completion rate
- Detection with **explainable rules** (keywords, patterns, logged fields), not an LLM grade

### Later (phase 2+)

- **Peer Model Answer**: for a missed step, show how another agent did it on the same task type
- Weighted / critical steps
- Conversation branches
- More task types and better detection (e.g. semantic matching with local embeddings)

**Why rules first:** agents need to trust a flag, and managers need to be able to configure it. A rule can always say *why* it fired.

---

## How it works: templates and scoring

A **template** is, for one `task_type`, an ordered list of required steps that must be evidenced in the call for it to count as correctly handled. Steps are things the **agent** does, not the caller. Templates are written by us as configurable quality expectations (like a QA rubric); they do not come with the dataset.

| Step field | Meaning |
| --- | --- |
| `id` | e.g. `confirm_ordered` |
| `name` | Short UI label |
| `description` | What "done" means |
| `required` | true/false (MVP: all required) |
| `evidence_rules` | How we detect it: agent wording (keywords/patterns), logged fields in `captured_by_agent`, optional dialog-act hints |

**Scoring a call**

1. Read the call's task type and load that task's template.
2. Scan the agent's turns, plus `captured_by_agent` where useful.
3. For each required step: evidence found → *Completed*, otherwise *Missed*.
4. Score, then flags: every missed required step becomes a flag in the post-call summary.

Two scoring models are being considered (open decision):

- **Equal weight (MVP docs):** score = completed ÷ required. Example: 3 of 4 steps = 75%.
- **Weighted, out of 100 (Figma):** every call starts at 100; each step is worth some points; a missed or wrong step loses its points. Severity bands: ≤5 slip, 6–10 inaccuracy, 11–20 mistake, >20 critical.

**Templates today:** Transfer money (7 steps, in PR #12), Replace card (4 steps, in docs/Figma). Pay bill is a candidate third template.

---

## Dataset: HarperValleyBank (HVB)

Simulated bank contact-centre calls (role-play): about 1,446 calls from 58 agents. HVB gives us the calls and labels; **we add** the templates, the checker, the score and the coaching UI.

**What it gives us**

- Task type on every call (8 types: replace card, transfer money, check balance, order checks, pay bill, reset password, schedule appointment, get branch hours)
- Agent and customer turns separated, plus agent IDs
- Human and machine (ASR) transcripts per turn
- Dialog-act tags per turn (hints, not the checklist)
- `customer_record.captured_by_agent`: what the agent logged during the call
- `task_assigned` vs `task_logged_by_agent`
- Per-turn timing

**What it doesn't give us**

- No "correct procedure" checklist, so we write the templates.
- No coaching score.
- Only about 4 real days of timeline. Progress charts use **synthetic coaching weeks**, clearly labelled "Demo data".
- Role-play calls, not live production data.

> **Project rule:** don't invent dataset fields. If a feature needs data HVB doesn't have, it is either synthetic and labelled as such, or out of scope.

**Citation:** Wu et al. 2020, *Harper Valley Bank*, arXiv:2010.13929, licensed CC BY 4.0.

---

## UI design

Designed in Figma: [Stepping Stones — UI Mockup](https://www.figma.com/design/tgQYPY631Tg5njDsoBYGNJ/Stepping-Stones-%E2%80%94-UI-Mockup?node-id=12-99&p=f). Every screen shares a left nav with **My calls · Progress · Procedures**.

| Screen | What it's for |
| --- | --- |
| Home / Your week | Weekly score, the one tip that matters most, next steps, what's going well, recent calls |
| Call review | The core post-call screen: score ring, transcript with evidence, required-steps path, step detail, flags |
| Procedures | Each task type's template: steps, points, what to say, logged fields |
| Progress | Weekly step-completion chart (synthetic weeks), by call type, steps you skip most |

**Design language:** one stone = one step on a call. A stone's *shape* says what happened, so meaning never depends on colour alone.

| State | Look | Meaning |
| --- | --- | --- |
| Completed | Solid green | Evidence found in the transcript |
| Missed | Cracked, red outline | No evidence; counts against the score and shows as a flag |
| Wrong | Red with ✕ | Done, but incorrectly (e.g. read back reversed) |
| Under review | Dashed outline | Agent pressed "I did do this"; neutral until checked |
| Procedure step | Solid brand teal | A step as written in a procedure |

**Score bands:** On track 85–100 · Getting there 60–84 · Focus here under 60.

**Copy tone:** second person, encouraging, specific.

---

## Tech stack

**Python · FastAPI · PostgreSQL · React (Vite)**

```
data/sample_calls.json ──(seed script)──► PostgreSQL :5432 ◄── SQL ──► FastAPI :8000 ◄── JSON/HTTP ──► React :5173
```

**Why this stack**

1. **One backend language:** data loading, step detection and the API are all Python.
2. **Detection is text processing** (keywords, patterns, logged-field checks), which Python keeps short.
3. **The data is relational** (agents → calls → step results → weekly progress), which suits SQL. Postgres still stores loose JSON like `captured_by_agent`.
4. **FastAPI documents itself:** `/docs` lists and tests every endpoint.
5. **React with Vite** for the frontend.

Postgres 16 runs via `docker-compose.yml`.

---

## Getting started

See [`docs/SETUP.md`](docs/SETUP.md) for setup instructions. More detail on the stack is in [`docs/TECH_STACK.md`](docs/TECH_STACK.md), and the page-by-page UI description is in [`docs/UI_OUTLINE.md`](docs/UI_OUTLINE.md).

## Project structure

```
backend/
  app/            main, config, database, models, schemas, routers/
  scripts/seed.py
  tests/
frontend/
  src/            App, pages/, components/, api/, hooks/, index.css
data/
  sample_calls.json
docs/
  TECH_STACK.md  SETUP.md  UI_OUTLINE.md
docker-compose.yml
README.md
```

---

## How we work

Work is tracked on a GitHub Project board using **Epic → Feature → Task** (GitHub sub-issues):

- **Epic**: a page or system (e.g. "Call review page")
- **Feature**: one section or capability of it
- **Task**: one PR, one owner, 1–3 days

Branches are named `feat/<issue#>-short-name`, `docs/…`, `fix/…`. PR bodies start with `Closes #N`, each PR gets one reviewer, and UI PRs include a screenshot next to the Figma frame.

## Contributors

| Name | Role | GitHub |
| --- | --- | --- |
| Brandon | Lead, UI design | [@brandonli5211](https://github.com/brandonli5211) |
| Diego | Templates & data | [@DiegoFigarella](https://github.com/DiegoFigarella) |
| Ozan | Scoring engine | [@ozantimur](https://github.com/ozantimur) |
| Anton / Tugra | _fill in_ | [@thlasagna](https://github.com/thlasagna) / [@TroubleVaser](https://github.com/TroubleVaser) |

## Links

- Figma: [Stepping Stones — UI Mockup](https://www.figma.com/design/tgQYPY631Tg5njDsoBYGNJ/Stepping-Stones-%E2%80%94-UI-Mockup?node-id=12-99&p=f)
- Dataset: [gridspace-stanford-harper-valley](https://github.com/cricketclub/gridspace-stanford-harper-valley)
- Industry partner: Genesys, Workforce Engagement Management (WEM) team
