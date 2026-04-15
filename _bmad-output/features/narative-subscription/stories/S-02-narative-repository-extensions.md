# S-02 — NarativeRepository Extensions

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-01

---

## Goal

Add three new methods (`clearAllSubscribers`, `updateSubscriptionsEnabled`, `unlikeFromMemberNaratives`) to `NarativeRepository` and extend the existing `update()` method to disconnect `likedBy` when promoting a subscriber to member.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D1 | **Subscribe = Like** — `likedBy` is the subscriber list, no new table. `like()` / `unlike()` are the subscribe/unsubscribe operations. `likedBy` already in `PRISMA_INCLUDE.Narative`. |
| D3 | **Revocation = `clearAllSubscribers()` — immediate, no grace period.** Called on toggle disable OR public→private change. |
| D12 | **`NarativeRepository.update()` extended** — disconnects `likedBy` when `memberToAddIdList` non-empty. Subscriber promoted to member → auto-removed from `likedBy`. Belongs in repository layer — atomic with member add. |

### Interfaces & Signatures

```typescript
// backend/src/narative/narative.repository.ts — ADD these methods

// Remove all subscribers from a narative (called on toggle disable OR public→private change)
async clearAllSubscribers(narativeId: string): Promise<void>;
// Implementation:
// await this.prismaService.narative.update({
//   where: { id: narativeId },
//   data: { likedBy: { set: [] } },
// });

// Toggle subscriptionsEnabled flag
async updateSubscriptionsEnabled(
  narativeId: string,
  enabled: boolean,
): Promise<NarativeEntity>;
// Implementation:
// return this.prismaService.narative.update({
//   where: { id: narativeId },
//   data: { subscriptionsEnabled: enabled },
//   include: PRISMA_INCLUDE.Narative,
// });

// Remove a specific user from likedBy across all naratives where blockingUserId is a member
// Called by existing block handler to revoke subscriptions on block
async unlikeFromMemberNaratives(
  blockingUserId: string,
  blockedUserId: string,
): Promise<void>;

// MODIFY existing update() — disconnect likedBy when memberToAddIdList non-empty (D12 / FR7)
// In the existing update() memberList connect block, also add:
// likedBy: memberToAddIdList?.length > 0
//   ? { disconnect: memberToAddIdList.map((id) => ({ id })) }
//   : undefined,
```

---

## Files to Create

None.

## Files to Modify

| File Path | Change | Risk |
|-----------|--------|------|
| `backend/src/narative/narative.repository.ts` | Add `clearAllSubscribers()`, `updateSubscriptionsEnabled()`, `unlikeFromMemberNaratives()`; modify `update()` to disconnect `likedBy` for `memberToAddIdList` | Low — additive + one-line `update()` change |

---

## Tasks

- [x] Add `async clearAllSubscribers(narativeId: string): Promise<void>` to `NarativeRepository` using the implementation shown in §Interfaces above (`likedBy: { set: [] }`)
- [x] Add `async updateSubscriptionsEnabled(narativeId: string, enabled: boolean): Promise<NarativeEntity>` to `NarativeRepository` using the implementation shown in §Interfaces above (update + `include: PRISMA_INCLUDE.Narative`)
- [x] Add `async unlikeFromMemberNaratives(blockingUserId: string, blockedUserId: string): Promise<void>` to `NarativeRepository` — implementation: find all naratives where `blockingUserId` is in `memberList`, then disconnect `blockedUserId` from `likedBy` of each
- [x] Locate the existing `update()` method in `NarativeRepository` — find the `memberList: { connect: ... }` block inside the Prisma `update()` call
- [x] Add the `likedBy` disconnect clause to the existing `update()` call exactly as specified in §Interfaces above: `likedBy: memberToAddIdList?.length > 0 ? { disconnect: memberToAddIdList.map((id) => ({ id })) } : undefined`

---

## Acceptance Criteria

- [x] `clearAllSubscribers(narativeId)` sets `likedBy` to empty array for the target narative (verified by querying `likedBy` after call)
- [x] `updateSubscriptionsEnabled(narativeId, false)` sets `subscriptionsEnabled = false` and returns the updated `NarativeEntity` with full `PRISMA_INCLUDE.Narative` shape
- [x] `unlikeFromMemberNaratives(blockingUserId, blockedUserId)` removes `blockedUserId` from `likedBy` of all naratives where `blockingUserId` is a member
- [x] `update()` with `memberToAddIdList = ['userId']` disconnects that user from `likedBy` atomically in the same Prisma call
- [x] `update()` with `memberToAddIdList = []` or undefined does NOT touch `likedBy`

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| `clearAllSubscribers` called when `likedBy` is already empty | `likedBy: { set: [] }` is a no-op in Prisma — no error | Transparent |
| Subscriber promoted to member (`update()` with `memberToAddIdList`) | `likedBy` disconnect executed atomically with `memberList` connect (D12) | Transparent — user gains full member access |

---

## Architectural Constraints

- Do NOT create a new `NarativeSubscription` model (D1 — Subscribe = Like, no new table)
- Do NOT add grace period logic to `clearAllSubscribers` (D3 — immediate, no grace period)
- The `likedBy` disconnect in `update()` must be in the same Prisma `update()` call as `memberList` connect — no separate call (D12 — atomic)
- Do NOT use `any` type anywhere

---

## Dev Notes

- `PRISMA_INCLUDE.Narative` is imported from `backend/src/prisma/prisma.include.ts` — use it in `updateSubscriptionsEnabled()` return call for consistency with all other repository methods
- The existing `like()` and `unlike()` methods already exist and are NOT modified in this story — they are the subscribe/unsubscribe primitives reused by the entry screen
- Follow the existing repository method pattern in `narative.repository.ts` for `async` method structure and `this.prismaService.narative.update()` call style

---

## Completion Notes

**Files created:** none
**Files modified:** `backend/src/narative/narative.repository.ts`
**Deviations:** none
**Escalations:** none
