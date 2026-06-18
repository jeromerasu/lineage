# Lineage

> Collaborative family health tree with AI-driven hereditary risk analysis.

Lineage helps families document, share, and reason about their medical history across generations. Real-time multiplayer canvas + AI risk analysis + genetic data integration.

**Status:** Design / Architecture phase. Code coming.

## What's here

- [`docs/PRD.md`](docs/PRD.md) — Product Requirements Document
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — System architecture + tech stack decisions
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — Milestones to public launch
- [`docs/decisions/`](docs/decisions/) — Architecture Decision Records (ADRs)

## Stack (planned)

| Surface | Tech |
|---|---|
| Web | Next.js + TypeScript |
| Mobile (iOS + Android) | React Native + Expo + TypeScript |
| `lineage-api` | Python + FastAPI |
| `lineage-analysis-worker` | Python + FastAPI + Anthropic SDK |
| `lineage-sync-server` | Python + Starlette/FastAPI WebSocket |
| `lineage-genetics-parser` | Python + FastAPI + biopython |
| `packages/types` | Shared TypeScript types (generated from OpenAPI) |
| Datastore | PostgreSQL (adjacency-list graph) |
| Real-time | WebSocket, last-write-wins with toast |
| LLM | Anthropic Claude (managed) with PII-stripping |

## Approach

- HIPAA design-for in v1, certify in v2 (PHI table separation + KMS field-level encryption + audit logging + BAA-eligible vendors only)
- Privacy-first architecture
- Multi-platform from day 1

## License

MIT
