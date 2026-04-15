---
feature: narative-member-management
phase: architecture
status: approved
linearTicket: NAR-461
prd: _bmad-output/features/narative-member-management/prd.md
uxSpec: _bmad-output/features/narative-member-management/ux-spec.md
stepsCompleted:
  - codebase-exploration
  - adr-decisions
  - data-model
  - graphql-api
  - typescript-interfaces
  - backend-file-structure
  - mobile-file-structure
  - state-management
  - security
  - performance
  - mobile-specific
  - edge-cases
  - testing
---

# Architecture — Narative Member Management

**Feature:** Narative Member Management  
**Linear:** NAR-461  
**PRD:** `_bmad-output/features/narative-member-management/prd.md`  
**UX Spec:** `_bmad-output/features/narative-member-management/ux-spec.md`  
**Stack:** React Native 0.81.5 + Expo ~54 · NestJS 11 · Apollo GraphQL · Prisma · PostgreSQL

---

## 1. Architecture Decision Log (ADL)

### ADR-001 — Add Member: Direct mutation, not join request
**Decision:** New `addNarativeMember(narativeId, userId)` mutation on `NarativeResolver`. Calls `narativeRepository.update()` with `memberToAddIdList`. Backend validates caller is a member and target is a friend of the caller.  
**Rationale:** Friend-add bypasses the request flow entirely (UX: ≤ 2 taps). Reuses the existing `memberToAddIdList` update path in `narativeRepository`. No new Prisma model needed.  
**Rejected alternative:** Creating a pre-approved `NarativeMembershipRequest` — unnecessary complexity, wrong semantic.

### ADR-002 — List Pending Requests: New query, member-gated
**Decision:** New `listNarativePendingRequests(narativeId): [NarativeMembershipRequest]` query on `NarativeMembershipRequestResolver`. Queries `NarativeMembershipRequest` records where `status = pending` AND `narativeId` matches. Caller must be a narative member (enforced in service).  
**Rationale:** The existing resolver only has mutations. A dedicated query keeps the pattern clean.  
**Note:** No new Prisma model, only a new `findManyPendingByNarative(narativeId)` repository method.

### ADR-003 — Self-Remove: New thin mutation on NarativeResolver
**Decision:** New `removeNarativeMember(narativeId)` mutation. Calls existing `narativeService.update()` with `memberToRemoveIdList: [callerId]`. The existing `NarativeEvent.updated` listener then auto-cancels any pending requests from that user.  
**Rationale:** Re-uses the existing member removal path end-to-end. Zero new DB logic.

### ADR-004 — Decline Fix: Change authorization from author-only to member-any
**Decision:** In `NarativeMembershipRequestService.decline()`, replace the `author.id === authorId` check with `narative.memberList.some(m => m.id === callerId)`.  
**Rationale:** Current code lets only the narative creator decline. PRD FR-17 requires any member to reject. This is a behavioral fix on existing code, not a new feature.

### ADR-005 — Notification Broadcasting: All members on request created
**Decision:** Change `NarativeMembershipRequestCreatedEvent` payload from `{ requesterId, narativeAuthorId }` to `{ requesterId, memberIdList }`. Update the notification listener to send push to all `memberIdList` entries.  
**Rationale:** There is no "author" in the member-equal model. All members must see incoming requests. The `authorId` field on `NarativeMembershipRequest` is the only place `narativeAuthorId` was used for routing; removing this routing dependency.

### ADR-006 — authorId: Keep column, keep writing to it, never read it
**Decision:** No migration. `authorId` stays in the schema as-is. `NarativeMembershipRequestRepository.create()` continues to write `narative.author.id` to `authorId` exactly as it does today — no change to the write path. The column is never read by any new code path.  
**Rationale:** The column carries historical data; no reason to change the write behavior. All reads of `author.id` are removed by ADR-004 (decline fix) and ADR-005 (notification change). The write stays identical; only the reads disappear.  
**Risk:** None. The write path is unchanged. No code reads `authorId` after ADR-004 and ADR-005 land.

