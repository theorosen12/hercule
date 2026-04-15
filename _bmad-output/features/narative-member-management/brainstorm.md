# Brainstorm — Narative Member Management

## Feature Concept

From the NarativePage, members can navigate to a dedicated members page that currently shows a static list. This feature introduces a new **Member Management page** — separate from the read-only Members page — accessible via a button visible only to members. It handles three capabilities: adding friends, validating pending join requests, and self-removing from the narative.

## Pendo Signals

- None — Pendo not connected during this session. No usage data available.

## Ideation Summary

### Ideas Explored

**HMW Questions explored:**
- How might we make inviting a member as fast as sharing a link?
- How might we avoid unwanted members being added by any member?
- How might we keep join request notifications non-fatiguing?
- How might we make member removal non-conflictual?
- How might we let a user join a narative without knowing anyone inside?
- How might we prevent accidental exposure of a private narative via invitations?

**Alternative approaches considered:**

| Option | Description | Verdict |
|--------|-------------|---------|
| A — Invite Link | Time-limited shareable link, anyone with it can join or request | Too viral, low control |
| B — Search & Add | Search by username/firstname/lastname, add immediately | ✅ Selected — but scoped to friends only |
| C — Request & Approve | Users request entry, members approve | Partially adopted for join request validation |
| D — Hybrid | Search & Add + request validation coexist | ✅ Adopted — friends-only add + request validation |

### Selected Approach

**Friends-only Add + Validate Join Requests + Self-Remove — on a dedicated Member Management page**

#### Page Architecture

| Page | Access | Capabilities |
|---|---|---|
| **Members page** (existing, read-only) | All visitors | View member list |
| **Member Management page** (new) | Members only — via button on Members page | Add friend, validate requests, self-remove |

- The button to access the Management page is **invisible to non-members**
- No badge on the Members page — pending request count is surfaced inside the Management page only

#### Capability Details

- **Add a member:** Any member can search within their **friend list** (username or first/last name) and add a friend immediately — no confirmation modal, no soft invite. The added person receives a push notification: "[Name] added you to [Narative name]."
- **Validate join requests:** Non-members send join requests from NarativePage (existing flow). Requests were previously routed to the author (deprecated role) — now broadcast to all members and surfaced in the Management page. Any member can approve or reject. **Rejection is final for all members** — first actor wins in both directions.
- **Self-remove:** A member can leave the narative from the Management page. One-tap + confirmation modal (destructive action). No forced removal of other members.

#### Key Constraints
- **Friends-only add** — cannot add arbitrary Nara users, only existing friends
- **No author/creator role** — all members are equal, all have identical access to the Management page
- **No admin tier** — no permission levels in v1

### Risks & Anti-patterns Identified

- **Non-consensual add:** Mitigated by friends-only restriction. Push notification must clearly name the adder and narative, with self-remove as the safety valve.
- **Join request routing migration:** Backend must decouple join request routing from the deprecated author field — now broadcast to all members.
- **Request flood:** Low risk (naratives are private), but requests sorted by recency and no batch actions in v1.
- **Silent departure:** Nice-to-have — notify remaining members when someone leaves. Deferred to post-v1.
- **Scale UX:** Scoped to small naratives in v1. Bulk actions deferred.

### Adjacent Opportunities

- Push notifications for new join requests to all members (required)
- "Recent members" activity indicator (future backlog)
- Block/report from Members page (future backlog)
- Notification when a member leaves (future backlog)
- Bulk approve/reject join requests (future backlog)

## Research Findings

None — skipped.

## Decisions for the PRD

1. **Entry point:** Members page → "Manage members" button (members-only, invisible to non-members)
2. **New page:** Member Management page — dedicated screen for all edit actions
3. **Add member mechanic:** Search within friend list by username or first/last name → one-tap add → immediate membership → push notification to added user: "[Name] added you to [Narative]"
4. **No confirmation modal on add** — friends-only scoping provides sufficient trust
5. **Join requests:** Existing backend flow (non-member requests from NarativePage). Decouple routing from author — now visible to all members in Management page
6. **Request validation:** Any member can approve or reject. Rejection is final for all members (first actor wins)
7. **Self-remove:** Management page only. Confirmation modal required (destructive). One-tap leave.
8. **No forced removal** of other members — out of scope
9. **No author/creator role** — all members are equal
10. **Badge / pending count:** Surfaced inside the Management page, not on the Members page or its entry button
11. **Push notification on new join request:** Sent to all members of the narative
12. **Search results:** Show username only (unique identifier) — no confirm modal
13. **Request card:** Shows requester avatar + username

## Out of Scope (decided here)

- Forced removal of a member by another member
- Admin/creator role differentiation — all members are equal
- Invite links (Option A)
- Block/report functionality
- Bulk approve/reject actions
- Notification when a member leaves (nice-to-have, deferred)
- Confirmation modal when adding a friend (skipped — friends-only is sufficient trust boundary)
