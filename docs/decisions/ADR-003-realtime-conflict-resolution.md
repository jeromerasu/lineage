# ADR-003: Real-Time Conflict Resolution

**Date:** 2026-06-17
**Status:** Accepted

---

## Context

Lineage's family tree canvas is a real-time collaborative space: multiple users can be editing simultaneously. When two users edit the same field on the same person card at the same moment, the system must decide what to do.

The two standard approaches are:

**Option A — Last-Write-Wins (LWW)**
The most recent write (by server clock) survives. The user whose write was discarded sees a toast notification ("Your change was overwritten by [Name]"). No data merging, no operational transforms.

**Option B — CRDT (Conflict-free Replicated Data Types)**
Each character insertion/deletion is a separate operation. Edits from multiple users are merged without conflicts by mathematical construction. Libraries: Yjs, Automerge. Significantly more complex to implement, especially for structured data like person card fields (enums, dates, lists).

The PRD specifies both options as candidates (§5.7, §7):
> *"Conflicting simultaneous edits to the same field are resolved with last-write-wins and surfaced to both editors via a brief toast notification."*
> *"WebSocket or CRDT-based (e.g., Yjs) for conflict-free collaborative edits on the canvas."*

---

## Decision

**Last-write-wins (LWW) with toast notification for v1.**

The `lineage-sync-server` (Java + Spring WebSocket + STOMP) will:
1. Broadcast field-level edit events to all clients on the canvas
2. Apply a server-side timestamp to each incoming edit
3. On concurrent writes to the same field, the later timestamp wins
4. The losing client receives a `CONFLICT` WebSocket message identifying which field was overwritten and by whom, and renders a dismissible toast: *"[Name]'s edit to [field] overwrote yours."*

---

## Consequences

**Positive:**
- Implementation is straightforward — no CRDT library integration, no operational transform math
- Spring WebSocket + STOMP handles the broadcast pattern natively
- The user experience is acceptable for the primary use case: family members editing different person cards simultaneously (the conflicting-same-field scenario is rare in a family health tree)
- Shipping faster means the real-world conflict rate can be measured before committing to CRDT complexity

**Negative:**
- Data loss is possible: if two users simultaneously edit the same medical history list item, one user's change is silently dropped (they only see a toast after the fact)
- The toast is surfaced but the lost content is not recoverable in v1 (no edit history)
- For bulk edits (e.g., a Family Organizer adding 10 conditions in rapid succession while another editor is active), multiple toasts may fire

**Conditions for re-evaluation (v2):**
- If user reports surface simultaneous edit conflicts more than once per 100 active sessions, CRDT (Yjs) should be prototyped for the medical history list field specifically — the highest-contention field
- The canvas node position (drag-and-drop) is a natural CRDT candidate if performance degrades at 10+ concurrent users per tree (PRD Risk §8)

**CRDT trade-off not taken in v1:**
Yjs would eliminate data loss but requires:
- Restructuring all person card fields as Y.Map / Y.Text / Y.Array types
- Persisting the Yjs document alongside the relational data model
- A Yjs awareness protocol for presence cursors (replaces the bespoke STOMP cursor broadcast)
- Non-trivial server-side Yjs document management (snapshot + GC)

The complexity cost exceeds the benefit for v1 given the low expected collision rate in this specific domain (families typically take turns, not simultaneous same-field edits).
