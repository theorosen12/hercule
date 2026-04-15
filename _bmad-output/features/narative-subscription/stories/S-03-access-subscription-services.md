# S-03 — NarativeAccessService + NarativeSubscriptionService

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-02

---

## Goal

Create `NarativeAccessService` (centralized access resolution) and `NarativeSubscriptionService` (toggle + subscriber count), then register both in `NarativeModule`.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D4 | **`NarativeAccessService.resolveAccess()` centralized** — all access checks go through it. Single source of truth for "can this user see this narative". Prevents access-check drift across resolvers. Inline checks in each resolver rejected (duplicates logic, drift risk). |
| D5 | **Subscription access only on public naratives** — `isSubscriber` grants access only when `confidentiality === 'public' && subscriptionsEnabled === true`. A user in `likedBy` of a private narative has no access. |
| D15 | **`NarativeSubscriptionService` manages toggle and subscriber count** — separate from `NarativeService`. Clear separation of concerns. |

### Interfaces & Signatures

```typescript
// backend/src/narative/services/narative-access.service.ts — CREATE

export interface NarativeAccessResult {
  canAccess: boolean;
  isMember: boolean;
  isSubscriber: boolean;
}

@Injectable()
export class NarativeAccessService {
  // Resolves whether a user can access a narative entity already loaded.
  // Does NOT fetch the narative — caller must provide it.
  resolveAccess(
    userId: string,
    narative: NarativeEntity,
  ): NarativeAccessResult;
  // Logic:
  // isMember = narative.memberList.some(m => m.id === userId)
  // isSubscriber = narative.likedBy?.some(u => u.id === userId) ?? false
  // canAccess = isMember
  //   || (isSubscriber && narative.subscriptionsEnabled && narative.confidentiality === 'public')
  // return { canAccess, isMember, isSubscriber }

  // Throws ForbiddenException if canAccess is false.
  assertCanAccess(userId: string, narative: NarativeEntity): void;
}
```

```typescript
// backend/src/narative/services/narative-subscription.service.ts — CREATE

@Injectable()
export class NarativeSubscriptionService {
  constructor(
    private readonly narativeRepository: NarativeRepository,
  ) {}

  // Toggle subscriptionsEnabled for a narative.
  // Caller must be in narative.memberList — throws ForbiddenException otherwise.
  // If disabling (enabled = false): calls clearAllSubscribers() BEFORE updating flag.
  async toggleSubscriptions(
    narativeId: string,
    requesterId: string,
    enabled: boolean,
  ): Promise<NarativeEntity>;

  // Return count of current subscribers (likedBy.length).
  async getSubscriberCount(narativeId: string): Promise<number>;
}
```

---

## Files to Create

| File Path | Class Name | Responsibility |
|-----------|------------|----------------|
| `backend/src/narative/services/narative-access.service.ts` | `NarativeAccessService` | `resolveAccess(userId, narative): NarativeAccessResult` + `assertCanAccess(userId, narative): void` |
| `backend/src/narative/services/narative-subscription.service.ts` | `NarativeSubscriptionService` | `toggleSubscriptions(narativeId, requesterId, enabled)` + `getSubscriberCount(narativeId)` |

## Files to Modify

| File Path | Change | Risk |
|-----------|--------|------|
| `backend/src/narative/narative.module.ts` | Add `NarativeAccessService`, `NarativeSubscriptionService` to `providers` and `exports` | Low |

---

## Tasks

- [x] Create `backend/src/narative/services/narative-access.service.ts` with class `NarativeAccessService` decorated with `@Injectable()`
- [x] Implement `resolveAccess(userId: string, narative: NarativeEntity): NarativeAccessResult` with the exact logic specified in §Interfaces above (isMember check, isSubscriber check, canAccess = isMember || (isSubscriber && subscriptionsEnabled && confidentiality === 'public'))
- [x] Implement `assertCanAccess(userId: string, narative: NarativeEntity): void` — calls `resolveAccess()`, throws `ForbiddenException` if `canAccess === false`
- [x] Export `NarativeAccessResult` interface from the same file
- [x] Create `backend/src/narative/services/narative-subscription.service.ts` with class `NarativeSubscriptionService` decorated with `@Injectable()`, injecting `NarativeRepository` via constructor
- [x] Implement `toggleSubscriptions(narativeId: string, requesterId: string, enabled: boolean): Promise<NarativeEntity>`:
  - Load the narative via `narativeRepository.find(narativeId)` (with `memberList` and `likedBy` included via `PRISMA_INCLUDE.Narative`)
  - Check `requesterId` is in `narative.memberList` — throw `ForbiddenException('Only members can toggle subscriptions')` otherwise
  - If `enabled === false`: call `narativeRepository.clearAllSubscribers(narativeId)` BEFORE calling `narativeRepository.updateSubscriptionsEnabled(narativeId, false)`
  - If `enabled === true`: call `narativeRepository.updateSubscriptionsEnabled(narativeId, true)` directly (no subscriber clear)
  - Return the updated `NarativeEntity`
