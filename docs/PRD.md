# Product Requirements Document

## Family Health Tree — *Codename: Lineage*

| Field | Detail |
|---|---|
| Author | Jerome Reyes |
| Status | Draft |
| Version | 0.1 |
| Last Updated | April 8, 2026 |
| Stage | 0 → 1 |

---

## 1. Problem Statement

Medical history is scattered, siloed, and lost between generations. People arrive at doctor appointments without knowing a grandparent's cancer history or a parent's hereditary condition — information that could meaningfully change their care plan. No tool today combines family tree visualization, collaborative health record logging, genetic data integration, and AI-driven risk analysis in one place. *Lineage* changes that.

---

## 2. Goals & Success Metrics

### Goals

- Make it effortless for families to collaboratively document and share medical history across generations.
- Surface actionable, AI-driven health risk insights so users can have more targeted conversations with their doctors.
- Build a privacy-first platform families trust with sensitive health data.

### Success Metrics (6-Month Post-Launch)

| Metric | Target |
|---|---|
| Registered family trees | 10,000 |
| Avg. members per tree | ≥ 5 |
| Medical history entries logged per tree | ≥ 12 |
| Genetic data uploads / links per user | ≥ 15% of active users |
| D30 Retention | ≥ 35% |
| Analysis tab views per active user / month | ≥ 3 |

---

## 3. Target Users

| Persona | Description |
|---|---|
| **The Health-Conscious Adult** | Wants a proactive view of their risk factors before symptoms appear. |
| **The Family Organizer** | Manages health info for aging parents and young children; needs collaborative access control. |
| **The Medical Professional** | Recommends the tool to patients as a between-visit prep and screening aid. |

---

## 4. Scope

### In Scope (v1)

- Family tree creation, management, and visual diagram view
- Person cards with Details and Analysis tabs
- Medical history logging per person
- AI-powered health risk analysis (Analysis tab)
- Genetic data upload and third-party linking (23andMe, AncestryDNA)
- Role-based access control (Admin / Editor / Viewer)
- Real-time collaborative editing with presence cursors
- Invite system (email-based, role-assigned)
- Multiple family trees per user account

### Out of Scope (v1)

- Direct EHR / medical record integrations (Epic, MyChart)
- Video or messaging between family members
- Mobile native app (web-first; responsive mobile web)
- Monetization / subscription tiers
- HIPAA Business Associate Agreements (BAA) for clinical use

---

## 5. Features & Requirements

### 5.1 Authentication & Account

- Sign up and log in via email/password, Google SSO, or Apple SSO.
- Each account holds one user identity that can belong to multiple family trees in different roles.
- Password reset and email verification required at signup.

### 5.2 Home Page — Tree Dashboard

- **Default state (new user):** A single tile labeled "Grow your Family Tree" centered on the page.
- **On click:** A modal prompts for a family tree name. On submit, a new tree is created and the user is navigated to the tree canvas page.
- **Returning / multi-tree state:** All trees the user owns or has been invited to are displayed as individual tiles in a grid. Each tile shows: tree name, member count, user's role in that tree, and last updated timestamp.
- Tiles are sorted by most recently edited.

### 5.3 Family Tree Canvas

The core experience. A pannable, zoomable canvas displaying the family tree as an interactive node diagram.

**Initial state:** One card representing the logged-in user appears in the center of the canvas.

**Card layout:**

- Displays: name, profile photo (optional), year of birth, status (Alive / Deceased / Unknown)
- A subtle relationship label appears beneath each card (e.g., "You," "Mother," "Maternal Grandfather")

**Hover state:** A `+` button appears on the card. Clicking it opens a dropdown to add a relationship:

- Parents (Mother, Father)
- Grandparents (Maternal / Paternal)
- Siblings (Full / Half / Step)
- Children
- Spouse / Partner

On selection, a new blank card is generated and connected to the source card with a styled line. Relationship lines use standard family tree conventions (horizontal for couples, vertical for parent-child).

**Canvas controls:**

- Pan: click and drag on empty canvas
- Zoom: scroll wheel or pinch gesture (mobile)
- Minimap: bottom-right corner for orientation on large trees
- Center on me: button that re-focuses the canvas on the current user's card

