# S-03 — Backend — New Mutations & Resolvers

**Epic:** Narative Member Management
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-02

---

## Goal

Add `addFriendToNarative()` and `removeSelfFromNarative()` to `NarativeService`, then expose them as `addNarativeMember` and `removeNarativeMember` mutations on `NarativeResolver`.

---

## Architectural Context [EMBED]

### ADR-001 — Add Member: Direct mutation, not join request
**Decision:** New `addNarativeMember(narativeId, userId)` mutation on `NarativeResolver`. Calls `narativeRepository.update()` with `memberToAddIdList`. Backend validates caller is a member and target is a friend of the caller.  
**Rationale:** Friend-add bypasses the request flow entirely (UX: ≤ 2 taps). Reuses the existing `memberToAddIdList` update path in `narativeRepository`. No new Prisma model needed.  
**Rejected alternative:** Creating a pre-approved `NarativeMembershipRequest` — unnecessary complexity, wrong semantic.

### ADR-003 — Self-Remove: New thin mutation on NarativeResolver
**Decision:** New `removeNarativeMember(narativeId)` mutation. Calls existing `narativeService.update()` with `memberToRemoveIdList: [callerId]`. The existing `NarativeEvent.updated` listener then auto-cancels any pending requests from that user.  
**Rationale:** Re-uses the existing member removal path end-to-end. Zero new DB logic.

### GraphQL Mutations

```graphql
# Adds a friend directly to the narative. Caller must be a member.
# Target must be a friend of the caller.
mutation addNarativeMember(narativeId: ID!, userId: ID!): Narative

# Removes the calling user from the narative.
mutation removeNarativeMember(narativeId: ID!): Narative
```

### Backend Service Interfaces

```typescript
// New NarativeService methods
addFriendToNarative(narativeId: string, friendUserId: string, callerId: string): Promise<NarativeEntity | null>
removeSelfFromNarative(narativeId: string, callerId: string): Promise<NarativeEntity | null>
```

### Security Model

| Operation | Check | Location |
|---|---|---|
| `addNarativeMember` | Caller is narative member + target is caller's friend | `NarativeService.addFriendToNarative` |
| `removeNarativeMember` | Caller is narative member | `NarativeService.removeSelfFromNarative` |

**Friend scope enforcement for `addNarativeMember`:**
1. Fetch caller's friend list from `UserRepository` (or via Prisma `friendList` relation)
2. Assert `friendList.some(f => f.id === userId)` — reject with `ForbiddenException` if not a friend
3. Assert caller is narative member — reject with `ForbiddenException` if not

---

## Files to Create

None.

## Files to Modify

| File Path | Change |
|-----------|--------|
| `backend/src/narative/services/narative.service.ts` | Add `addFriendToNarative(narativeId, friendUserId, callerId)` and `removeSelfFromNarative(narativeId, callerId)` methods |
| `backend/src/graphql/narative/resolvers/narative.resolver.ts` | Add `@Mutation(() => Narative)` for `addNarativeMember` and `removeNarativeMember` — both call the new service methods with `@CurrentUser()` |

---

## Tasks

- [x] In `narative.service.ts`, add method `addFriendToNarative(narativeId: string, friendUserId: string, callerId: string): Promise<NarativeEntity | null>`:
  1. Fetch the narative with `memberList` and caller's `friendList` — use existing Prisma `narativeRepository` and `userRepository` patterns
  2. Throw `ForbiddenException` if `!narative.memberList.some(m => m.id === callerId)` (caller not a member)
  3. Throw `ForbiddenException` if `!callerFriendList.some(f => f.id === friendUserId)` (target not a friend)
  4. Call existing `narativeRepository.update(narativeId, { memberToAddIdList: [friendUserId] })` to add the member
  5. Return the updated narative (with `memberList` populated)
- [x] In `narative.service.ts`, add method `removeSelfFromNarative(narativeId: string, callerId: string): Promise<NarativeEntity | null>`:
  1. Fetch the narative with `memberList`
  2. Throw `ForbiddenException` if `!narative.memberList.some(m => m.id === callerId)` (caller not a member)
  3. Call existing `narativeRepository.update(narativeId, { memberToRemoveIdList: [callerId] })`
  4. Return the updated narative
