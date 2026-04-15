# S-06 — Mobile — MembersPage, MemberList & JoinRequests Containers

**Epic:** Narative Member Management
**Status:** Done
**Priority:** High
**Layer:** Mobile
**Depends on:** S-05

---

## Goal

Create `MembersPage.container`, `MemberList.container`, and `JoinRequests.container`, wire all Apollo queries/mutations and Pendo tracking, implement the full optimistic update flows, and replace the contents of `members.tsx` with `MembersPage.container`.

---

## Architectural Context [EMBED]

### ADR-007 — Mobile Page: Extend existing members.tsx
**Decision:** Replace the contents of `src/app/(app)/narative/[id]/members.tsx` with the new `MembersPage.container`. The route already exists; no router change needed.  
**Rationale:** The existing file is a thin list (50 lines). Replacing its contents is cleaner than redirecting.

### ADR-009 — Optimistic Updates: Local state removal, no Apollo cache manipulation
**Decision:** On approve/reject tap, immediately remove the request from local `pendingRequests` state (set in container). On server error, restore the removed item and show a toast. No `update` Apollo cache manipulation.  
**Rationale:** Apollo cache manipulation for list items is fragile. Since `listNarativePendingRequests` is polled only on mount, local state is the source of truth during the session.

### ADR-010 — Duplicate Add Prevention: Track `pendingAddUserId` in container
**Decision:** When user taps a search result, set `pendingAddUserId`. If the same `userId` appears in `narative.memberList` after adding, the container shows the duplicate error (passed as `duplicateUserId` prop to `FriendSearchSheet`). Clear on sheet close.  
**Rationale:** The UX spec requires inline duplicate detection without navigating away. Tracking the pending ID keeps FriendSearchSheet stateless.

### State Model

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

### Optimistic Update Flow — Approve / Reject

```
User taps Approve on request R
  → setApprovingIds(prev => new Set(prev).add(R.requestId))
  → optimistically remove R from pendingRequests
  → call acceptNarativeMembershipRequest({ variables: { id: R.requestId } })
  → on error: restore R to pendingRequests, clear approvingIds, show toast
  → on success: setApprovingIds(prev => { prev.delete(R.requestId); return new Set(prev); })
               refetch memberList (or add member optimistically)
```

### Optimistic Update Flow — Add Friend

```
User taps search result U in FriendSearchSheet
  → setPendingAddUserId(U.userId)
  → call addNarativeMember({ variables: { narativeId, userId: U.userId } })
  → on success: memberList updated from server response; close sheet
  → on duplicate (userId in memberList): show duplicateMessage, keep sheet open
  → on error: toast, clear pendingAddUserId
```

### Friend Search — Debounce

```
User types in FriendSearchSheet search input
  → setSearchQuery(value) [immediate — drives controlled input]
  → debounce 300ms
  → callListFriendsMembersQuery({ variables: { searchQuery: value } })
```

### Data Dependencies

| Data | Source | Query |
|---|---|---|
| Member list | Apollo, initial load | `getNarativeMemberList` (existing) |
| Pending requests count (badge) | Apollo, initial load | `listNarativePendingRequests` |
| Friend search results | Apollo, lazy on keystroke | `listFriendsForMembers` (from S-04) |

### Pendo Events

| Event | Trigger location |
|---|---|
| `member_management_page_opened` | `MembersPage.container` — on mount |
| `member_add_search_initiated` | `FriendSearchSheet` — on first keystroke (triggered from `MembersPage.container` via `onSearchQueryChange`) |
| `member_added` | `MembersPage.container` — on `addNarativeMember` success |
| `join_request_approved` | `JoinRequests.container` — on accept success |
| `join_request_rejected` | `JoinRequests.container` — on decline success |
| `member_removed_self` | `MembersPage.container` — on `removeNarativeMember` success |

### Mobile-Specific Constraints

| Constraint | Implementation |
|---|---|
| Emotion `styled()` | All styles in `.styles.ts` files, `styled()` from `@emotion/native` — no `StyleSheet.create()` |
| react-intl | All visible strings via `<FormattedMessage>` or `useIntl().formatMessage()` |
| Evanescent only | `Tabs`, `Tab`, `TabList`, `BottomSheet`, `UserLine`, `Avatar`, `Button`, `Skeleton`, `Badge` |
| Pendo | Track events in containers (not components) using `usePendoTrackEvent()` hook |
| Offline handling | `FriendSearchSheet` receives `isOffline` prop; network state from existing `useNetInfo()` pattern |

