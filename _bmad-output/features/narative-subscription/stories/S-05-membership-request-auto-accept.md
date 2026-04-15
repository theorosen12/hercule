# S-05 — Membership Request Accept Fix (Any Member)

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-03

---

## Goal

Modify `NarativeMembershipRequestService.accept()` to allow any narative member to accept a membership request — replacing the current author-only restriction.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D10 | **No auto-accept for membership requests** — `sendNarativeMembershipRequest` unchanged. All requests require explicit member validation. Auto-accept removed as too permissive for intimate groups. |
| D11 | **`acceptNarativeMembershipRequest` extended to allow any narative member** — not just the narative author. Current implementation restricts to `author` only — this must be changed. New check: `caller is in narative.memberList`. Resolver signature unchanged. |

### Interfaces & Signatures

```typescript
// backend/src/narative-membership-request/narative-membership-request.service.ts — MODIFY accept()

// Current implementation (to change):
// accept(id, authorId): checks authorId === request.authorId → only narative author can accept

// New implementation (D11):
// accept(id, callerId):
//   1. Load the membership request with narative.memberList
//   2. Check: callerId is in narative.memberList → else ForbiddenException
//   3. Accept: update status to 'accepted', add requester to narative memberList
//   Signature unchanged: accept(id: string, callerId: string)
```

### Error Handling

| Error Condition | Error Type | GQL Code | Message |
|-----------------|------------|----------|---------|
| Non-member calling `acceptNarativeMembershipRequest` | `ForbiddenException` | FORBIDDEN | "Only narative members can accept requests" |

---

## Files to Create

None.

## Files to Modify

| File Path | Change | Risk |
|-----------|--------|------|
| `backend/src/narative-membership-request/narative-membership-request.service.ts` | Modify `accept()`: replace author-only check with member-of-narative check (D11) | Medium — changes authorization logic |

---

## Tasks

- [x] Open `backend/src/narative-membership-request/narative-membership-request.service.ts` and locate the `accept(id: string, callerId: string)` method
- [x] Identify the current authorization check (`narativeMembershipRequest.author?.id !== authorId`) — replaced
- [x] Load narative via `narativeRepository.find(narativeId)` to get `memberList` (already injected)
- [x] Replace the author-only check with: `if (!narative.memberList?.some(m => m.id === callerId)) throw new ForbiddenException('Only narative members can accept requests')`
- [x] Leave all other `accept()` logic unchanged (status update to 'accepted', add requester to `narative.memberList`, emit events)
- [x] Verify that `acceptNarativeMembershipRequest` resolver passes `user.id` as `callerId` — resolver signature unchanged

---

## Acceptance Criteria

- [x] A narative member (non-author) can call `acceptNarativeMembershipRequest` and successfully accept — requester is added to `narative.memberList`
- [x] The narative author can still accept (they are in `memberList`)
- [x] A user who is NOT in `narative.memberList` calling `accept()` receives `ForbiddenException`
- [x] An author who was removed from `memberList` (edge case) calling `accept()` receives `ForbiddenException`
- [x] Resolver signature of `acceptNarativeMembershipRequest(id: ID!)` is unchanged
- [x] No auto-accept logic is introduced anywhere (D10)

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| Caller is narative author but NOT in `memberList` | `ForbiddenException` — membership in `memberList` is the authoritative check, not `authorId` | FORBIDDEN GQL error |
| Caller is member → accepted | Requester added to `narative.memberList`, status set to 'accepted' | Success |
| Non-member caller | `ForbiddenException('Only narative members can accept requests')` | FORBIDDEN GQL error |

---

## Architectural Constraints

- Do NOT add `autoAcceptIfEligible` parameter or any auto-accept logic (D10 — explicitly removed)
- The resolver signature `acceptNarativeMembershipRequest(id: ID!)` must NOT change
- The authorization check must verify membership via `narative.memberList` — NOT via `narative.authorId`
- Do NOT use `any` type anywhere

---

## Dev Notes

- The `accept()` method signature `accept(id: string, callerId: string)` stays identical — only the internal authorization logic changes
- The extra `memberList` load in `accept()` adds one query vs current implementation — acceptable (noted in architecture §9 Performance)
- If the membership request service uses `EventEmitter2` to emit events after acceptance, leave that logic intact

---

## Completion Notes

**Files created:** none
**Files modified:** `backend/src/narative-membership-request/narative-membership-request.service.ts`
**Deviations:** Author-only check was `narativeMembershipRequest.author?.id !== authorId` returning null (not throwing). Replaced with `ForbiddenException` as specified. Narative loaded via `narativeRepository.find()` (already injected) rather than modifying PRISMA_INCLUDE.
**Escalations:** none
