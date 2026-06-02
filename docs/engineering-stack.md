# Engineering Stack — AI Business Command Centre

This document explains *why* each part of the stack was chosen. The architecture and diagrams live in
[`architecture.md`](architecture.md); this is the rationale behind the decisions.

---

## Guiding philosophy

The project re-platforms an earlier no-code (Make.com) build into something a mid-to-large organisation
would recognise as production engineering. Every choice favours **type safety, observability, and
managed infrastructure** over cleverness or breadth. Ship one tool end to end, then widen.

---

## The stack, and why

| Layer | Choice | Why this, not the alternative |
|---|---|---|
| Language | Python 3.12 | The mature ecosystem for AI/agent work; modern typing support |
| Packaging | `uv` | Single fast tool for envs and dependencies; replaces pip/venv/poetry juggling |
| Web framework | FastAPI | Async, type-safe, the de-facto Python AI-service framework |
| Agent framework | Pydantic AI | Type-safe tool calling and model-agnostic, with native Logfire tracing — far less ceremony than a graph framework for a single agent |
| Models | Claude Haiku → Sonnet | Cheap, fast routing on Haiku; auto-escalate to Sonnet for multi-step reasoning |
| Memory | Upstash Redis | Managed cloud Redis with TTL; removes the local Docker dependency entirely |
| Long-term store | Supabase Postgres | Managed Postgres for preferences, audit log and embeddings; generous free tier |
| Channel SDK | Slack Bolt for Python | The official, well-documented SDK |
| Hosting | Railway | Git-push deploys with first-class managed Redis/Postgres add-ons |
| Observability | Logfire | Zero-config tracing that plugs straight into Pydantic AI |
| Evaluation | Pydantic Evals | LLM-as-judge harness that runs in CI alongside tests |

---

## Why Pydantic AI over a graph framework

For a single agent with five tools, a graph-based framework adds concepts and configuration that earn
their place only once you genuinely need multi-agent orchestration. Pydantic AI keeps the surface area
small: define typed tools, give the agent a system prompt, and tracing comes for free. The decision was
to start with the lightest thing that is still production-grade, and only reach for more when the problem
demands it.

---

## Why managed infrastructure over local

A concrete example from the build: Docker Desktop failed to install on Windows 11 due to a permissions
issue on a leftover data folder. Rather than burn a session debugging the host, the design switched to
Upstash Redis — managed cloud Redis used for both local development and production. The async Redis client
treats `redis://` and `rediss://` URLs identically, so it was a configuration change, not a code change.

The principle: when a managed equivalent exists, removing an environment-specific failure mode is worth
more than the satisfaction of fixing it locally.

---

## Observability and evaluation as first-class

Tracing is wired in from the first commit, not retrofitted: every inbound message, model call, tool call
and reply is captured. Evaluation runs in CI using an LLM-as-judge over a fixed set of representative
requests, scoring whether the agent picked the right tool and produced a correct, well-formed answer. A
regression fails the build the same way a unit test would. This is the difference between a demo and
something you can operate.

---

## Migration path

The data backend starts on Google Sheets (parity with the prior no-code build) and migrates to Postgres
one tool at a time, so the system stays working throughout rather than requiring a big-bang cutover.

> Australian English throughout.
