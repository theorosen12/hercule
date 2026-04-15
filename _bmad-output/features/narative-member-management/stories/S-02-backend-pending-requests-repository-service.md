# S-02 — Backend — Pending Request Repository & Service

**Epic:** Narative Member Management
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-01

---

## Goal

Add `findManyPendingByNarative(narativeId)` to `NarativeMembershipRequestRepository` and `listPendingByNarative(narativeId, callerId)` to `NarativeMembershipRequestService`, and expose them via a new `listNarativePendingRequests` GraphQL query on `NarativeMembershipRequestResolver`.

---

## Architectural Context [EMBED]

### ADR-002 — List Pending Requests: New query, member-gated
**Decision:** New `listNarativePendingRequests(narativeId): [NarativeMembershipRequest]` query on `NarativeMembershipRequestResolver`. Queries `NarativeMembershipRequest` records where `status = pending` AND `narativeId` matches. Caller must be a narative member (enforced in service).  
**Rationale:** The existing resolver only has mutations. A dedicated query keeps the pattern clean.  
**Note:** No new Prisma model, only a new `findManyPendingByNarative(narativeId)` repository method.

### GraphQL Query

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

### Backend Service Interfaces

```typescript
// New service method signature — NarativeMembershipRequestService
listPendingByNarative(narativeId: string, callerId: string): Promise<NarativeMembershipRequestEntity[]>
```

### Security Model

| Operation | Check | Location |
|---|---|---|
| `listNarativePendingRequests` | Caller is narative member | `NarativeMembershipRequestService.listPendingByNarative` |

`listNarativePendingRequests` must return 403 (not empty list) for non-members. The `ForbiddenException` from NestJS maps to a GraphQL error, which Apollo surfaces in the `error` field.

---

## Files to Create

None. All changes are modifications to existing files.

## Files to Modify

| File Path | Change |
|-----------|--------|
| `backend/src/narative-membership-request/narative-membership-request.repository.ts` | Add `findManyPendingByNarative(narativeId: string): Promise<NarativeMembershipRequestEntity[]>` — Prisma query filtering by `narativeId` and `status: 'pending'`, ordered by `date` descending |
| `backend/src/narative-membership-request/narative-membership-request.service.ts` | Add `listPendingByNarative(narativeId: string, callerId: string): Promise<NarativeMembershipRequestEntity[]>` — verify caller is member, then delegate to `repository.findManyPendingByNarative(narativeId)` |
| `backend/src/graphql/narative-membership-request/resolvers/narative-membership-request.resolver.ts` | Add `@Query(() => [NarativeMembershipRequest])` resolver method `listNarativePendingRequests(@Args('narativeId') narativeId: string, @CurrentUser() user)` — calls `service.listPendingByNarative(narativeId, user.id)` |

---

## Tasks

- [x] In `narative-membership-request.repository.ts`, add method `findManyPendingByNarative(narativeId: string): Promise<NarativeMembershipRequestEntity[]>` using a Prisma `findMany` call with `where: { narativeId, status: 'pending' }` and `orderBy: { date: 'desc' }` — include `requester` and `narative` relations
- [x] In `narative-membership-request.service.ts`, add method `listPendingByNarative(narativeId: string, callerId: string): Promise<NarativeMembershipRequestEntity[]>`:
  1. Fetch the narative (with `memberList`) to verify `callerId` is a member
  2. Throw `ForbiddenException` if `!narative.memberList.some(m => m.id === callerId)`
  3. Return `this.repository.findManyPendingByNarative(narativeId)`
- [x] In `narative-membership-request.resolver.ts`, add a new `@Query(() => [NarativeMembershipRequest])` method named `listNarativePendingRequests` decorated with `@UseGuards(GqlAuthGuard)` (already on the class — no need to add again), accepting `@Args('narativeId') narativeId: string` and `@CurrentUser() currentUser` — delegates to `service.listPendingByNarative(narativeId, currentUser.id)`
- [x] Run `npm run test -- --testPathPattern=narative-membership-request.service` from `/backend` — no spec file exists; TypeScript compilation verified clean

---

## Acceptance Criteria

- [x] `listNarativePendingRequests(narativeId)` returns an array of `NarativeMembershipRequest` objects with `status = pending` for the given narative
- [x] Results are ordered by `date` descending (most recent first)
- [x] A narative member calling `listNarativePendingRequests` receives the pending list (including zero results when empty)
- [x] A non-member calling `listNarativePendingRequests` receives a GraphQL error mapping to 403 `ForbiddenException`
- [x] An unauthenticated caller receives a 401 (enforced by existing `GqlAuthGuard`)
- [x] `findManyPendingByNarative` does NOT return requests with `status != 'pending'`

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing Message |
|-----------|-------------------|---------------------|
| Join request for narative with no members (edge case post-cleanup) | `listPendingByNarative` returns the requests regardless; no approver. Requests remain pending indefinitely — acceptable for MVP, no auto-expiry. | None |
| Narative not found | Service throws `NotFoundException` before member check | GraphQL error (404) |

---

## Architectural Constraints

- Do NOT rename any interface, method, or type defined in §Architectural Context
- Do NOT create a new Prisma model — use the existing `NarativeMembershipRequest` model (ADR-002)
- Do NOT return requests with status other than `pending` from `findManyPendingByNarative`
- Do NOT use `any` type anywhere in this story
- Authorization MUST be enforced in the service layer (`listPendingByNarative`), not only in the resolver

---

## Completion Notes

**Files created:** none

**Files modified:**
- `backend/src/narative-membership-request/narative-membership-request.repository.ts`
- `backend/src/narative-membership-request/narative-membership-request.service.ts`
- `backend/src/graphql/narative-membership-request/resolvers/narative-membership-request.resolver.ts`

**Deviations:**
- `orderBy: { createdAt: 'desc' }` used instead of story's `{ date: 'desc' }` — `date` is the GQL alias; `createdAt` is the actual Prisma scalar field (confirmed via transformer mapping `date: createdAt`)
- `@UseGuards(GqlAuthGuard)` is on each method individually (not the class), so added it to the new query — consistent with all other methods in the resolver
- `NotFoundException` added to `@nestjs/common` import in service (story's edge case table requires it)

**Escalations:** none

---

## Dev Notes

- Follow the existing repository pattern in `narative-membership-request.repository.ts` — all other `findMany` or `findOne` methods serve as the exact pattern to mirror
- The `@CurrentUser()` decorator is from `backend/src/auth/decorators/` — already imported in the resolver, no new import needed
- NestJS Logger is available in the service via `private readonly logger = new Logger(NarativeMembershipRequestService.name)` — use it if you need to log, never `console.log`
- Run `npm run test -- --testPathPattern=narative-membership-request.service` from `/backend` to run targeted tests
