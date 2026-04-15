---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
inputDocuments:
  - '_bmad-output/project-context.md'
  - '_bmad-output/planning-artifacts/product-brief-DEVZONE-2026-03-14.md'
  - '_bmad-output/features/narative-member-management/context.md'
  - '_bmad-output/features/narative-member-management/brainstorm.md'
  - '_bmad-output/features/narative-member-management/data-brief.md'
workflowType: 'prd'
feature: 'narative-member-management'
linearIssue: 'NAR-461'
classification:
  projectType: 'Mobile App (React Native/Expo) — feature addition'
  domain: 'Consumer Social / Memory Sharing'
  complexity: 'Medium'
  projectContext: 'brownfield'
---

# Product Requirements Document — Narative Member Management

**Author:** Matthieu
**Date:** 2026-04-13
**Linear:** NAR-461
**Branch:** `matthieugouverith/nar-461-narative-member-management-add-members-validate-join`

---

## Executive Summary

The Narative Member Management feature transforms the static Members page into a live membership hub by introducing a dedicated **Member Management page** — accessible to all narative members equally, invisible to non-members. It addresses a structural gap in Nara: naratives are collaborative memory spaces whose membership is frozen at creation, with no in-app mechanism to expand or manage the group. The former "author" routing model (now deprecated) left join requests orphaned. This feature restores collective ownership by enabling any member to add friends, validate pending join requests, and self-remove — with no admin tier, no gatekeeper, and no external coordination required.

**Target users:** All members of an existing narative on Nara (React Native/Expo, iOS/Android).

**Problem solved:** Once a narative is created, its membership cannot be managed from within the app. Adding someone requires out-of-band coordination. Pending join requests from non-members have no visible destination since the author role was removed. Members who want to leave have no self-serve option.

### What Makes This Special

Nara's trust model is already built — the friend graph. This feature leverages it directly: members can only add existing Nara friends, eliminating consent flows and confirmation modals. The friend relationship *is* the approval. Combined with first-actor-wins request validation (any member can approve or reject, rejection is final for all) and push notifications routed to all members rather than a single gatekeeper, this is a fully decentralised membership model that matches how Nara's social graph actually works.

**Differentiator:** No admin role. No permission tiers. No invite links. Friends adding friends, any member handling the door — because in Nara, the narative belongs to everyone in it equally.

---

## Success Criteria

### User Success

- A member can add a Nara friend to a narative in **≤ 2 taps** from the Member Management page
- A member can view all pending join requests and act on one in **≤ 3 taps**
- A member can self-remove from a narative in **≤ 2 taps** with a confirmation modal
- The added user receives a push notification naming the adder and the narative within **< 5 seconds** of the action
- All members receive a push notification when a new join request arrives
- Non-members cannot access or discover the Member Management page
- All members have identical access to the Management page — no permission errors

### Business Success

- **Membership growth:** Naratives using the feature show a measurable increase in average member count (baseline: static since creation)
- **Request resolution rate:** ≥ 80% of pending join requests resolved within 7 days of launch
- **Feature adoption:** ≥ 50% of active naratives have at least one member who used the Management page within 30 days of release
- **Zero access violations:** No reports of non-members accessing the Management page

### Technical Success

- `addMember(narativeId, userId)` succeeds only for users in the caller's friend list
- `joinRequests(narativeId)` returns pending requests to any narative member (not gated on author field)
- `approveJoinRequest` / `rejectJoinRequest` resolve requests for all members simultaneously
- `leaveNarative(narativeId)` removes the caller from the member list
- Backend join request routing fully decoupled from deprecated author field

### Measurable Outcomes

| Outcome | Metric | Target |
|---|---|---|
| Frictionless add | Taps to add a friend | ≤ 2 |
| Request resolution | % requests resolved within 7 days | ≥ 80% |
| Feature adoption | % active naratives using Management page | ≥ 50% in 30 days |
| Notification delivery | Push latency on member add | < 5 seconds |
| Access control | Unauthorised access incidents | 0 |

---

## Product Scope

### MVP — Phase 1

**Approach:** Experience MVP — minimum that makes a member feel genuinely in control of narative membership without leaving the app. All 5 user journeys must work end-to-end.

**Resource requirements:** 1 mobile developer + 1 backend developer. No new infrastructure. Evanescent design system covers UI primitives.