### ADR-007 — Mobile Page: Extend existing members.tsx
**Decision:** Replace the contents of `src/app/(app)/narative/[id]/members.tsx` with the new `MembersPage.container`. The route already exists; no router change needed.  
**Rationale:** The existing file is a thin list (50 lines). Replacing its contents is cleaner than redirecting.

### ADR-008 — Friend Search: Reuse existing `users(isFriend: true)` query
**Decision:** The `FriendSearchSheet` container reuses `listFriendsForNarativeForm.query.gql` pattern — `users(filters: { searchQuery, isFriend: true })` with `useLazyQuery`. No new backend query needed.  
**Rationale:** This query already exists in `shared/infra/queries/listFriendsForNarativeForm` with exactly the fields needed. A new `.gql` in the `members` domain will simply copy this pattern with a different name for isolation.

### ADR-009 — Optimistic Updates: Local state removal, no Apollo cache manipulation
**Decision:** On approve/reject tap, immediately remove the request from local `pendingRequests` state (set in container). On server error, restore the removed item and show a toast. No `update` Apollo cache manipulation.  
**Rationale:** Apollo cache manipulation for list items is fragile. Since `listNarativePendingRequests` is polled only on mount, local state is the source of truth during the session. Simple and predictable.

### ADR-010 — Duplicate Add Prevention: Track `pendingAddUserId` in container
**Decision:** When user taps a search result, set `pendingAddUserId`. If the same `userId` appears in `narative.memberList` after adding, the container shows the duplicate error (passed as `duplicateUserId` prop to `FriendSearchSheet`). Clear on sheet close.  
**Rationale:** The UX spec requires inline duplicate detection without navigating away. Tracking the pending ID keeps FriendSearchSheet stateless.

---

## 2. Data Model Changes

### 2.1 No Schema Migration Required

`authorId` is kept as-is — NOT NULL, same type, same schema. No Prisma migration file needed.

`NarativeMembershipRequestRepository.create()` continues to write `narative.author.id` to `authorId` exactly as today — write path is unchanged. All reads of `author.id` are eliminated by ADR-004 and ADR-005.

### 2.2 No New Prisma Models

All member management operations use existing models:
- `NarativeMembershipRequest` — join request lifecycle
- `Narative.memberList` — member add/remove via existing `memberToAddIdList` / `memberToRemoveIdList`

---

## 3. GraphQL API

### 3.1 New Queries

```graphql
# Lists all pending join requests for a narative.
# Caller must be a member of the narative.
query listNarativePendingRequests($narativeId: ID!): [NarativeMembershipRequest!]!
```

**Response shape** (existing `NarativeMembershipRequest` GQL entity):
```graphql
type NarativeMembershipRequest {
  id: String!
  requester: User
  narative: Narative
  date: DateTime
  status: RequestStatus
}
```

### 3.2 New Mutations

```graphql
# Adds a friend directly to the narative. Caller must be a member.
# Target must be a friend of the caller.
mutation addNarativeMember(narativeId: ID!, userId: ID!): Narative

# Removes the calling user from the narative.
mutation removeNarativeMember(narativeId: ID!): Narative
```

### 3.3 Modified Mutations (behavior change, same signature)

```graphql
# No signature change. Authorization changes: any member can now reject (not just narative author).
mutation declineNarativeMembershipRequest(id: ID!): NarativeMembershipRequest
```

### 3.4 Unchanged Mutations (reused as-is)

```graphql
mutation acceptNarativeMembershipRequest(id: ID!): NarativeMembershipRequest
mutation sendNarativeMembershipRequest(narativeId: ID!): Narative
mutation cancelNarativeMembershipRequest(narativeId: ID!): Narative
```

---

## 4. TypeScript Interfaces

### 4.1 Mobile Domain Entities

```typescript
// mobile/src/domains/narative/members/domain/entities/member.entity.ts
export type NarativeMember = {
  userId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
};

// mobile/src/domains/narative/members/domain/entities/join-request.entity.ts
export type NarativeJoinRequest = {
  requestId: string;
  userId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
  date: string; // ISO
};
```

### 4.2 Component Props

