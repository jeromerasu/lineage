# ADR-001: Tech stack — backend language

**Date:** 2026-06-17 (revised 2026-06-17)
**Status:** Accepted (supersedes initial Java+Python split decision)

---

## Context

Lineage needs four backend services: a REST API, an async analysis worker (LLM + graph traversal), a real-time sync server (WebSocket), and a genetics file parser. The founder's primary backend experience is Java/Spring Boot (11 years across Oracle Cloud + KataFit). Python is a target learning area for career diversification and resume signal in the healthtech / AI hiring market.

An earlier draft of this ADR proposed Java for three services and Python for one (`lineage-analysis-worker`). After reviewing the full service map, the decision was revised to all-Python.

---

## Decision

All four backend services in **Python + FastAPI**.

| Service | Framework | Key libraries |
|---|---|---|
| `lineage-api` | FastAPI + SQLAlchemy 2.0 (async) + Alembic | REST + auth + persistence |
| `lineage-analysis-worker` | FastAPI + Celery/Dramatiq + Anthropic SDK | LLM analysis + graph traversal |
| `lineage-sync-server` | Starlette / FastAPI WebSocket + uvicorn workers | Real-time canvas (asyncio) |
| `lineage-genetics-parser` | FastAPI + biopython + cyvcf2 | Genome file parsing |

**Tooling stack:**

| Tool | Role |
|---|---|
| `uv` | Package management (replaces Poetry/pip; faster resolution) |
| `pyproject.toml` | Project config per service |
| Pydantic v2 | DTOs — snake_case wire discipline natively supported |
| SQLAlchemy 2.0 (async) | ORM for `lineage-api` |
| Alembic | Schema migrations |
| `pytest` + `pytest-asyncio` | Test framework |
| Ruff | Lint + format (replaces black + isort + flake8) |
| mypy --strict | Type-checking |
| Python 3.12+ | Runtime |

---

## Rationale

1. **Ecosystem fit is genuine, not marginal.** `biopython` + `cyvcf2` are the dominant libraries for genome file parsing — re-implementing equivalent functionality in Java is real wheel-reinvention. `networkx` + `pandas` for graph traversal and cohort analysis are best-in-class. Anthropic's Python SDK is more mature than its Java SDK as of 2026.

2. **Operational simplicity of single-language backend.** One mental model, one dependency manager (`uv`), one test framework (`pytest`), one type-checker (`mypy`). A two-language backend doubles the context-switching cost without proportional benefit — especially on a solo project where the same engineer owns all services.

3. **Resume signal.** "Rebuilt full backend in Python with FastAPI + Anthropic + biopython" is a stronger career narrative than "added one Python service to a Java codebase." Aligns with the shape of modern healthtech / AI hiring. Java depth is already demonstrated at Oracle Cloud + KataFit + byb; Python depth is the gap to close.

4. **Python in 2026 is enterprise-grade.** `async/await` is mature, type hints with `mypy --strict` approach Java's compile-time guarantees, SQLAlchemy 2.0 + Alembic is a solid ORM + migration story, `uvicorn` + `gunicorn` is a proven ASGI deployment pattern.

---

## Consequences

### Positive

- Single backend language; lower cognitive overhead for solo-founder context switching
- Best-of-class ecosystem libraries for the AI / genetic-data / graph workloads
- Career signal: "Senior backend engineer with Java + Python depth" — covers both enterprise and AI/ML hiring markets
- `uv` + Ruff + mypy --strict gives a modern, fast, type-safe Python DX that is not the Python of 2018
- Pydantic v2 DTO validation with snake_case natively avoids the camelCase/snake_case wire discipline issues that bit byb (commit 51d2289)

### Negative

- Initial dev velocity slower than Java for the first 2–3 weeks while ramping on FastAPI patterns (dependency injection, lifespan events, SQLAlchemy 2.0 async session management)
- Less "batteries included" than Spring Boot — auth, distributed tracing, observability require more deliberate assembly
- GIL constraints on CPU-bound work — accepted; Lineage workloads are I/O-bound (LLM HTTP calls, DB queries, WebSocket fanout). CPU-bound genome parsing in `lineage-genetics-parser` runs in a process pool (`ProcessPoolExecutor`) to sidestep the GIL
- Less mature JVM-style profiling tooling — `py-spy` and `cProfile` are adequate for Lineage's expected scale

### Accepted trade-offs

- Spring Boot's WebSocket + STOMP story is more mature than Python WebSocket at scale, but `uvicorn` with FastAPI WebSocket handles the v1 load target (10+ concurrent users per tree per ADR-003). Re-evaluate if WebSocket connection count exceeds 10,000 concurrent.
- TypeScript remains on the client layer (web + mobile). Code sharing across layers is via OpenAPI contract: `lineage-api` generates an OpenAPI 3.1 spec; `packages/types` consumes it to produce typed clients for both `apps/web` and `apps/mobile`. Pydantic models → OpenAPI → TypeScript types is a clean one-way pipeline.