| Capability | Justification |
|---|---|
| Member Management page (new screen) | Entry point for all management actions |
| Members-only access guard (resolver level) | Non-members must not see or access management |
| Friend search by username / first+last name | Core add flow — friends-only scope |
| `addMember` mutation + real-time list update | Immediate add, no confirmation modal |
| Duplicate membership guard (inline error) | Prevents confusion |
| `joinRequests(narativeId)` query | Surface pending requests |
| Pending request card (avatar + username) | Informed approve/reject decision |
| `approveJoinRequest` mutation | First-actor-wins |
| `rejectJoinRequest` mutation (deletes request) | Silent rejection, no REJECTED state in v1 |
| `leaveNarative` mutation + confirmation modal | Self-remove with destructive action guard |
| Push: member added → added user | Warm notification naming adder + narative |
| Push: new join request → all members | Action trigger with badge |
| Push: request approved → requester | Closing the loop |
| Backend: decouple join requests from author field | Critical migration — feature cannot function otherwise |
| Badge count on Management page button | Members see pending count at entry |

### Growth — Phase 2

- Notification to remaining members when someone leaves
- "Recent members" activity indicator on Members page
- Mutual narative count in search results for disambiguation
- Batch approve/reject join requests

### Vision — Phase 3

- Block/report a user from the Members page
- Role differentiation if product evolves beyond equal-access model
- Member activity timeline within a narative

### Risk Mitigation

- **Backend migration (highest risk):** Decoupling join request routing from the deprecated author field is the most complex backend task. Scoped explicitly as MVP-blocking.
- **Race condition (concurrent approval):** Idempotent `addMember` at DB level — second call returns a no-op, not an error.
- **Scope creep:** All Phase 2/3 items are explicitly named and deferred. Push notification for "request approved" can slip to Phase 2 without breaking core functionality.

---

## User Journeys

### Journey 1 — Léa adds a friend (Primary — Happy Path)

*Léa just got back from a weekend with friends. She created the narative before the trip but forgot to add Sophie, who joined for the last two days.*

**Opening Scene:** Léa opens the narative from her feed. She taps "Members" — Sophie isn't listed. A "Manage members" button is visible (she's a member).

**Rising Action:** She taps it. The Member Management page opens: member list, empty "Pending Requests" section, and an "Add a friend" action. She taps "Add". A bottom sheet opens with a search field. She types "Sophie" — her friend list returns two results: `@sophiemlt` and `@sophied`. She recognises the username immediately.

**Climax:** One tap on `@sophiemlt`. Sophie is added instantly — no modal, no waiting. The member list updates in real time.

**Resolution:** Sophie receives a push notification: *"Léa added you to Weekend Normandie."* She opens the app and finds herself inside a rich narative. The memory is complete.

**Capabilities:** Friend-scoped search, `addMember`, real-time member list update, push on add.

---

### Journey 2 — Duplicate add attempt (Edge Case)

*Léa searches for her partner Théo — already a member.*

**Scene:** Search returns `@theo_g`. She taps him. An inline message appears in the search sheet: *"Théo is already a member of this narative."* No toast, no crash. She dismisses and moves on.

**Capabilities:** Duplicate membership guard, inline error in search sheet.

---

### Journey 3 — Karim requests to join, Antoine approves (Join Request Flow)

*Karim sees a narative on Antoine's profile — a family reunion he wasn't at.*

**Opening Scene:** Karim visits the NarativePage as a non-member. He taps "Request to join" (existing flow). His request is sent.

**Member side:** Antoine receives a push: *"Karim wants to join Réunion Famille Juillet."* He sees the "Manage members" button with a badge: **1**. He taps it.

**Climax:** The Management page shows Karim's request card: avatar, `@karim_b`, **Approve** / **Reject**. Antoine taps **Approve**.

**Resolution:** Karim is immediately added. He receives: *"Your request to join Réunion Famille Juillet was approved."* The request disappears from the pending list for all members.

**Capabilities:** Join request badge, pending request list, `approveJoinRequest`, push to all members on new request, push to requester on approval.

---

### Journey 4 — Request rejection (Edge Case)

*The same narative receives a request from `@unknown_user42`. Antoine doesn't recognise them.*

**Scene:** Antoine taps **Reject** — no confirmation modal (not a destructive action on his end). The request is deleted and disappears for all members. The requester receives no notification (silent rejection). First actor wins.

