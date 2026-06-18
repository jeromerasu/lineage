# ADR-001: Tech Stack

**Date:** 2026-06-17
**Status:** Accepted

---

## Context

Lineage is a greenfield project with a single founding engineer (Jerome Reyes). The stack must:

1. Ship a polished web and mobile product without separate teams
2. Handle real-time collaborative state, async LLM jobs, raw genome file parsing, and REST API concerns — which have meaningfully different scaling and runtime characteristics
3. Support a future HIPAA certification path (BAA-eligible vendors, audit trails, encryption primitives)
4. Double as a portfolio piece and intentional learning vehicle for the founder

The founding engineer has deep production experience with Java + Spring Boot (byb / KataFit). Python is an intentional addition for career diversification (data science ecosystem, LLM tooling, ML adjacency). TypeScript is non-negotiable for any FE layer — web and mobile must share types.

---

## Decision

### Java (Spring Boot) for three backend services

- `lineage-api` — REST API, auth, tree/card/role persistence
- `lineage-sync-server` — WebSocket + STOMP real-time server
- `lineage-genetics-parser` — genome file parsing, SNP normalization

**Why Java over Node/Python for these:** Java is the founder's primary production language. Spring Boot's ecosystem covers WebSocket (Spring WebSocket + STOMP), encryption primitives (BouncyCastle), and Flyway migrations out of the box. Strong typing catches classification bugs (e.g., relation enum values) at compile time. JVM memory management is predictable under long-lived WebSocket connections.

### Python (FastAPI) for one service

- `lineage-analysis-worker` — hereditary risk graph traversal + ClinVar/OMIM cross-reference + LLM synthesis

**Why Python here:** The scientific Python ecosystem (`networkx` for graph analysis, `biopython` for SNP parsing, established OMIM/ClinVar client libraries) is materially ahead of Java alternatives for biomedical data work. FastAPI + Redis Streams is the standard pattern for async LLM job queues. This service is bursty (triggered by tree updates, not by live user sessions), so it's safe to run on a separate pool. The founder has identified this as an intentional Python learning track to build practical experience with LLM orchestration and data science tooling.

### TypeScript + React ecosystem for all client surfaces

- Web: Next.js + React
- Mobile: React Native + Expo

**Why Next.js:** React Flow (xyflow) for the family tree canvas is the strongest maintained option; it's React-native. Next.js adds SSR for SEO and a clean API route layer if needed for thin BFF calls. TypeScript throughout.

**Why React Native + Expo over Flutter/Swift/Kotlin:** Shared TypeScript types between web and mobile via `packages/types`. Founder already knows React. Expo simplifies OTA updates and provisioning. Day-1 iOS + Android from a single codebase — critical for a consumer health app where iOS and Android both matter.

---

## Consequences

**Positive:**
- Founder ships using strengths (Java, TypeScript) while deliberately expanding (Python)
- Service boundaries match natural deployment unit boundaries (bursty async ≠ long-lived WebSocket ≠ REST)
- Type safety end-to-end: Java generics + TypeScript from API contract to UI
- Python service is independently deployable and scalable

**Negative:**
- Three languages in the repo (Java, Python, TypeScript) means three build systems, three linting setups, three CI pipelines
- Code sharing across languages is limited to OpenAPI contract (generates TS types + Java model stubs)
- Python expertise on the team is limited to what the founder builds during development — bus factor of one on the analysis worker

**Accepted trade-offs:**
- The multi-language complexity is mitigated by clear service ownership — each service has one language, one build tool, one deployer
- Monorepo gives visibility across all services without requiring polyglot expertise per developer (see ADR-002)