### 5.4 Person Card — Expanded View

Clicking a card opens an expanded side panel (not a new page) with two tabs: **Details** and **Analysis**.

#### Details Tab

| Field | Type | Notes |
|---|---|---|
| Name | Text | First + Last |
| Relation | Dropdown | Full/Half/Step Sibling, Parent, Grandparent, Child, Spouse, Self |
| Date of Birth | Date picker | |
| Status | Dropdown | Alive / Deceased / Unknown |
| Date of Death | Date picker | Shown only if Status = Deceased |
| Biological Sex | Dropdown | Used for sex-linked hereditary risk modeling |
| Medical History | List | One condition per line; free text with type-ahead suggestions (ICD-10 backed) |
| Notes | Long text | Free-form field for additional context |
| **Upload Genetic Data** | Button | See §5.5 |

Medical history entries support tagging by category (Cancer, Cardiovascular, Neurological, Autoimmune, Other) for use in the analysis engine.

#### Analysis Tab

Populated automatically by the AI analysis engine (see §5.6). Displays:

- **Risk factors identified** — conditions this person is at elevated risk for, with a plain-language explanation of why (hereditary pattern, genetic marker, family prevalence).
- **Upstream / downstream impact** — which relatives in the tree may share this risk, shown as a visual summary (e.g., "2 of your children may be at elevated risk for Type 2 Diabetes").
- **Recommended screenings** — actionable suggestions (e.g., "Consider a colonoscopy at age 40 given your family history").
- **Data confidence indicator** — a badge showing how complete the tree's data is; prompts user to fill gaps that would improve analysis accuracy.
- **Disclaimer:** *"This analysis is for informational purposes only and does not constitute medical advice. Consult your physician before making any healthcare decisions."*

Analysis re-runs automatically whenever new medical history or genetic data is added.

### 5.5 Genetic Data Integration

Each person card has an **"Upload Genetic Data"** button in the Details tab.

**Options presented:**

1. **File upload** — accepts raw genome files (`.txt`, `.zip`, `.vcf`) from any consumer genetic testing kit.
2. **Link 23andMe** — OAuth connection to pull available health and ancestry data via the 23andMe API.
3. **Link AncestryDNA** — OAuth connection to pull available data via the AncestryDNA API.

**Data handling:**

- Uploaded files are encrypted at rest and in transit.
- Only the authenticated tree Admin can upload or link genetic data for another member; individuals can always upload their own.
- A badge on the card indicates genetic data is present (visible only to Admins and Editors).

### 5.6 AI Analysis Engine

The analysis engine combines three data sources: manually logged medical history, genetic data, and hereditary patterns derived from the tree structure.

**How it works:**

- On each tree update, the engine traverses the family graph to identify hereditary patterns (e.g., condition appearing in 2+ generations, maternal vs. paternal lineage patterns, sex-linked conditions).
- It cross-references known hereditary risk databases (e.g., ClinVar, OMIM) alongside any uploaded genetic markers.
- A large language model (GPT-4 class or equivalent) synthesizes findings into plain-language explanations and recommendations for each person's Analysis tab.
- Risk signals propagate both upstream (parents, grandparents) and downstream (children, grandchildren) — e.g., if a grandchild logs early-onset diabetes, the engine flags risk for cousins and parents.

**Privacy constraints:**

- Analysis runs server-side; no personal health data is sent to third-party LLM APIs without anonymization.
- Users can opt out of AI analysis per tree.

### 5.7 Collaboration & Access Control

#### Invite System

An **"Invite"** button sits in the top-left of every family tree canvas page. Clicking it opens a modal:

- Email address input field
- Role selector: **Admin**, **Editor**, or **Viewer**
- Optional personal message
- "Send Invite" button

Invitees receive an email with a deep link. If they don't have an account, they're directed to sign up first, then land on the shared tree.

#### Role Permissions

