# S-06 — NarativeService + NarativeResolver Access Integration

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-04, S-05

---

## Goal

Extend `NarativeService.update()` to revoke all subscriptions atomically when confidentiality changes from public to private, and integrate `NarativeAccessService` into `narative.resolver.ts` so subscribers on public naratives gain read access.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D4 | **`NarativeAccessService.resolveAccess()` centralized** — all access checks go through it. Single source of truth for "can this user see this narative". Inline checks in each resolver rejected (duplicates logic, drift risk). |
| D5 | **Subscription access only on public naratives** — `isSubscriber` grants access only when `confidentiality === 'public' && subscriptionsEnabled === true`. A user in `likedBy` of a private narative has no access. |
| D8 | **Public → private visibility change triggers `clearAllSubscribers()`** in `NarativeService.update()`. Must be atomic with the visibility change — same service call. Separate endpoint for revocation rejected (race condition risk). |

### Interfaces & Signatures

```typescript
// backend/src/narative/services/narative.service.ts — MODIFY update()
// Add: when update input contains confidentiality === 'private'
//   AND existing narative has confidentiality === 'public'
//   → call narativeRepository.clearAllSubscribers(id) BEFORE persisting the update (D8)
```

```typescript
// backend/src/graphql/narative/resolvers/narative.resolver.ts — MODIFY narative query
// Extend access guard: replace or extend current check with:
//   narativeAccessService.assertCanAccess(user.id, narative)
// This allows: isMember || (isSubscriber && confidentiality === 'public' && subscriptionsEnabled)
// narative must be loaded with PRISMA_INCLUDE.Narative (already includes likedBy, memberList)
```

---

## Files to Create

None.

## Files to Modify

| File Path | Change | Risk |
|-----------|--------|------|
| `backend/src/narative/services/narative.service.ts` | Modify `update()`: call `clearAllSubscribers(narativeId)` when `confidentiality` changes `'public'` → `'private'` | Medium — touches narative update path |
| `backend/src/graphql/narative/resolvers/narative.resolver.ts` | Integrate `NarativeAccessService.assertCanAccess()` to allow subscriber access on public naratives | Medium — touches access guard |

---

## Tasks

**NarativeService.update() — subscriber revocation (D8):**

- [x] Open `backend/src/narative/services/narative.service.ts` and locate the `update()` method
- [x] `currentNarative` already loaded at top of update() via `narativeRepository.find(id)`
- [x] Add condition: if `currentNarative.confidentiality === 'public'` AND `updateInput.confidentiality === 'private'`, call `await narativeRepository.clearAllSubscribers(id)` BEFORE persisting the update
- [x] The `clearAllSubscribers` call precedes the persistence call (D8, atomic order)
- [x] `NarativeRepository` already injected in `NarativeService`

**NarativeResolver — subscriber access (D4, D5):**

- [x] Open `backend/src/graphql/narative/resolvers/narative.resolver.ts` and locate the `narative` query resolver
- [x] Identified existing check: `if (narative.confidentiality === 'private') { isMember check → return null }`
- [x] Injected `NarativeAccessService` into `NarativeResolver` constructor
- [x] Replaced existing check with `narativeAccessService.assertCanAccess(userId, narative)` — uses exact logic from S-03
- [x] Narative loaded with `PRISMA_INCLUDE.Narative` (includes `likedBy` and `memberList`) — no extra queries

---

## Acceptance Criteria

- [x] `updateNarative(confidentiality: 'private')` on a previously public narative results in empty `likedBy` after the call
- [x] `updateNarative(confidentiality: 'private')` on an already-private narative does NOT call `clearAllSubscribers`
- [x] A subscriber (in `likedBy`) on a public narative with `subscriptionsEnabled = true` can successfully call the `narative` query
- [x] A subscriber on a public narative with `subscriptionsEnabled = false` receives GQL FORBIDDEN
- [x] A subscriber on a private narative receives GQL FORBIDDEN
- [x] A non-member, non-subscriber user still receives GQL FORBIDDEN from `narative` query
- [x] Existing member access to the `narative` query is unaffected (no regression)

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| Public → private with subscribers | `clearAllSubscribers()` called BEFORE `update()` persists `confidentiality: 'private'` (D8) | Immediate, silent revocation |
| Already-private narative updated to `confidentiality: 'private'` | No `clearAllSubscribers()` call | Transparent |
| Subscriber on public narative, `subscriptionsEnabled = false` | `assertCanAccess()` → `canAccess = false` → FORBIDDEN | Access denied |
| Authenticated user navigates directly to `narative` query (bypassing entry route) | `assertCanAccess()` denies at API layer | Access denied |

---

## Architectural Constraints

- `clearAllSubscribers()` MUST be called BEFORE persisting the confidentiality change — never after (D8)
- Access logic in `narative.resolver.ts` must use `NarativeAccessService.assertCanAccess()` — do NOT inline the subscriber check (D4)
- Narative loaded for access check must use `PRISMA_INCLUDE.Narative` — no extra query for `likedBy` (D9 Performance)
- Do NOT use `any` type anywhere

---

## Dev Notes

- Read the existing access guard in `narative.resolver.ts` carefully before modifying — the goal is to extend, not replace, member access
- `NarativeAccessService` is exported from `NarativeModule` (wired in S-03) — ensure the module containing `NarativeResolver` imports `NarativeModule` if it doesn't already

---

## Completion Notes

**Files created:** none
**Files modified:** `backend/src/narative/services/narative.service.ts`, `backend/src/graphql/narative/resolvers/narative.resolver.ts`
**Deviations:** none
**Escalations:** none
