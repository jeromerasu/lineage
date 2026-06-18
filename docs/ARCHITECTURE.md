# Architecture

System-level decisions for Lineage. Reflects locked-in choices from initial design sessions.

## Stack

### Client layer

| Surface | Tech | Why |
|---|---|---|
| Web | Next.js + React + TypeScript | Mature canvas libraries (React Flow / xyflow), SSR for SEO, type-safe with API |
| Mobile (iOS + Android) | React Native + Expo + TypeScript | Single codebase ships both platforms, type-sharing with web, day-1 mobile presence |

### Backend services

All four backend services are Python + FastAPI. See [ADR-001](decisions/ADR-001-tech-stack.md) for the decision rationale.

| Service | Tech | Responsibilities |
|---|---|---|
| `lineage-api` | Python + FastAPI + SQLAlchemy 2.0 (async) + Alembic | REST API for trees, person cards, auth, invites, role permissions. Owns all persistence to Postgres. |
| `lineage-analysis-worker` | Python + FastAPI + Celery/Dramatiq + Anthropic SDK + networkx + pandas | Async job processor for AI hereditary-risk analysis. Graph traversal, ClinVar/OMIM cross-reference, LLM synthesis. Receives jobs via Redis Streams. |
| `lineage-sync-server` | Python + Starlette/FastAPI WebSocket + uvicorn | Real-time multiplayer canvas — presence cursors, card edits, last-write-wins conflict resolution. Python asyncio handles v1 WebSocket load. |
| `lineage-genetics-parser` | Python + FastAPI + biopython + cyvcf2 | Parses raw genome uploads (.txt / .zip / .vcf, 23andMe / AncestryDNA formats). Normalizes SNP data, encrypts at rest. biopython + cyvcf2 are the dominant ecosystem here. |

**Python tooling across all services:** `uv` (package management), `pyproject.toml` (project config), Pydantic v2 (DTOs + validation), Ruff (lint + format), mypy --strict (type checking), pytest + pytest-asyncio (tests). Python 3.12+.

Backend split intentionally — different scaling characteristics (LLM analysis is bursty, WebSocket is long-lived connections, REST is short request/response).

### Datastore

- PostgreSQL — single database for v1
- Graph stored as adjacency-list: `people` table (nodes) + `relationships` table (edges with type metadata)
- Recursive CTEs handle 3-5 generation traversals
- Migrate to Neo4j only if profiling shows hereditary-pattern queries are the bottleneck at scale

### LLM

- Managed via Anthropic Claude (enterprise tier for BAA-eligibility)
- PII-stripping anonymization layer in `lineage-analysis-worker` before any LLM call
- Migration path to self-hosted private LLM defined for v2 HIPAA certification

### Real-time conflict resolution

- Last-write-wins with toast notification to losing edit
- See [ADR-003](decisions/ADR-003-realtime-conflict-resolution.md) for trade-off analysis vs CRDT

### Monorepo structure

Single GitHub repo. Turborepo for TypeScript apps + packages. Java + Python services live alongside as siblings (their own Maven / Poetry build configs). See [ADR-002](decisions/ADR-002-monorepo-structure.md).

### Privacy / HIPAA

- Design for HIPAA in v1, certify in v2
- All health data: AES-256 at rest, TLS 1.3 in transit
- PHI table separation from non-PHI
- KMS-managed field-level encryption for sensitive fields
- Audit log table records every read/write of PHI
- Vendor selection limited to BAA-eligible only (Anthropic enterprise, Liveblocks enterprise, AWS RDS w/ BAA, Auth0/Clerk enterprise)
- See [ADR-004](decisions/ADR-004-hipaa-design.md)

### Auth

TBD — choosing between Clerk (best DX, BAA-eligible) and self-rolled JWT (full control, KataFit pattern). See open questions.
