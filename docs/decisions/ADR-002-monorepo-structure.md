# ADR-002: Monorepo Structure

**Date:** 2026-06-17
**Status:** Accepted

---

## Context

Lineage has four backend services (three Java, one Python), two frontend surfaces (Next.js web, React Native mobile), and a shared TypeScript types package. The question is whether to host these in one repository (monorepo) or separate repositories (polyrepo).

Key constraints:
- Single founding engineer — low overhead matters
- TypeScript types must be shared between web and mobile with zero drift
- PRD is in design phase — repo structure should encourage docs-first iteration before code sprawls
- The team will grow post-Alpha; the structure should support eventual parallel work by multiple engineers without repo proliferation

---

## Decision

**Single GitHub repository (`jeromerasu/lineage`) with the following top-level layout:**

```
lineage/
├── apps/
│   ├── web/          # Next.js
│   └── mobile/       # React Native + Expo
├── services/
│   ├── lineage-api/             # Java + Spring Boot (Maven)
│   ├── lineage-analysis-worker/ # Python + FastAPI (Poetry)
│   ├── lineage-sync-server/     # Java + Spring Boot (Maven)
│   └── lineage-genetics-parser/ # Java + Spring Boot (Maven)
└── packages/
    └── types/        # Shared TypeScript types (generated from OpenAPI)
```

**Build orchestration:**
- TypeScript apps and packages: Turborepo (`turbo.json` at root)
- Java services: each has its own `pom.xml`; Maven Wrapper per service; no root Maven aggregator
- Python service: Poetry (`pyproject.toml`) in `services/lineage-analysis-worker/`

Turborepo does NOT attempt to build Java or Python services. CI pipelines detect changes per directory and route to the appropriate build tool.

---

## Consequences

**Positive:**
- Single `git clone` to get everything — zero context switching between repos for a solo founder
- Shared TypeScript types in `packages/types` are consumed by both `apps/web` and `apps/mobile` with `workspace:*` protocol; Turborepo enforces build order
- Docs (`docs/`) sit alongside all code — PRD, ADRs, and ROADMAP are in the same PR flow as code changes
- Cross-service refactors (e.g., renaming a relationship type enum) are atomic commits
- GitHub Actions matrix can fan out per service directory, so CI isn't all-or-nothing

**Negative:**
- Turborepo only orchestrates TypeScript builds — Java and Python CI pipelines are manually maintained per service
- Repository grows large over time; shallow clones recommended for CI
- Newcomers need to understand that `turbo build` does NOT build the Java services — this must be documented in each service's README

**Accepted trade-offs:**
- The Java/Python build isolation is explicit and documented — it is less ergonomic than a full monorepo tool like Nx or Bazel, but avoids the complexity of wrapping Maven in a JavaScript build tool
- Polyrepo was considered (one repo per service) but rejected: type drift between `apps/web` and `apps/mobile` without a shared package is a known source of production bugs (cf. camelCase/snake_case onboarding incident in byb), and the PR-review overhead of cross-repo coordination is high for a solo-to-small team