```typescript
// JoinRequestCard
export type JoinRequestCardProps = {
  requestId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
  onApprove: (requestId: string) => void;
  onReject: (requestId: string) => void;
  isApproving?: boolean;
  isRejecting?: boolean;
};

// FriendSearchSheet
export type FriendSearchResult = {
  userId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
};

export type FriendSearchSheetProps = {
  narativeId: string;
  isVisible: boolean;
  onClose: () => void;
  onMemberAdded: (userId: string) => void;
  // State props (controlled by container)
  searchResults: FriendSearchResult[];
  isLoading: boolean;
  searchQuery: string;
  emptyMessage?: string;
  duplicateUserId?: string;
  duplicateMessage?: string;
  isOffline?: boolean;
};
```

### 4.3 Backend Service Inputs

```typescript
// New service method signatures (NarativeMembershipRequestService)
listPendingByNarative(narativeId: string, callerId: string): Promise<NarativeMembershipRequestEntity[]>

// Modified decline signature (callerId replaces authorId)
decline(id: string, callerId: string): Promise<NarativeMembershipRequestEntity | null>

// New NarativeService methods
addFriendToNarative(narativeId: string, friendUserId: string, callerId: string): Promise<NarativeEntity | null>
removeSelfFromNarative(narativeId: string, callerId: string): Promise<NarativeEntity | null>
```

---

## 5. Backend File Structure

### 5.1 Modified Files

```
backend/src/
├── prisma/
│   └── schema.prisma                                          MODIFY — authorId nullable
│
├── narative-membership-request/
│   ├── narative-membership-request.repository.ts             MODIFY
│   │   └── + findManyPendingByNarative(narativeId)
│   │   └── create() — unchanged (authorId write stays as-is)
│   │
│   └── narative-membership-request.service.ts                MODIFY
│       └── + listPendingByNarative(narativeId, callerId)
│       └── MODIFY decline() — member check replaces author check
│       └── MODIFY create() — event payload only: narativeAuthorId → memberIdList
│
├── narative/
│   └── services/
│       └── narative.service.ts                               MODIFY
│           └── + addFriendToNarative(narativeId, friendUserId, callerId)
│           └── + removeSelfFromNarative(narativeId, callerId)
│
├── graphql/
│   ├── narative-membership-request/
│   │   └── resolvers/
│   │       └── narative-membership-request.resolver.ts       MODIFY
│   │           └── + Query: listNarativePendingRequests
│   │           └── MODIFY declineNarativeMembershipRequest — passes callerId
│   │
│   └── narative/
│       └── resolvers/
│           └── narative.resolver.ts                          MODIFY
│               └── + Mutation: addNarativeMember
│               └── + Mutation: removeNarativeMember
│
├── events/
│   └── narative-membership-request/
│       └── created.event.ts                                  MODIFY
│           └── payload: { requesterId, memberIdList } (replaces narativeAuthorId)
│
└── notification/
    └── listeners/
        └── narative-membership-request.listener.ts           MODIFY
            └── handleNarativeMembershipRequestCreatedEvent — broadcast to memberIdList
```

### 5.2 New Files

No new backend files beyond the domain modifications above. No migration file.

---

## 6. Mobile File Structure

### 6.1 New Domain

```
mobile/src/domains/narative/members/
├── domain/
│   └── entities/
│       ├── member.entity.ts
│       └── join-request.entity.ts
│
├── infra/
│   ├── queries/
│   │   └── listNarativePendingRequests/
│   │       ├── listNarativePendingRequests.query.gql
│   │       └── listNarativePendingRequests.query.hooks.ts    (generated)
│   │
│   └── mutations/
│       ├── addNarativeMember/
│       │   ├── addNarativeMember.mutation.gql
│       │   └── addNarativeMember.mutation.hooks.ts           (generated)
│       │
│       └── removeNarativeMember/
│           ├── removeNarativeMember.mutation.gql
│           └── removeNarativeMember.mutation.hooks.ts        (generated)
│
└── view/
    ├── components/
    │   ├── JoinRequestCard/
    │   │   ├── JoinRequestCard.component.tsx
    │   │   ├── JoinRequestCard.styles.ts
    │   │   └── JoinRequestCard.stories.tsx                   (exists)
    │   │
    │   └── FriendSearchSheet/
    │       ├── FriendSearchSheet.component.tsx
    │       ├── FriendSearchSheet.styles.ts
    │       └── FriendSearchSheet.stories.tsx                 (exists)
    │
    └── containers/
        ├── MembersPage/
        │   ├── MembersPage.container.tsx                     (main orchestrator)
        │   └── MembersPage.styles.ts
        │
        ├── MemberList/
        │   ├── MemberList.container.tsx                      (members tab)
        │   └── MemberList.styles.ts
        │
        └── JoinRequests/
            ├── JoinRequests.container.tsx                    (requests tab)
            └── JoinRequests.styles.ts
```