**Capabilities:** `rejectJoinRequest`, final deletion for all members, silent rejection.

---

### Journey 5 — Camille self-removes (Self-Remove)

*Camille was added to a narative about a trip she couldn't attend. She wants out.*

**Scene:** Camille opens the Management page and taps "Leave this narative". A confirmation modal: *"Leave Voyage à Lisbonne? You won't be able to rejoin unless a member adds you again."* She taps **Leave**.

**Resolution:** Camille is removed. The Management page closes. She's back on the NarativePage as a non-member — the "Manage members" button is gone. Other members are not notified (v1).

**Capabilities:** `leaveNarative`, confirmation modal, member → non-member state transition.

---

### Journey Requirements Summary

| Journey | Capabilities Required |
|---|---|
| Add friend | Friend search, `addMember`, real-time update, push to added user |
| Duplicate add | Duplicate guard, inline error |
| Join request + approve | Request badge, pending list, `approveJoinRequest`, push to all members + requester |
| Reject request | `rejectJoinRequest`, final deletion, silent |
| Self-remove | `leaveNarative`, confirmation modal, state transition |

---

## Domain-Specific Requirements

### GDPR & Privacy

- User data accessed during search (username, avatar) is limited to fields the target user consented to share on their profile
- Friend graph data is never exposed beyond the requesting user's own friend list
- `leaveNarative` removes the user from the member list; narative content contributed by the user is not deleted (content belongs to the narative, not the member)
- Infrastructure: `eu-west-3` (Paris) — all data remains in-region

### Push Notification Consent

- Push permission was granted at app onboarding — this feature does not re-request it
- Notifications fire only if a valid Expo push token exists — silent failure otherwise
- No notification is sent on rejection (v1) — rejection is silent to the requester

---

## Mobile-Specific Constraints

### Platform

- React Native 0.81.5 + Expo ~54.0.31 — cross-platform (iOS + Android)
- Expo Router file-based routing — new screen under `src/app/(app)/`
- Portrait-only — no landscape handling needed
- No new native modules, no new device permissions

### Offline Behaviour

- Management mutations (`addMember`, `approveJoinRequest`, `rejectJoinRequest`, `leaveNarative`) require network — show standard offline error state
- Read-only member list may be Apollo-cached — acceptable to show stale data with a refresh indicator

### Push Notification Strategy

| Trigger | Recipient(s) | Copy |
|---|---|---|
| Member added | Added user | "[Name] added you to [Narative name]" |
| New join request | All narative members | "[Name] wants to join [Narative name]" |
| Join request approved | Requester | "Your request to join [Narative name] was approved" |

- Copy must be warm and personal — not system-toned
- Deep link: tapping any notification opens the relevant narative
- Silent failure on invalid/missing push token — no error surfaced to acting member

### Implementation Rules

- Styling: Emotion `styled()` exclusively — no `StyleSheet.create()`
- i18n: all user-facing strings via `react-intl` — run `translations:extract` + `translations:compile` after implementation
- Apollo: run `api:generate-types` after any backend schema changes
- Design system: use Evanescent (`@nara/evanescent/`) components before creating custom ones
- No `console.log` — ESLint blocks it; use `console.warn` / `console.error` or NestJS `Logger`

---

## Security Considerations

### Authentication & Authorisation

- All management mutations must be authenticated — unauthenticated requests return 401
- Membership is verified at the **GraphQL resolver level** for every mutation — client-side guards are insufficient
- All members are equal — no permission elevation, no admin bypass

### Data Access

- `addMember` search queries only the caller's friend list — querying the global user table constitutes a security violation
- Profile data returned in search results and request cards is limited to publicly consented fields (username, avatar)
- All API communication over HTTPS

### PII & Data Handling

- No phone numbers, emails, or private profile data are exposed in search or request flows
- Friend graph data is session-scoped — not persisted externally or logged
- Self-remove does not cascade to content deletion — user data (contributions) remains part of the narative per right-to-erasure scope

---

## Pendo Tracking Plan

All events use the existing Pendo SDK (`rn-pendo-sdk 3.8.0`). Follow existing event naming conventions.

