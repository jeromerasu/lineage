# ADR-004: HIPAA Design Strategy

**Date:** 2026-06-17
**Status:** Accepted

---

## Context

Lineage stores Protected Health Information (PHI): medical history, genetic data, biological sex, date of birth, and family relationship graphs that — in combination — can uniquely identify individuals and their health status.

HIPAA (Health Insurance Portability and Accountability Act) imposes technical, administrative, and physical safeguards on covered entities and their business associates. The PRD explicitly raises this as an open question (§9 Q5) and a risk (§8):

> *"Do we pursue HIPAA compliance in v1, or design for it and defer certification to v2?"*
> *"HIPAA compliance requirements triggered by clinical use — Likelihood: Medium, Impact: High"*

Full HIPAA certification in v1 would require:
- A signed Business Associate Agreement (BAA) with every vendor that touches PHI
- A formal Risk Analysis and Risk Management Plan
- Workforce training documentation
- An appointed Security Officer
- An incident response plan
- Penetration testing and a third-party compliance audit

This is 3-6 months of compliance work on top of engineering, and is not appropriate for a pre-Alpha product.

---

## Decision

**Design for HIPAA in v1. Certify in v2.**

This means: every architectural and code-level decision in v1 will be made as if HIPAA applies, so that certification in v2 is an audit + documentation exercise rather than a code rewrite.

### Technical controls implemented in v1

**Encryption:**
- All PHI at rest: AES-256 (AWS RDS encryption enabled; KMS-managed key per environment)
- All PHI in transit: TLS 1.3 minimum (enforced at load balancer; HSTS headers set)
- Field-level encryption for the highest-sensitivity fields (medical history list, raw genetic file references): application-layer AES-256-GCM with KMS-managed DEKs (Data Encryption Keys), rotated annually

**PHI table separation:**
- Non-PHI tables: `users`, `family_trees`, `tree_memberships`, `invites`
- PHI tables: `people`, `medical_history_entries`, `genetic_data_refs`
- PHI tables live in the same Postgres instance but are tagged in schema comments; future migration to a separate encrypted RDS instance (or Postgres tablespace with column-level encryption) is additive

**Audit log:**
Every read and write of a PHI table row is logged to an append-only `phi_audit_log` table:

```sql
CREATE TABLE phi_audit_log (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    actor_id    UUID NOT NULL,          -- user performing the action
    action      TEXT NOT NULL,          -- READ | CREATE | UPDATE | DELETE
    table_name  TEXT NOT NULL,
    row_id      UUID NOT NULL,
    changed_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    ip_address  INET,
    user_agent  TEXT
);
```

This table is write-only from the application layer (no UPDATE or DELETE permissions granted to the app DB user on this table).

**Access control:**
- Role-based (Admin / Editor / Viewer) enforced at API layer via Spring Security `@PreAuthorize`
- No PHI is returned in API responses for Viewer-role callers beyond what the PRD explicitly permits
- Genetic data badge (presence indicator) is visible only to Admin and Editor — the raw file reference is never exposed to Viewer

**Anonymization before LLM calls:**
- `lineage-analysis-worker` strips all direct identifiers (name, DOB, address, genetic file IDs) before constructing the LLM prompt
- The prompt contains only: anonymized person IDs, relationship graph structure, medical condition codes (ICD-10), and age ranges (not exact DOBs)
- LLM provider: Anthropic Claude (enterprise tier with BAA)

### Vendor BAA eligibility (v1 vendor list)

| Vendor | Use | BAA status |
|---|---|---|
| AWS RDS (PostgreSQL) | Primary datastore | BAA available (AWS HIPAA Eligible Service) |
| AWS S3 | Genetic file storage | BAA available |
| AWS KMS | Key management | BAA available |
| Anthropic Claude (enterprise) | LLM synthesis | BAA available (enterprise tier) |
| Clerk / Auth0 (enterprise) | Auth | BAA available (enterprise tier) — final vendor TBD per ADR-001 open question |
| Liveblocks (enterprise) | Real-time presence (optional) | BAA available (enterprise tier) — only if chosen over custom sync server |

All vendors used in production must be on the BAA-eligible list above. No vendor that processes PHI may be added without confirming BAA availability.

### What is explicitly deferred to v2

- Executing BAAs with all vendors
- Formal HIPAA Risk Analysis document
- Security Officer appointment
- Workforce HIPAA training documentation
- Third-party penetration test
- Incident response plan (HITECH breach notification procedure)
- HIPAA-specific marketing language review

---

## Consequences

**Positive:**
- v1 ships faster without compliance overhead while maintaining a clean certification path
- The audit log, PHI table separation, KMS encryption, and anonymization layer are architecturally correct and would survive a HIPAA audit — they're not workarounds
- No v2 code rewrites required to achieve certification — only BAA execution + documentation
- Investors and early customers can be told truthfully: "We are HIPAA-ready and on a v2 certification track"

**Negative:**
- v1 cannot be used in any context where HIPAA applies (clinical settings, covered entity employees acting in their professional capacity)
- Marketing must explicitly avoid clinical language ("for informational purposes only" disclaimers must be present on all analysis output — already specified in PRD §5.4)
- Any breach before v2 certification would carry full legal exposure — the v1 encryption and access controls are the primary risk mitigation

**Accepted trade-offs:**
- The risk of a breach in Alpha/Beta is low (limited user base, no marketing to clinical users) and the encryption + access control posture is materially stronger than most consumer health apps
- The "design for HIPAA" constraint slightly increases implementation cost per feature (e.g., every new PHI-touching endpoint must log to `phi_audit_log`) but this discipline produces better code regardless of compliance status