### 6.2 Modified Files

```
mobile/src/app/(app)/narative/[id]/
└── members.tsx                                               MODIFY — replace contents with MembersPage.container
```

### 6.3 GQL Operations

**listNarativePendingRequests.query.gql**
```graphql
query listNarativePendingRequests($narativeId: ID!) {
  listNarativePendingRequests(narativeId: $narativeId) {
    id
    requester {
      id
      firstName
      lastName
      username
      profilePicture {
        id
        url
      }
    }
    date
    status
  }
}
```

**addNarativeMember.mutation.gql**
```graphql
mutation addNarativeMember($narativeId: ID!, $userId: ID!) {
  addNarativeMember(narativeId: $narativeId, userId: $userId) {
    id
    memberList {
      id
      firstName
      lastName
      username
      profilePicture {
        id
        url
      }
    }
  }
}
```

**removeNarativeMember.mutation.gql**
```graphql
mutation removeNarativeMember($narativeId: ID!) {
  removeNarativeMember(narativeId: $narativeId) {
    id
  }
}
```

---

## 7. State Management

### 7.1 MembersPage Container — State Model

```typescript
// All state lives in MembersPage.container.tsx
const [activeTab, setActiveTab] = useState<'members' | 'requests'>('members');
const [isFriendSearchVisible, setIsFriendSearchVisible] = useState(false);
const [searchQuery, setSearchQuery] = useState('');
const [pendingRequests, setPendingRequests] = useState<NarativeJoinRequest[]>([]);
const [approvingIds, setApprovingIds] = useState<Set<string>>(new Set());
const [rejectingIds, setRejectingIds] = useState<Set<string>>(new Set());
const [pendingAddUserId, setPendingAddUserId] = useState<string | null>(null);
```

### 7.2 Optimistic Update Flow — Approve / Reject

```
User taps Approve on request R
  → setApprovingIds(prev => new Set(prev).add(R.requestId))
  → optimistically remove R from pendingRequests
  → call acceptNarativeMembershipRequest({ variables: { id: R.requestId } })
  → on error: restore R to pendingRequests, clear approvingIds, show toast
  → on success: setApprovingIds(prev => { prev.delete(R.requestId); return new Set(prev); })
               refetch memberList (or add member optimistically)
```

### 7.3 Optimistic Update Flow — Add Friend

```
User taps search result U in FriendSearchSheet
  → setPendingAddUserId(U.userId)
  → call addNarativeMember({ variables: { narativeId, userId: U.userId } })
  → on success: memberList updated from server response; close sheet
  → on duplicate (userId in memberList): show duplicateMessage, keep sheet open
  → on error: toast, clear pendingAddUserId
```

### 7.4 Friend Search — Debounce

```
User types in FriendSearchSheet search input
  → setSearchQuery(value) [immediate — drives controlled input]
  → debounce 300ms
  → callListFriendsMembersQuery({ variables: { searchQuery: value } })
```

### 7.5 Data Dependencies

| Data | Source | Query |
|---|---|---|
| Member list | Apollo, initial load | `getNarativeMemberList` (existing) |
| Pending requests count (badge) | Apollo, initial load | `listNarativePendingRequests` |
| Friend search results | Apollo, lazy on keystroke | `listFriendsForMembers` (new .gql, same pattern as `listFriendsForNarativeForm`) |

---

## 8. Security Model

### 8.1 Backend Authorization Rules