| Event Name | Trigger | Properties |
|---|---|---|
| `member_management_page_opened` | Member opens Management page | `narativeId`, `memberCount`, `pendingRequestCount` |
| `member_add_search_initiated` | Member taps "Add a friend" | `narativeId` |
| `member_added` | `addMember` mutation succeeds | `narativeId`, `addedUserId` |
| `member_add_duplicate_error` | Inline duplicate error shown | `narativeId` |
| `join_request_viewed` | Member opens pending request list | `narativeId`, `pendingCount` |
| `join_request_approved` | Member taps Approve | `narativeId`, `requesterId` |
| `join_request_rejected` | Member taps Reject | `narativeId` |
| `member_self_removed` | `leaveNarative` mutation succeeds | `narativeId` |
| `member_self_remove_cancelled` | Member cancels confirmation modal | `narativeId` |

**Success metrics tracked via Pendo:**
- Feature adoption: `member_management_page_opened` unique naratives / 30 days
- Request resolution: ratio of `join_request_approved` + `join_request_rejected` to total requests / 7 days
- Add funnel: `member_add_search_initiated` → `member_added` conversion rate

---

## Functional Requirements

### Access Control

- **FR1:** Members can access the Member Management page from the Members page of a narative they belong to
- **FR2:** Non-members cannot see the entry point to the Member Management page
- **FR3:** The system verifies membership at the API level before executing any management mutation
- **FR4:** All members of a narative have identical management capabilities — no permission tiers or admin roles

### Add Member

- **FR5:** Members can search for users to add by username
- **FR6:** Members can search for users to add by first name and/or last name
- **FR7:** Search results are scoped exclusively to the acting member's friend list
- **FR8:** Members can add a friend to the narative immediately with a single tap — no confirmation step
- **FR9:** The system prevents adding a user already in the narative and surfaces a clear inline message
- **FR10:** The system prevents adding a user not in the acting member's friend list

### Member Management Page

- **FR11:** Members can view the current member list from the Management page
- **FR12:** Members can see the count of pending join requests on the Management page entry point
- **FR13:** Members can see pending join requests ordered by recency
- **FR14:** The Management page reflects membership changes in real time without manual refresh

### Join Request Management

- **FR15:** Members can view a pending requester's avatar and username before acting
- **FR16:** Members can approve a pending join request, immediately adding the requester
- **FR17:** Members can reject a pending join request, permanently removing it for all members
- **FR18:** Once acted on by any member, a join request is no longer actionable by others (first-actor-wins)
- **FR19:** Rejected join requests are deleted — not persisted in a rejected state
- **FR20:** Approved join requests result in immediate full membership for the requester

### Self-Remove

- **FR21:** Members can initiate leaving a narative from the Management page
- **FR22:** The system presents a confirmation step before executing self-remove, noting that rejoining requires a member to re-add them
- **FR23:** After self-removing, the former member immediately loses access to the narative and Management page
- **FR24:** Self-remove does not delete content the member contributed to the narative

### Notifications

- **FR25:** The system sends a push notification to a user when added to a narative, naming the adder and the narative
- **FR26:** The system sends a push notification to all narative members when a join request is received, naming the requester and the narative
- **FR27:** The system sends a push notification to a requester when their request is approved, naming the narative
- **FR28:** Push token failures are handled silently — they do not fail management mutations

---

## Non-Functional Requirements

### Performance

- **NFR1:** Friend search results appear within **1 second** of input stopping (debounced query)
- **NFR2:** Management mutations complete and update the UI within **2 seconds** under normal network conditions
- **NFR3:** Push notifications are delivered within **5 seconds** of the triggering server action
- **NFR4:** The Management page loads (member list + pending requests) within **2 seconds** on a standard mobile connection

### Security

- **NFR5:** Unauthenticated management mutation requests return 401
- **NFR6:** Membership verification occurs at the GraphQL resolver level — not only on the client
- **NFR7:** Friend search queries only the authenticated user's friend list — results outside this scope constitute a security violation
- **NFR8:** Search and request card data is limited to user-consented profile fields (username, avatar)
- **NFR9:** All API communication over HTTPS

### Reliability

- **NFR10:** Push notification failures do not cause management mutations to fail — notifications are best-effort
- **NFR11:** Concurrent `approveJoinRequest` calls for the same request are idempotent — one membership created regardless of how many members act simultaneously
- **NFR12:** `approveJoinRequest` is atomic — membership creation and request deletion succeed together or neither commits
