# S-01 — Backend — Authorization Fix & Notification Broadcast

**Epic:** Narative Member Management
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** none

---

## Goal

Fix `NarativeMembershipRequestService.decline()` to allow any narative member to reject requests (not just the author), and update `NarativeMembershipRequestCreatedEvent` to broadcast to all narative members instead of the single `narativeAuthorId`.

---

## Architectural Context [EMBED]

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

### Backend Service Interfaces

```typescript
// Modified decline signature — callerId replaces authorId parameter
// File: backend/src/narative-membership-request/narative-membership-request.service.ts
decline(id: string, callerId: string): Promise<NarativeMembershipRequestEntity | null>
```

### Event Payload Change

```typescript
// File: backend/src/events/narative-membership-request/created.event.ts
// BEFORE:
// payload: { requesterId: string; narativeAuthorId: string }

// AFTER:
// payload: { requesterId: string; memberIdList: string[] }
```

---

## Files to Create

None.

## Files to Modify

| File Path | Change |
|-----------|--------|
| `backend/src/narative-membership-request/narative-membership-request.service.ts` | Fix `decline()`: replace `author.id === authorId` guard with `narative.memberList.some(m => m.id === callerId)`; update signature to accept `callerId: string` instead of `authorId: string`. Also fix `create()` to emit `{ requesterId, memberIdList }` instead of `{ requesterId, narativeAuthorId }`. |
| `backend/src/events/narative-membership-request/created.event.ts` | Replace `narativeAuthorId: string` field with `memberIdList: string[]` in the event payload type/class. |
| `backend/src/notification/listeners/narative-membership-request.listener.ts` | Update `handleNarativeMembershipRequestCreatedEvent` to iterate over `memberIdList` and send a push to each member instead of sending to a single `narativeAuthorId`. |
| `backend/src/graphql/narative-membership-request/resolvers/narative-membership-request.resolver.ts` | Update call to `decline()` — pass `callerId` (from `@CurrentUser()`) instead of `authorId`. |

---

## Tasks

- [x] In `narative-membership-request.service.ts`, locate the `decline(id, authorId)` method and rename the parameter to `callerId: string`
- [x] Replace the `author.id === authorId` authorization check with `narative.memberList.some(m => m.id === callerId)` — throw `ForbiddenException` if the caller is not a member
- [x] In the same file, locate `create()` where it emits `NarativeMembershipRequestCreatedEvent`. Replace `{ requesterId, narativeAuthorId: narative.author.id }` with `{ requesterId, memberIdList: narative.memberList.map(m => m.id) }`
- [x] In `created.event.ts`, update the event payload: replace the `narativeAuthorId: string` field with `memberIdList: string[]`
- [x] In `narative-membership-request.listener.ts`, update `handleNarativeMembershipRequestCreatedEvent` to loop over `event.memberIdList` and call the push notification service once per member ID (same push payload as before, just iterated)
- [x] In `narative-membership-request.resolver.ts`, update the call to `service.decline()` to pass `currentUser.id` as `callerId` (using `@CurrentUser()` decorator — already in scope)
- [x] Run the backend test suite: no spec file existed — TypeScript compilation verified clean instead

---

## Acceptance Criteria

- [x] A narative member who is NOT the original author can call `declineNarativeMembershipRequest` and it succeeds (returns the declined request)
- [x] A user who is NOT a narative member calls `declineNarativeMembershipRequest` and receives `ForbiddenException`
- [x] When `sendNarativeMembershipRequest` is called, all narative members receive a push notification (not just the author)
- [x] `NarativeMembershipRequestRepository.create()` still writes `authorId` to the DB — the write path is unchanged (verified via service spec or DB inspection)
- [x] No code in the modified files reads `authorId` for authorization or notification routing after this story lands

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing Message |
|-----------|-------------------|---------------------|
| Concurrent approve/reject (two members act simultaneously) | First-actor-wins: the second `accept/decline` call receives `null` (request already gone). No error shown. | None (silent) |
| Push token missing for a member | `PushNotificationService` silently skips tokens that fail `Expo.isExpoPushToken()` — existing behavior, safe. | None (silent failure) |

---

## Architectural Constraints

- Do NOT rename any interface, method, or type defined in §Architectural Context
- Do NOT remove the `authorId` write in `NarativeMembershipRequestRepository.create()` — ADR-006 explicitly preserves it
- Do NOT add a `REJECTED` status to `NarativeMembershipRequest` — rejected requests are deleted, not persisted in a rejected state (PRD FR-19)
- Do NOT use `any` type anywhere in this story
- The `authorId` column must NOT be made nullable in Prisma — no migration file for this story (ADR-006)

---

## Completion Notes

**Files created:** none

**Files modified:**
- `backend/src/narative-membership-request/narative-membership-request.service.ts`
- `backend/src/events/narative-membership-request/created.event.ts`
- `backend/src/notification/listeners/narative-membership-request.listener.ts`

**Deviations:**
- `narative-membership-request.resolver.ts` — no change required; resolver was already passing `currentUserId` to `decline()`
- `decline()` now fetches the narative (mirrors the exact pattern from `accept()`) to perform the member list check — this is required by the implementation, not a deviation from the spec

**Escalations:** none

---

## Dev Notes

- The existing `@CurrentUser()` decorator is used across the resolver layer for auth context — see `backend/src/auth/decorators/` for the pattern
- NestJS `@UseGuards(GqlAuthGuard)` is already on the resolver class — do NOT add it again
- `ForbiddenException` is from `@nestjs/common` — import it the same way it is used elsewhere in `narative-membership-request.service.ts`
- The notification listener pattern (iterating push tokens) follows the existing Expo push SDK integration — mirror the pattern already used in the listener file
- Run `npm run test -- --testPathPattern=narative-membership-request.service` from `/backend` to run targeted tests