| Operation | Check | Location |
|---|---|---|
| `listNarativePendingRequests` | Caller is narative member | `NarativeMembershipRequestService.listPendingByNarative` |
| `addNarativeMember` | Caller is narative member + target is caller's friend | `NarativeService.addFriendToNarative` |
| `removeNarativeMember` | Caller is narative member | `NarativeService.removeSelfFromNarative` |
| `acceptNarativeMembershipRequest` | Caller is narative member | `NarativeMembershipRequestService.accept` (already implemented) |
| `declineNarativeMembershipRequest` | Caller is narative member | `NarativeMembershipRequestService.decline` (changing from author-only) |

All mutations are protected by `@UseGuards(GqlAuthGuard)` at resolver level.

### 8.2 Friend Scope Enforcement

`addNarativeMember` backend validation:
1. Fetch caller's friend list from `UserRepository` (or via Prisma `friendList` relation)
2. Assert `friendList.some(f => f.id === userId)` — reject with `ForbiddenException` if not a friend
3. Assert caller is narative member — reject with `ForbiddenException` if not

This double-check means even a crafted GraphQL call cannot add a stranger.

### 8.3 Non-Member Access

`listNarativePendingRequests` must return 403 (not empty list) for non-members. The `ForbiddenException` from NestJS maps to a GraphQL error, which Apollo surfaces in the `error` field.

---

## 9. Performance Constraints (from NFRs)

| NFR | Target | Implementation |
|---|---|---|
| NFR1 — Friend search | ≤ 1s | Debounce 300ms client-side; `users` query with `isFriend + searchQuery` already indexed on `username/firstName/lastName` |
| NFR2 — Approve/reject mutation | ≤ 2s | Optimistic update means 0ms perceived latency |
| NFR3 — Push notification | ≤ 5s | Async event-driven via `EventEmitter2` + Expo push SDK; fire-and-forget after mutation resolves |
| NFR4 — Members page load | ≤ 2s | Two parallel queries (`getNarativeMemberList` + `listNarativePendingRequests`) on mount |

---

## 10. Mobile-Specific Constraints

| Constraint | Implementation |
|---|---|
| Offline handling | `FriendSearchSheet` receives `isOffline` prop; network state from existing `useNetInfo()` pattern |
| Keyboard avoidance | Evanescent `BottomSheet` has `avoidKeyboard: true` built-in via `react-native-modal` — no extra work |
| Emotion `styled()` | All styles in `.styles.ts` files, `styled()` from `@emotion/native` — no `StyleSheet.create()` |
| react-intl | All visible strings via `<FormattedMessage>` or `useIntl().formatMessage()` |
| Portrait-only | Inherited from app-level config — no orientation lock needed at component level |
| Pendo | Track events in containers (not components) using `usePendoTrackEvent()` hook |
| Evanescent only | `Tabs`, `Tab`, `TabList`, `BottomSheet`, `UserLine`, `Avatar`, `Button`, `Skeleton`, `Badge` — no custom primitives |

### 10.1 Pendo Events (from PRD tracking plan)

| Event | Trigger location |
|---|---|
| `member_management_page_opened` | `MembersPage.container` — on mount |
| `member_add_search_initiated` | `FriendSearchSheet.container` — on first keystroke |
| `member_added` | `MembersPage.container` — on `addNarativeMember` success |
| `join_request_approved` | `JoinRequests.container` — on accept success |
| `join_request_rejected` | `JoinRequests.container` — on decline success |
| `member_removed_self` | `MembersPage.container` — on `removeNarativeMember` success |

---

## 11. Edge Cases

| Scenario | Handling |
|---|---|
| Concurrent approve/reject (two members act simultaneously) | First-actor-wins: the second `accept/decline` call receives `null` (request already gone). Mobile: the stale card silently disappears from `pendingRequests` on next refetch. No error shown. |
| User adds friend who becomes a member between search and tap | `addNarativeMember` backend detects duplicate via `memberList.some(m => m.id === userId)` and returns `ForbiddenException`. Mobile shows duplicate error in FriendSearchSheet. |
| Self-removal as last member | `removeNarativeMember` succeeds. The narative becomes an empty narative (existing behavior — no auto-delete enforced). |
| Join request for narative with no members (edge case post-cleanup) | `listPendingByNarative` returns requests; no approver. Requests remain pending indefinitely. Acceptable for MVP — no auto-expiry. |
| Offline: user taps approve | Mutation fails. Restore optimistic removal. Toast: "No connection." |
| Push token missing for a member | `PushNotificationService` silently skips tokens that fail `Expo.isExpoPushToken()` — existing behavior, safe. |
| Requester already member when request is processed | `accept()` calls `narativeRepository.update()` with `memberToAddIdList`. The `@@unique([narativeId, requesterId])` at Prisma layer prevents duplicate member relations. |