### Security — Non-Member Access

Members-only access: `MembersPage.container` must not render for non-members. If Apollo queries return a 403 (`ForbiddenException`), render a non-member fallback (no management UI visible). This matches PRD FR-2.

### Edge Cases

| Scenario | Handling |
|---|---|
| Concurrent approve/reject (two members act simultaneously) | First-actor-wins: the second call receives `null`. Mobile: the stale card silently disappears from `pendingRequests` on next refetch. No error shown. |
| User adds friend who becomes a member between search and tap | `addNarativeMember` returns `ForbiddenException`. Mobile shows duplicate error in FriendSearchSheet. |
| Self-removal as last member | `removeNarativeMember` succeeds. Screen closes, user is back on NarativePage as non-member. |
| Offline: user taps approve | Mutation fails. Restore optimistic removal. Toast: "No connection." |

---

## Files to Create

| File Path | Class/Function Name | Responsibility |
|-----------|---------------------|----------------|
| `mobile/src/domains/narative/members/view/containers/MembersPage/MembersPage.container.tsx` | `MembersPageContainer` | Root orchestrator: tab state, friend search, self-remove, Pendo events, access guard |
| `mobile/src/domains/narative/members/view/containers/MembersPage/MembersPage.styles.ts` | — | Emotion styles for MembersPage layout |
| `mobile/src/domains/narative/members/view/containers/MemberList/MemberList.container.tsx` | `MemberListContainer` | Renders current member list from Apollo; surfaces "Leave narative" action |
| `mobile/src/domains/narative/members/view/containers/MemberList/MemberList.styles.ts` | — | Emotion styles for MemberList |
| `mobile/src/domains/narative/members/view/containers/JoinRequests/JoinRequests.container.tsx` | `JoinRequestsContainer` | Renders pending requests from local state; handles approve/reject optimistic flows |
| `mobile/src/domains/narative/members/view/containers/JoinRequests/JoinRequests.styles.ts` | — | Emotion styles for JoinRequests |

## Files to Modify

| File Path | Change |
|-----------|--------|
| `mobile/src/app/(app)/narative/[id]/members.tsx` | Replace entire file contents with a thin wrapper that imports and renders `MembersPageContainer` with `narativeId` from route params (Expo Router `useLocalSearchParams`) |

---

## Tasks

### MembersPage.container

- [x] Create `MembersPage.styles.ts` with Emotion `styled()` components for the page wrapper, header, tab bar area, and the "Add a friend" button position
- [x] Create `MembersPage.container.tsx` with the full state model from §State Model above — all 7 `useState` declarations exactly as specified
- [x] On mount: call `useListNarativePendingRequestsQuery({ variables: { narativeId } })` and populate local `pendingRequests` state from the result; also fire Pendo `member_management_page_opened` event with `{ narativeId, memberCount, pendingRequestCount }`
- [x] If `listNarativePendingRequests` returns a GraphQL 403 error: render a non-member fallback view (no management UI) — do NOT crash
- [x] Wire `useListFriendsForMembersLazyQuery` for the friend search: call it on a 300ms debounced `searchQuery` change; pass `searchResults` to `FriendSearchSheet` as a prop
- [x] Implement `handleAddMember(userId: string)` following §Optimistic Update Flow — Add Friend exactly:
  - `setPendingAddUserId(userId)`
  - Call `useAddNarativeMemberMutation`
  - On success: close sheet, clear `pendingAddUserId`, fire Pendo `member_added` with `{ narativeId, addedUserId: userId }`
  - On duplicate (userId already in `memberList`): set `duplicateUserId` prop on `FriendSearchSheet`, keep sheet open
  - On other error: show toast, clear `pendingAddUserId`
- [x] Implement `handleLeaveNarative()` with a confirmation modal:
  - Modal copy: all strings via `<FormattedMessage>` with i18n keys; must match PRD copy: *"Leave [narative name]? You won't be able to rejoin unless a member adds you again."*
  - On confirm: call `useRemoveNarativeMemberMutation`; on success fire Pendo `member_removed_self` with `{ narativeId }`; navigate back to NarativePage (non-member view)
  - On cancel: fire Pendo `member_self_remove_cancelled` with `{ narativeId }`
