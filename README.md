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
| Web | Next.js + TypeScript + React |
| Mobile (iOS + Android) | React Native + Expo + TypeScript |
| `lineage-api` | Java + Spring Boot |
| `lineage-analysis-worker` | Python + FastAPI |
| `lineage-sync-server` | Java + Spring WebSocket |
| `lineage-genetics-parser` | Java + Spring Boot |
| `packages/types` | Shared TypeScript types (generated from OpenAPI) |
| Datastore | PostgreSQL (adjacency-list graph model) |
| Real-time | WebSocket, last-write-wins with toast on conflict |
| LLM | Anthropic Claude (managed) with PII-stripping anonymization layer |

## Approach

- HIPAA design-for in v1, certify in v2 (PHI table separation + KMS field-level encryption + audit logging + BAA-eligible vendors only)
- Privacy-first architecture
- Multi-platform from day 1

## License

MIT