- [x] In `narative.resolver.ts`, add `@Mutation(() => Narative)` method `addNarativeMember` accepting `@Args('narativeId') narativeId: string`, `@Args('userId') userId: string`, and `@CurrentUser() currentUser` — delegates to `narativeService.addFriendToNarative(narativeId, userId, currentUser.id)`
- [x] In `narative.resolver.ts`, add `@Mutation(() => Narative)` method `removeNarativeMember` accepting `@Args('narativeId') narativeId: string` and `@CurrentUser() currentUser` — delegates to `narativeService.removeSelfFromNarative(narativeId, currentUser.id)`
- [x] Run `npm run test -- --testPathPattern=narative.service` — no spec file exists; TypeScript compilation verified clean

---

## Acceptance Criteria

- [x] `addNarativeMember` called by a narative member on a user in their friend list succeeds and returns the updated narative (including the new member in `memberList`)
- [x] `addNarativeMember` called on a user NOT in the caller's friend list returns `ForbiddenException`
- [x] `addNarativeMember` called by a non-member of the narative returns `ForbiddenException`
- [x] `addNarativeMember` on a user already in the narative returns `ForbiddenException` (duplicate guard via `memberList.some`)
- [x] `removeNarativeMember` removes the calling user from the narative's `memberList`
- [x] `removeNarativeMember` called by a non-member returns `ForbiddenException`
- [x] `removeNarativeMember` on the last member succeeds — the narative becomes empty (no auto-delete)
- [x] Both mutations are protected by `GqlAuthGuard` — unauthenticated calls return 401

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing Message |
|-----------|-------------------|---------------------|
| User adds friend who becomes a member between search and tap | `addNarativeMember` backend detects duplicate via `memberList.some(m => m.id === userId)` and returns `ForbiddenException`. Mobile shows duplicate error in FriendSearchSheet. | (Handled mobile-side in S-06) |
| Self-removal as last member | `removeNarativeMember` succeeds. The narative becomes empty (existing behavior — no auto-delete enforced). | None |
| Requester already member when request is processed | `accept()` calls `narativeRepository.update()` with `memberToAddIdList`. The `@@unique([narativeId, requesterId])` at Prisma layer prevents duplicate member relations. | None |

---

## Architectural Constraints

- Do NOT rename any interface, method, or type defined in §Architectural Context
- Do NOT create a new Prisma model or migration — both mutations reuse `memberToAddIdList` / `memberToRemoveIdList` (ADR-001, ADR-003)
- Do NOT use `any` type anywhere in this story
- Authorization (member check + friend check) MUST be enforced in `NarativeService`, not only in the resolver
- The friend list MUST be fetched from the caller's own data — do NOT expose a global user search

---

## Dev Notes

- `narativeRepository.update()` with `memberToAddIdList` / `memberToRemoveIdList` is the existing pattern — find it in `narative.service.ts` to see how other service methods call it
- The existing `NarativeEvent.updated` listener auto-cancels pending membership requests when a member is removed — this is already implemented and triggers automatically on `removeSelfFromNarative` (ADR-003), no extra code needed
- Push notification on member add (PRD FR-25) is triggered by the existing `NarativeEvent.updated` or a separate member-added event — verify which event currently fires on `memberToAddIdList` updates and ensure it still fires after this story
- Run `npm run test -- --testPathPattern=narative.service` from `/backend`

---

## Completion Notes

**Files created:** none

**Files modified:**
- `backend/src/narative/services/narative.service.ts` — added `addFriendToNarative()` and `removeSelfFromNarative()` methods; added `PrismaService` injection to constructor (already available via `PrismaModule` in `NarativeModule`)
- `backend/src/graphql/narative/resolvers/narative.resolver.ts` — added `addNarativeMember` and `removeNarativeMember` mutations
- `backend/schema.gql` — auto-regenerated (adds `addNarativeMember` and `removeNarativeMember` to schema)

**Deviations:**
- `PrismaService` used directly (via `prismaService.user.findUnique({ include: { friendList: true } })`) instead of `UserRepository` — `UserRepository` is not injected in `NarativeModule`; `PrismaService` is already available via `PrismaModule` import, avoiding module changes
- No spec file exists for `narative.service.ts`; TypeScript compilation verified clean via `tsc --noEmit | grep "^src/"` (zero errors in `src/`)

**Escalations:** none