---

## 12. Testing Strategy

### 12.1 Backend Unit Tests

| Test | File |
|---|---|
| `listPendingByNarative` — returns only `pending` records | `narative-membership-request.service.spec.ts` |
| `listPendingByNarative` — throws 403 if caller not member | `narative-membership-request.service.spec.ts` |
| `decline` — any member can decline (not just author) | `narative-membership-request.service.spec.ts` |
| `addFriendToNarative` — rejects non-friend target | `narative.service.spec.ts` |
| `addFriendToNarative` — rejects non-member caller | `narative.service.spec.ts` |
| `removeSelfFromNarative` — removes caller from memberList | `narative.service.spec.ts` |

### 12.2 Mobile Unit Tests

| Test | File |
|---|---|
| `JoinRequestCard` renders approve/reject buttons | `JoinRequestCard.component.test.tsx` |
| `JoinRequestCard` shows spinner during `isApproving` | `JoinRequestCard.component.test.tsx` |
| `FriendSearchSheet` shows empty state when `searchResults = []` | `FriendSearchSheet.component.test.tsx` |
| `FriendSearchSheet` shows duplicate error when `duplicateUserId` set | `FriendSearchSheet.component.test.tsx` |
| `MembersPage` passes correct `count` to Requests tab | `MembersPage.container.test.tsx` |
| Optimistic removal: request gone before server response | `JoinRequests.container.test.tsx` |
| Optimistic rollback: request restored on server error | `JoinRequests.container.test.tsx` |

### 12.3 Storybook Coverage

Already created:
- `JoinRequestCard.stories.tsx` — Default, NoAvatar, ApprovingInFlight, RejectingInFlight, Interactive
- `FriendSearchSheet.stories.tsx` — Idle, Loading, WithResults, EmptyResults, DuplicateError, Offline, Interactive

---

## Completeness Check

### PRD Acceptance Criteria Coverage

| AC | Architecture Coverage |
|---|---|
| Any member can view the member list | `getNarativeMemberList` (existing query) used in `MemberList.container` |
| Any member can add a Nara friend | `addNarativeMember` mutation (new) + `FriendSearchSheet` + `MembersPage` |
| Any member can approve a join request | `acceptNarativeMembershipRequest` (existing, already member-scoped) |
| Any member can reject a join request | `declineNarativeMembershipRequest` (ADR-004 fix) |
| Any member can self-remove | `removeNarativeMember` mutation (new) |
| Join request badge count visible | `listNarativePendingRequests` drives `Tab count` prop |
| Offline graceful degradation | `isOffline` prop in `FriendSearchSheet`, mutation error handling |
| Push notifications to all members on new request | ADR-005 — `memberIdList` broadcast |
| Non-members cannot access | `GqlAuthGuard` + member check in all new service methods |
| Duplicate add detection | ADR-010 — `pendingAddUserId` + backend double-check |

### UX Spec Screen Coverage

| Screen | Container |
|---|---|
| Members tab (member list) | `MemberList.container` inside `MembersPage` |
| Requests tab (pending approvals) | `JoinRequests.container` inside `MembersPage` |
| Add Friend sheet | `FriendSearchSheet.component` (controlled by `MembersPage`) |
| Duplicate error state in sheet | `FriendSearchSheet` `duplicateUserId` prop |
| Empty requests state | `JoinRequests.container` — empty list branch |
| Self-remove confirmation | Action within `MemberList.container` (ActionSheet or confirmation modal) |

---

**Gate 3 — Architecture Review**

To approve this architecture and proceed to story generation, respond: **A**  
To request revisions, respond: **R [reason]**