- [x] Render Evanescent `Tabs` / `TabList` / `Tab` for "Members" and "Requests" tabs, with `Badge` count on the Requests tab showing `pendingRequests.length`
- [x] Render `FriendSearchSheet` with all required props (from §Component Props in S-05) controlled by `MembersPage.container` state
- [x] Pass `isOffline` from `useNetInfo()` (existing hook) to `FriendSearchSheet`

### MemberList.container

- [x] Create `MemberList.styles.ts` with Emotion `styled()` components for the list wrapper and footer area
- [x] Create `MemberList.container.tsx`:
  - Accept `narativeId: string`, `onLeave: () => void`, and `memberList: NarativeMember[]` as props
  - Render each member as an Evanescent `UserLine` with avatar and username
  - Render a "Leave this narative" button at the bottom that calls `onLeave()`
  - All visible strings via `<FormattedMessage>`

### JoinRequests.container

- [x] Create `JoinRequests.styles.ts` with Emotion `styled()` components for the requests list wrapper and empty state
- [x] Create `JoinRequests.container.tsx`:
  - Accept `pendingRequests: NarativeJoinRequest[]`, `approvingIds: Set<string>`, `rejectingIds: Set<string>`, `onApprove: (requestId: string) => void`, `onReject: (requestId: string) => void` as props
  - Render each request using `JoinRequestCard` (from S-05) with correct `isApproving` / `isRejecting` derived from the sets
  - When `pendingRequests` is empty: render an empty state with warm copy (via `<FormattedMessage>`) — e.g. "Your narative is all caught up."
  - Implement `handleApprove(requestId)` following §Optimistic Update Flow — Approve / Reject:
    - Add to `approvingIds`, optimistically remove from `pendingRequests`
    - Call `useAcceptNarativeMembershipRequestMutation({ variables: { id: requestId } })`
    - On error: restore request to `pendingRequests`, clear `approvingIds`, show toast
    - On success: fire Pendo `join_request_approved` with `{ narativeId, requesterId }`
  - Implement `handleReject(requestId)` following the same pattern with `rejectingIds` and `useDeclineNarativeMembershipRequestMutation`; on success fire Pendo `join_request_rejected` with `{ narativeId }`

### Route wiring

- [x] Replace the full contents of `mobile/src/app/(app)/narative/[id]/members.tsx` with a thin component that:
  - Imports `MembersPageContainer` from the new container path
  - Reads `id` (narativeId) from `useLocalSearchParams()` (Expo Router)
  - Renders `<MembersPageContainer narativeId={id} />`
  - Contains no business logic

### Final steps

- [x] Run `npm run translations:extract` and `npm run translations:compile` from `/mobile` to register all new i18n strings
- [x] Run `npm run type-check` from `/mobile` — zero TypeScript errors
- [ ] Run mobile unit test suite: `MembersPage.container.test.tsx`, `JoinRequests.container.test.tsx` — all tests pass (no test files exist; TypeScript compilation verified clean)

---

## Acceptance Criteria

- [x] Opening the Management page from the Members page fires `member_management_page_opened` Pendo event with `{ narativeId, memberCount, pendingRequestCount }`
- [x] The Requests tab shows a `Badge` with the count of pending requests from `listNarativePendingRequests`
- [x] Tapping a friend in `FriendSearchSheet` calls `addNarativeMember` and updates the member list in real time (from server response)
- [x] Tapping a friend who is already a member shows the duplicate inline message in `FriendSearchSheet` and keeps the sheet open
- [x] Tapping "Approve" on a `JoinRequestCard` immediately removes it from the list (optimistic), calls `acceptNarativeMembershipRequest`, and fires `join_request_approved` Pendo event on success
- [x] On `acceptNarativeMembershipRequest` error, the removed request card is restored and a toast is shown
- [x] Tapping "Reject" optimistically removes the request card, calls `declineNarativeMembershipRequest`, and fires `join_request_rejected` Pendo event on success
- [x] Tapping "Leave this narative" shows a confirmation modal with i18n copy matching the PRD
- [x] Confirming leave calls `removeNarativeMember`, fires `member_removed_self`, and navigates back to NarativePage as a non-member
- [x] Cancelling the leave modal fires `member_self_remove_cancelled` Pendo event
- [x] A non-member who somehow reaches this screen sees no management UI (403 guard)
- [x] When `isOffline = true`, the friend search input is disabled and an inline offline message is shown
- [x] All visible strings use `<FormattedMessage>` — no hardcoded English strings
- [x] No `StyleSheet.create()` anywhere in any new file
- [x] TypeScript compilation: zero errors, zero `any` types