| Permission | Admin | Editor | Viewer |
|---|---|---|---|
| View all cards and data | ✅ | ✅ | ✅ |
| Edit any person's card | ✅ | ✅ | ❌ |
| Add new person cards | ✅ | ✅ | ❌ |
| Upload / link genetic data | ✅ | ✅ | ❌ |
| Invite new users | ✅ | ❌ | ❌ |
| Remove users from tree | ✅ | ❌ | ❌ |
| Delete the family tree | ✅ (owner only) | ❌ | ❌ |
| Change another user's role | ✅ | ❌ | ❌ |

#### Real-Time Collaboration

When multiple users are on the canvas simultaneously:

- Each active user's cursor is visible to others in a distinct, assigned color with their name label attached.
- Card edits appear in real-time — typing in a field, adding a card, or moving a card is reflected instantly for all viewers.
- A presence bar in the top-right shows avatar icons for everyone currently on the canvas.
- Conflicting simultaneous edits to the same field are resolved with last-write-wins and surfaced to both editors via a brief toast notification.

Implementation: WebSocket-based (e.g., via Liveblocks, PartyKit, or a custom Socket.io layer).

---

## 6. User Flows

### New User Creating First Tree

```
Sign Up → Home (single "Grow your tree" tile) → Click tile → Enter tree name
→ Canvas with "You" card → Hover card → Add Mom / Dad → Cards generated
→ Click a card → Add Details → Analysis tab populates
```

### Invited User Joining a Tree

```
Receive email invite → Click link → Sign up or log in → Land on shared canvas
→ Role-appropriate permissions applied automatically
```

### AI Analysis Loop

```
Log medical history entry → Engine re-runs → Analysis tab updates
→ Upstream/downstream relatives' Analysis tabs update → User notified
```

---

## 7. Technical Considerations

| Area | Consideration |
|---|---|
| **Data privacy** | All health data encrypted at rest (AES-256) and in transit (TLS 1.3). Consider HIPAA-readiness as a v2 priority. |
| **Real-time sync** | WebSocket or CRDT-based (e.g., Yjs) for conflict-free collaborative edits on the canvas. |
| **AI / analysis** | Health data must be anonymized before sending to any third-party LLM API. On-premise or private LLM deployment is preferred for v1 given sensitivity. |
| **Genetic APIs** | 23andMe and AncestryDNA APIs have access restrictions — early partnership outreach recommended. File upload is the reliable fallback. |
| **Graph data model** | Family tree is a directed graph (nodes = people, edges = relationships with type metadata). Graph DB (Neo4j) or adjacency-list in Postgres both viable. |
| **Platform** | Web-first (React). Responsive for mobile browsers. Native mobile apps deferred to v2. |

---

## 8. Dependencies & Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| 23andMe / AncestryDNA API access denied or deprecated | Medium | High | Prioritize file upload; treat API links as enhancement |
| Health data breach / privacy violation | Low | Critical | Encryption, access controls, audit logs, legal review before launch |
| AI analysis producing inaccurate or alarmist health claims | Medium | High | Strong disclaimers; human review of output templates; tiered confidence scores |
| HIPAA compliance requirements triggered by clinical use | Medium | High | Consult healthcare attorney pre-launch; avoid clinical language in marketing |
| Real-time sync complexity at scale | Medium | Medium | Use proven CRDT library (Yjs); load test at 10+ concurrent users per tree |

---

## 9. Open Questions

1. Should analysis results ever be exportable as a PDF for users to bring to doctor appointments?
2. Do we build a notification system (email / push) to alert users when a relative updates health data that affects their risk profile?
3. Should users be able to mark specific health entries as private — visible only to themselves, not other tree members?
4. What is the minimum data set required before the AI engine returns useful analysis vs. a "not enough data" state?
5. Do we pursue HIPAA compliance in v1, or design for it and defer certification to v2?
6. Can users merge two separate family trees (e.g., when two families are linked by marriage)?

---

## 10. Timeline (Proposed)

| Milestone | Target |
|---|---|
| PRD sign-off + legal review | Week 1–2 |
| Design mocks (canvas, card, analysis) | Week 4 |
| Eng spec + data model | Week 5 |
| Alpha (canvas, cards, details, roles) | Week 12 |
| Beta (AI analysis, real-time collab, genetic upload) | Week 18 |
| Security audit + privacy review | Week 20 |
| Public launch | Week 24 |