- [x] Implement `getSubscriberCount(narativeId: string): Promise<number>` — load the narative, return `narative.likedBy.length`
- [x] Add `NarativeAccessService` and `NarativeSubscriptionService` to the `providers` array in `backend/src/narative/narative.module.ts`
- [x] Add `NarativeAccessService` and `NarativeSubscriptionService` to the `exports` array in `backend/src/narative/narative.module.ts`

---

## Acceptance Criteria

- [x] `NarativeAccessService.resolveAccess()` returns `{ canAccess: true, isMember: true, isSubscriber: false }` for a member
- [x] `NarativeAccessService.resolveAccess()` returns `{ canAccess: true, isMember: false, isSubscriber: true }` for a subscriber on a public narative with `subscriptionsEnabled = true`
- [x] `NarativeAccessService.resolveAccess()` returns `{ canAccess: false, ... }` for a subscriber on a public narative with `subscriptionsEnabled = false`
- [x] `NarativeAccessService.resolveAccess()` returns `{ canAccess: false, ... }` for a subscriber on a private narative
- [x] `NarativeAccessService.assertCanAccess()` throws `ForbiddenException` when `canAccess = false`
- [x] `NarativeSubscriptionService.toggleSubscriptions()` with `enabled = false` calls `clearAllSubscribers()` BEFORE `updateSubscriptionsEnabled(false)`
- [x] `NarativeSubscriptionService.toggleSubscriptions()` called by non-member throws `ForbiddenException`
- [x] Both services are injectable via NestJS DI in other modules that import `NarativeModule`

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| Subscriber on private narative | `resolveAccess()`: `canAccess = false` (D5 — subscription only on public naratives) | Access denied via `assertCanAccess()` |
| Non-member calling `toggleSubscriptions` | `ForbiddenException('Only members can toggle subscriptions')` | FORBIDDEN GQL error |
| Subscriber revocation: toggle disabled | `clearAllSubscribers()` called BEFORE `updateSubscriptionsEnabled(false)` — atomic order ensures no access window | Immediate revocation |

---

## Architectural Constraints

- `NarativeAccessService.resolveAccess()` does NOT fetch the narative — it operates on an already-loaded `NarativeEntity` (caller responsibility)
- `canAccess` logic must be exactly: `isMember || (isSubscriber && narative.subscriptionsEnabled && narative.confidentiality === 'public')` — no deviation (D5)
- `clearAllSubscribers()` MUST be called BEFORE `updateSubscriptionsEnabled(false)` — never after (D3)
- Do NOT add auto-accept logic anywhere (D10)
- Do NOT use `any` type anywhere

---

## Dev Notes

- Unit tests for `NarativeAccessService.resolveAccess()` are specified in architecture §12 — implement all 5 cases
- Unit tests for `NarativeSubscriptionService.toggleSubscriptions()` are specified in architecture §12 — implement all 3 cases
- `NarativeEntity` type comes from the Prisma client generated in S-01 — it now includes `subscriptionsEnabled: boolean`
- Follow the existing `@Injectable()` service pattern in `backend/src/narative/services/narative.service.ts` for DI structure
- `confidentiality` on `NarativeEntity` is the `ConfidentialityType` enum (`'public'` | `'private'`)

---

## Completion Notes

**Files created:** `backend/src/narative/services/narative-access.service.ts`, `backend/src/narative/services/narative-subscription.service.ts`
**Files modified:** `backend/src/narative/narative.module.ts`
**Deviations:** none
**Escalations:** none