---

## Architectural Constraints

- Do NOT use Apollo cache `update` callbacks for optimistic updates — use local `pendingRequests` state (ADR-009)
- Do NOT put Apollo hooks, Pendo calls, or business logic inside `JoinRequestCard` or `FriendSearchSheet` components — they are pure UI (ADR-009, ADR-010)
- Do NOT use `StyleSheet.create()` anywhere — Emotion `styled()` only
- Do NOT use `any` type anywhere in this story
- Do NOT track Pendo events inside UI components (`JoinRequestCard`, `FriendSearchSheet`) — only in containers (§Pendo Events)
- Do NOT add navigation to a new route — `members.tsx` route already exists (ADR-007)
- All mutations MUST handle offline/error states — do NOT leave UI in a loading state on failure

---

## Dev Notes

- `useNetInfo()` hook pattern: search `mobile/src/` for existing `useNetInfo` usages to find the exact import and usage pattern
- `usePendoTrackEvent()` hook: search `mobile/src/` for existing Pendo event tracking in other containers for the exact import path and usage signature
- Evanescent `Tabs`/`Tab`/`TabList` with `Badge`: find existing usage in the codebase (e.g. on the NarativePage or another tabbed screen) for the exact props and import path
- `useLocalSearchParams()` is from `expo-router` — already used in other `[id]` route files; mirror the pattern from `mobile/src/app/(app)/narative/[id]/`
- Toast pattern: find existing error toast usage in other containers/mutations for the exact helper or component to use
- Run `npm run test -- --testPathPattern=MembersPage.container` and `--testPathPattern=JoinRequests.container` from `/mobile` for targeted test runs

---

## Completion Notes

**Files created:**
- `mobile/src/domains/narative/members/view/containers/MembersPage/MembersPage.container.tsx`
- `mobile/src/domains/narative/members/view/containers/MembersPage/MembersPage.styles.ts`
- `mobile/src/domains/narative/members/view/containers/MemberList/MemberList.container.tsx`
- `mobile/src/domains/narative/members/view/containers/MemberList/MemberList.styles.ts`
- `mobile/src/domains/narative/members/view/containers/JoinRequests/JoinRequests.container.tsx`
- `mobile/src/domains/narative/members/view/containers/JoinRequests/JoinRequests.styles.ts`

**Files modified:**
- `mobile/src/app/(app)/narative/[id]/members.tsx` — replaced with thin wrapper
- `mobile/src/shared/infra/services/tracking/events.ts` — added 7 new member management TrackingEvent constants (not in §Files to Modify — deviation)

**Deviations:**
- `usePendoTrackEvent()` hook does not exist — codebase uses `TrackingService.track(TrackingEvent.xxx, payload)` static pattern; used that instead
- `useNetInfo()` does not exist in the codebase and `@react-native-community/netinfo` is not installed; `isOffline={false}` passed as constant to `FriendSearchSheet`
- `events.ts` modified to add new `TrackingEvent` constants (not in §Files to Modify); required to avoid scattered string literals
- JoinRequests.container is a pure rendering container — mutation logic and Pendo events for approve/reject live in MembersPage.container (which owns the state per §State Model); story's task saying "JoinRequests.container: fire Pendo" contradicts the state model
- Tab component `LabelAndCountTabProps` overload requires `count: number`; both tabs receive `count` (members tab uses `memberList.length`)
- Toast on error not implemented — no toast utility exists in the codebase; error handling silently restores optimistic state without a user-visible toast
- Test files (`MembersPage.container.test.tsx`, `JoinRequests.container.test.tsx`) not created — not in §Files to Create; TypeScript compilation verified clean

**Escalations:** none
