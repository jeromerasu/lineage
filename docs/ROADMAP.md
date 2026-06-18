# Roadmap

Public milestones from PRD §10. The architecture phase (this repo's current state) precedes Week 1.

## Architecture Phase (Pre-Week 1)

- [x] PRD written and uploaded
- [x] Tech stack locked (ADR-001)
- [x] Monorepo structure decided (ADR-002)
- [x] Real-time conflict resolution approach decided (ADR-003)
- [x] HIPAA design strategy decided (ADR-004)
- [ ] Auth provider decision (Clerk vs. self-rolled JWT)
- [ ] Data model ERD (people + relationships adjacency-list schema)
- [ ] Design system + component library selection

## Milestones

| Milestone | Target | Status |
|---|---|---|
| PRD sign-off + legal review | Week 1–2 | In progress |
| Design mocks (canvas, card, analysis) | Week 4 | Not started |
| Eng spec + data model | Week 5 | Not started |
| Alpha (canvas, cards, details, roles) | Week 12 | Not started |
| Beta (AI analysis, real-time collab, genetic upload) | Week 18 | Not started |
| Security audit + privacy review | Week 20 | Not started |
| Public launch | Week 24 | Not started |

## Alpha Scope (Week 12)

The minimum lovable product: a family tree you can build and share.

- Family tree canvas (pannable, zoomable, React Flow)
- Person cards — Details tab (name, DOB, status, medical history, notes)
- Role-based access (Admin / Editor / Viewer)
- Invite system (email deep-link)
- Real-time presence cursors + last-write-wins conflict toast
- Auth (email/password + Google SSO)

No AI analysis, no genetic uploads in Alpha.

## Beta Scope (Week 18)

- AI Analysis tab per person card (`lineage-analysis-worker` live)
- Genetic data file upload + 23andMe / AncestryDNA OAuth
- `lineage-genetics-parser` live (SNP normalization + encryption)
- Apple SSO
- Mobile-responsive web

## v2 Planning (Post-Launch)

- React Native + Expo mobile apps (iOS + Android)
- HIPAA certification + BAA execution
- PDF export of analysis results (Open Question #1)
- Per-entry privacy controls (Open Question #3)
- Family tree merge (Open Question #6)
- Notification system for upstream/downstream risk updates (Open Question #2)
