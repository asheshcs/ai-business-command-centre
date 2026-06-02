# AI Business Command Centre

> A production-grade, multi-channel AI assistant that exposes core business operations as agent tools — with tracing and an evaluation harness built in from day one.

<p>
  <img src="https://img.shields.io/badge/status-week_1_shipped-brightgreen?style=flat-square"/>
  <img src="https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pydantic_AI-E92063?style=flat-square&logo=pydantic&logoColor=white"/>
  <img src="https://img.shields.io/badge/Logfire-E92063?style=flat-square&logo=pydantic&logoColor=white"/>
</p>

---

## The problem

A small business runs on a handful of repetitive operational questions and actions — *what's in the
pipeline, send that invoice, book that meeting, post a status, pull the weekly report.* These usually
live across disconnected tools and a person's head. This project turns them into a single assistant
you can talk to from the channels you already use.

## What this is

A re-platforming of an earlier no-code (Make.com) build into a real engineering stack. An agent
receives a natural-language request from a channel (Slack first), reasons about it, and calls the
right business tool — then formats the answer back into the channel. Every step is traced, secrets
are handled properly, and an evaluation framework runs in CI.

## The five tools

| Tool | What it does |
|---|---|
| `check_pipeline` | Reads the live sales pipeline and answers queries like *"deals over $100k closing this month"* |
| `send_invoice` | Creates an invoice entry and sends it to a client |
| `schedule_meeting` | Checks availability and books a meeting |
| `post_update` | Posts a status message to a channel |
| `pull_report` | Returns the latest weekly KPI summary |

## Architecture

```
channel webhook ─▶ FastAPI ─▶ adapter (normalise)
                                  │
                                  ▼
                       load Redis conversation memory
                                  │
                                  ▼
                   Pydantic AI agent  (Claude Haiku, escalates to Sonnet)
                                  │  tool call
                                  ▼
                   Python tool fn ─▶ data backend (Sheets ▸ Postgres)
                                  │
                                  ▼
                 adapter (format) ─▶ back to channel   │  every step traced in Logfire
```

## Tech stack

| Layer | Choice | Why |
|---|---|---|
| Language / packaging | Python 3.12, `uv` | Fast, single-tool dependency management |
| Web framework | FastAPI | Async, type-safe, the Python AI-service default |
| Agent framework | Pydantic AI | Type-safe tool calling, model-agnostic, native Logfire |
| Primary / fallback model | Claude Haiku → Sonnet | Cheap routing, auto-escalation for multi-step reasoning |
| Memory | Upstash Redis | Per-user context with TTL; no local Docker needed |
| Long-term store | Supabase Postgres | Preferences, audit log, embeddings |
| Hosting | Railway | Git-push deploys, first-class Redis/Postgres add-ons |
| Observability | Logfire | Native Pydantic AI instrumentation |
| Evaluation | Pydantic Evals | LLM-as-judge, wired into CI |

## What this project demonstrates

- A clean migration path from no-code prototype to a stack a mid-to-large org would recognise as legitimate.
- Treating **observability and evaluation as first-class**, not afterthoughts — tracing on every call and
  an LLM-as-judge eval gate in CI.
- Pragmatic engineering decisions documented in full (e.g. dropping Docker on Windows in favour of a
  cloud Redis to remove a class of environment failures).

## Documentation

- [`docs/architecture.md`](docs/architecture.md) — full design, diagrams, request lifecycle, and the
  observability/evaluation approach.
- `docs/engineering-stack.md` — full stack rationale and architecture.
- `docs/week-1-playbook.md` — a 16-phase, end-to-end build playbook: prerequisites → accounts → service
  accounts → app creation → bootstrap → application code → local test → deploy → security → observability → evals.

> Built with Claude Code. Documentation in Australian English.
