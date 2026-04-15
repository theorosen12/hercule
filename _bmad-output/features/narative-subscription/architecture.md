---
stepsCompleted: [architecture-written, architecture-approved]
inputDocuments:
  - _bmad-output/features/narative-subscription/prd.md
  - _bmad-output/features/narative-subscription/ux-spec.md
  - _bmad-output/features/narative-subscription/context.md
generatedAt: '2026-04-10'
epicIssue: NAR-388
---

# Architecture — Narative Subscription

_Generated: 2026-04-10 | Epic: NAR-388 | Status: Draft_

---

## 0. Codebase Baseline

### Existing — do not recreate, patterns to follow

| What | Where | Notes |
|------|-------|-------|
| `Narative.likedBy` / `User.likedNaratives` relation | `prisma/schema.prisma` | Subscribe = Like mechanic foundation — already in `PRISMA_INCLUDE.Narative` |
| `ConfidentialityType` enum (`public` / `private`) | `prisma/schema.prisma` | Already on `Narative` model |
| `NarativeRepository.like()` / `unlike()` | `narative.repository.ts` | Subscription add/remove primitives — reused as-is |
| `NarativeRepository.update()` | `narative.repository.ts` | Existing member add/remove — must be extended to disconnect `likedBy` on member add (FR7) |
| `NarativeService.update()` | `narative.service.ts` | Must be extended to call `clearAllSubscribers()` when `confidentiality` changes to `private` |
| `NarativeMembershipRequestService` (create, cancel, cancelMany, accept, decline) | `narative-membership-request.service.ts` | `accept()` currently author-only — must be extended to allow any narative member |
| `NarativeMembershipRequestRepository` | `narative-membership-request.repository.ts` | No changes needed |
| `NarativeMembershipRequestResolver` | `narative-membership-request.resolver.ts` | `acceptNarativeMembershipRequest` resolver — no param change, only service logic change |
| `likeNarative` / `unlikeNarative` mutations | `narative.resolver.ts` | Already exist — reused by entry screen subscribe action |
| `sendNarativeMembershipRequest` mutation | `narative-membership-request.resolver.ts` | Already exists — reused as-is. No `autoAcceptIfEligible` param. |
| `useLikeNarativeMutation` / `useUnlikeNarativeMutation` | `mobile/…/shared/infra/mutations/` | Already exist — reused by entry screen container |
| `useSendNarativeMembershipRequestMutation` | `mobile/…/page/infra/mutations/` | Already exists — reused as-is from page domain |
| `AskMembership` component | `mobile/…/page/view/components/AskMembership/` | Existing pattern for membership request UI — entry screen creates its own version |
| `LikeButton` container | `mobile/…/shared/view/containers/LikeButton/` | Existing like/subscribe pattern |
| `GqlAuthGuard` | `backend/src/jwt-app/graphql-jwt.guard.ts` | `@UseGuards(GqlAuthGuard)` on all new resolvers |
| `PRISMA_INCLUDE.Narative` | `backend/src/prisma/prisma.include.ts` | Already includes `likedBy`, `memberList`, `thumbnail` — no change needed |
| `EventEmitter2` pattern | `backend/src/events/` | `NarativeEvent` enum + `NarativeCreatedEvent` pattern |
| `NarativeTransformer` | `backend/src/graphql/narative/narative.transformer.ts` | Transformer pattern for GraphQL entity mapping |

### Nothing exists yet for subscription — green-field

Backend:
- No `narative-access.service.ts`
- No `narative-subscription.service.ts`
- No `narative-entry-screen.entity.ts`
- No `narative-subscription.resolver.ts`
- No migration for `subscriptionsEnabled`

Mobile:
- No `mobile/src/domains/narative/subscription/` domain
- No `entry.tsx` route under `narative/[id]/`

---

## 1. Architectural Decision Log

| # | Decision | Rationale | Alternatives Rejected |
|---|----------|-----------|----------------------|
| D1 | **Subscribe = Like** — `likedBy` is the subscriber list, no new table | Reuses existing Prisma relation. `like()` / `unlike()` are the subscribe/unsubscribe operations. `likedBy` already in `PRISMA_INCLUDE.Narative`. Zero schema overhead. | New `NarativeSubscription` model (unnecessary complexity, migration risk — cost vs benefit not justified for v0) |
| D2 | **`subscriptionsEnabled Boolean @default(true)`** on `Narative` | Toggle state stored server-side on the entity it governs. Default `true`: all new naratives are subscription-enabled without extra migration | Separate `SubscriptionSettings` table (over-engineering) |
| D3 | **Revocation = `clearAllSubscribers()` — immediate, no grace period** | PRD FR6/FR12: disable toggle OR public→private change → all subscribers lose access immediately. Simplest consistent contract. | Grace period / "pending revocation" state (PRD explicitly rejects) |
| D4 | **`NarativeAccessService.resolveAccess()` centralized** — all access checks go through it | Single source of truth for "can this user see this narative". Prevents access-check drift across resolvers. | Inline checks in each resolver (duplicates logic, drift risk) |
| D5 | **Subscription access only on public naratives** — `isSubscriber` grants access only when `confidentiality === 'public' && subscriptionsEnabled === true` | PRD FR13: subscriptions are a public-narative feature. Private naratives remain membership-only. A user in `likedBy` of a private narative has no access. | Subscription access on private naratives (contradicts privacy intent) |
| D6 | **`NarativeEntryScreen` renders "S'abonner" CTA only when `confidentiality === 'public'`** — on private naratives, only "Demander à devenir membre" | PRD FR19/FR20: subscribe is unavailable on private naratives. `confidentiality` field returned by `narativeEntryScreen` query drives this conditional. | Always showing subscribe CTA (contradicts scope decision) |
| D7 | **`SubscriptionToggleRow` not rendered when `narativeVisibility === 'private'`** — component returns null | PRD FR25: the toggle is meaningless on private naratives. Absence is cleaner than a disabled state. | Showing toggle as disabled on private (confusing to member) |
| D8 | **Public → private visibility change triggers `clearAllSubscribers()`** in `NarativeService.update()` | PRD FR12: when confidentiality changes to private, all subscribers lose access immediately and silently. Must be atomic with the visibility change — same service call. | Separate endpoint for revocation (race condition risk) |
| D9 | **`PrivacyChangeWarningBanner` — inline banner, no modal** in `NarativeSetup` screen | UX spec D19: warning is informational, not a decision point. Banner shown below visibility dropdown when `subscriberCount > 0` + transition to private. Confirmation implicit via "Enregistrer". | ConfirmationModal (creates a decision gate where PRD says there is none) |
| D10 | **No auto-accept for membership requests** — `sendNarativeMembershipRequest` unchanged. All requests require explicit member validation. | Brainstorming decision: auto-accept is too permissive for intimate groups. Removed entirely. | Auto-accept based on friendship graph or connected state (removed as over-permissive) |
| D11 | **`acceptNarativeMembershipRequest` extended to allow any narative member** — not just the narative author | Brainstorming decision: any member can validate from the notification page. Current implementation restricts to `author` only — this must be changed. New check: `caller is in narative.memberList`. | Author-only restriction (blocks the feature — members can't validate) |
| D12 | **`NarativeRepository.update()` extended** — disconnects `likedBy` when `memberToAddIdList` non-empty | PRD FR7: subscriber promoted to member → auto-removed from `likedBy`. Statuses are exclusive. Belongs in repository layer — atomic with member add. | Event listener (non-atomic, race condition risk) |
| D13 | **No paste link / deep link flow** — out of scope | Brainstorming: dedicated project created ([Deep Link — Paste a Nara Link](https://linear.app/naraa/project/deep-link-paste-a-nara-link-9ee92a6ce950)). | Building paste flow in this epic (scope creep) |
| D14 | **`narativeEntryScreen` is a new query** — not reusing `narative` | Entry screen exposes computed fields (`isAlreadySubscribed`, `isMember`, `momentCount`, `confidentiality`) that are entry-screen-specific. Avoids polluting the existing `narative` type. | Extending `narative` query (would expose entry fields to all callers) |
| D15 | **`NarativeSubscriptionService` manages toggle and subscriber count** — separate from `NarativeService` | Clear separation of concerns. `NarativeService` handles narative CRUD. Subscription operations (toggle, clear) are subscription-domain responsibility. | Adding methods to `NarativeService` (grows the service unboundedly) |
| D16 | **`/(app)/narative/[id]/entry` is the universal entry point** — all external navigation (notifications, share links, deep links) always targets `/entry`, never directly `/(app)/narative/[id]`. The `NarativeEntryContainer` is the access gate: it queries `narativeEntryScreen`, renders nothing while loading, then either redirects (member / active subscriber) or displays the entry screen. This prevents flash-of-wrong-content and centralizes the access decision. | Single decision point — no caller needs to know the user's relationship to the narative before navigating. The server response drives the routing decision, not client-side assumptions. | Caller decides which route based on cached data (stale, error-prone, duplicates server logic); pre-flight query before navigation (adds round-trip latency before any navigation occurs) |

---

## 2. Data Model (Prisma)

### 2.1 New Models

None. No new Prisma model required.

### 2.2 Modifications to Existing Models

**`Narative` model** — add `subscriptionsEnabled`:

```prisma
model Narative {
  // ... all existing fields unchanged ...
  subscriptionsEnabled Boolean @default(true)   // NEW — added for subscription toggle

  // existing relations — unchanged
  likedBy              User[]  @relation("LikedNaratives")
  memberList           User[]  @relation("NarativeToUser")
  // ... rest unchanged
}
```

### 2.3 Migration Plan

- **Migration name:** `{timestamp}_add_subscriptions_enabled_to_narative`
- **Additive-only:** Yes — `Boolean @default(true)` backfills all existing naratives with `true` automatically
- **Backfill required:** No — Prisma default handles it in migration SQL
- **Breaking change to existing API:** No

---

## 3. GraphQL API (Backend)

### 3.1 New Types / DTOs

```graphql
type NarativeEntryScreen {
  id: ID!
  title: String!
  location: String!
  startDate: DateTime!
  endDate: DateTime
  thumbnail: Thumbnail
  memberList: [User!]!
  momentCount: Int!
  subscriptionsEnabled: Boolean!
  isAlreadySubscribed: Boolean!
  isMember: Boolean!
  confidentiality: ConfidentialityType!
}
```

### 3.2 New Queries

| Query Name | Args | Returns | Auth | Description |
|------------|------|---------|------|-------------|
| `narativeEntryScreen` | `narativeId: ID!` | `NarativeEntryScreen` (nullable) | Yes — `GqlAuthGuard` | Landing data for the Narative Entry Screen. Returns null if narative not found. `isMember`, `isAlreadySubscribed`, `confidentiality` computed server-side from authenticated userId. |

### 3.3 New Mutations

| Mutation Name | Args | Returns | Auth | Description |
|---------------|------|---------|------|-------------|
| `toggleNarativeSubscriptions` | `narativeId: ID!, enabled: Boolean!` | `Narative!` | Yes — member only | Enable / disable subscriptions. Disabling triggers `clearAllSubscribers()`. |

**No modifications to existing mutations.** `sendNarativeMembershipRequest`, `likeNarative`, `unlikeNarative` are reused as-is.

**Modified behavior (no signature change):**

`acceptNarativeMembershipRequest(id: ID!)` — service logic changes: authorization check updated from "caller must be narative author" to "caller must be in `narative.memberList`" (D11). Resolver signature unchanged.

### 3.4 Error Handling

| Error Condition | Error Type | GQL Code | Message |
|-----------------|------------|----------|---------|
| Narative not found (`narativeEntryScreen`) | `null` returned | — | Silent null — container handles |
| Unauthenticated on any new resolver | `ForbiddenError` | UNAUTHENTICATED | Handled by `GqlAuthGuard` |
| Non-member calling `toggleNarativeSubscriptions` | `ForbiddenException` | FORBIDDEN | "Only members can toggle subscriptions" |
| Non-member calling `acceptNarativeMembershipRequest` | `ForbiddenException` | FORBIDDEN | "Only narative members can accept requests" |
| Subscriber (non-member) accessing private narative content | `ForbiddenException` via `NarativeAccessService` | FORBIDDEN | Access denied |
| User already member calling `sendNarativeMembershipRequest` | `null` returned | — | Logged server-side, silent to client |

---

## 4. TypeScript Interfaces & Service Contracts

### 4.1 Repository Additions — `NarativeRepository`

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

// Remove a specific user from likedBy across all naratives of a blocking user
// Called by existing block handler to revoke subscriptions
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

### 4.2 Service Interfaces

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

```typescript
// backend/src/narative/services/narative.service.ts — MODIFY update()
// Add: when update input contains confidentiality === 'private'
//   AND existing narative has confidentiality === 'public'
//   → call narativeRepository.clearAllSubscribers(id) BEFORE persisting the update (D8)
```

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

### 4.3 Mobile Hooks

```typescript
// mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/
//   getNarativeEntryScreen.query.hooks.ts — auto-generated by codegen

interface GetNarativeEntryScreenQuery {
  narativeEntryScreen: {
    id: string;
    title: string;
    location: string;
    startDate: string;
    endDate: string | null;
    momentCount: number;
    subscriptionsEnabled: boolean;
    isAlreadySubscribed: boolean;
    isMember: boolean;
    confidentiality: 'public' | 'private';  // D6: drives subscribe CTA visibility
    thumbnail: { id: string; default: { id: string; url: string } } | null;
    memberList: Array<{
      id: string;
      firstName: string;
      lastName: string;
      profilePicture: { id: string; url: string } | null;
    }>;
  } | null;
}

function useGetNarativeEntryScreenQuery(
  options: QueryHookOptions<GetNarativeEntryScreenQuery, { narativeId: string }>,
): QueryResult<GetNarativeEntryScreenQuery>;
// fetchPolicy: 'network-only' — set in container
```

```typescript
// mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/
//   toggleNarativeSubscriptions.mutation.hooks.ts — auto-generated by codegen

function useToggleNarativeSubscriptionsMutation(): [
  (options: MutationFunctionOptions<..., { narativeId: string; enabled: boolean }>) => Promise<...>,
  MutationResult<...>,
];
```

No subscription-domain copy of `sendNarativeMembershipRequest` — page domain mutation reused as-is (D10).

### 4.4 Component Props Interfaces

```typescript
// NarativeEntryScreen — full-screen layout, blurred thumbnail background
interface NarativeEntryScreenProps {
  title: string;
  location: string;
  startDate: Date;
  endDate: Date | null;
  memberList: Array<{
    id: string;
    firstName: string;
    lastName: string;
    profilePicture: { url: string } | null;
  }>;
  momentCount: number;
  thumbnailUrl: string | null;
  confidentiality: 'public' | 'private'; // D6: determines CTA layout
  subscriptionsEnabled: boolean;
  isAlreadySubscribed: boolean;
  hasPendingMembershipRequest: boolean;
  loading: boolean;
  hasError: boolean;
  onSubscribe: () => void;
  onUnsubscribe: () => void;
  onRequestMembership: () => void;
  onCancelMembershipRequest: () => void;
  onRetry: () => void;
}

// SubscriptionToggleRow — shown in narative settings, only when narativeVisibility === 'public' (D7)
interface SubscriptionToggleRowProps {
  narativeVisibility: 'public' | 'private'; // returns null when 'private'
  enabled: boolean;
  subscriberCount: number;
  loading: boolean;
  onToggle: (enabled: boolean) => void;
}

// PrivacyChangeWarningBanner — inline banner in NarativeSetup (D9)
interface PrivacyChangeWarningBannerProps {
  subscriberCount: number;  // injected in text: "Les {N} abonnements existants seront supprimés !"
  visible: boolean;         // controlled by parent NarativeSetup
}
```

```typescript
// Container props
interface NarativeEntryContainerProps {
  narativeId: string;
}

interface SubscriptionToggleRowContainerProps {
  narativeId: string;
  narativeVisibility: 'public' | 'private'; // passed from settings screen
}
```

---

## 5. Backend File Structure

### 5.1 Files to Create

| File Path | Class Name | Responsibility |
|-----------|------------|----------------|
| `backend/src/narative/services/narative-access.service.ts` | `NarativeAccessService` | Centralized access resolution: `resolveAccess(userId, narative)` + `assertCanAccess()` |
| `backend/src/narative/services/narative-subscription.service.ts` | `NarativeSubscriptionService` | `toggleSubscriptions(narativeId, requesterId, enabled)` + `getSubscriberCount(narativeId)` |
| `backend/src/graphql/narative/entities/narative-entry-screen.entity.ts` | `NarativeEntryScreen` (GQL `@ObjectType`) | GraphQL output type for `narativeEntryScreen` query — all fields from §3.1 |
| `backend/src/graphql/narative/resolvers/narative-subscription.resolver.ts` | `NarativeSubscriptionResolver` | `narativeEntryScreen` query + `toggleNarativeSubscriptions` mutation |

### 5.2 Files to Modify

| File Path | Change | Risk |
|-----------|--------|------|
| `backend/prisma/schema.prisma` | Add `subscriptionsEnabled Boolean @default(true)` to `Narative` model | Low — additive |
| `backend/src/narative/narative.repository.ts` | Add `clearAllSubscribers()`, `updateSubscriptionsEnabled()`, `unlikeFromMemberNaratives()`; modify `update()` to disconnect `likedBy` for `memberToAddIdList` (D12) | Low — additive + one-line update() change |
| `backend/src/narative/services/narative.service.ts` | Modify `update()`: when `confidentiality` changes to `'private'` → call `clearAllSubscribers()` before persisting (D8) | Medium — touches narative update path |
| `backend/src/narative/narative.module.ts` | Add `NarativeAccessService`, `NarativeSubscriptionService` to providers and exports | Low |
| `backend/src/graphql/narative/resolvers/narative.resolver.ts` | Extend `narative` query: use `NarativeAccessService.resolveAccess()` to allow subscriber access when `confidentiality === 'public' && subscriptionsEnabled` | Medium — touches access guard |
| `backend/src/narative-membership-request/narative-membership-request.service.ts` | Modify `accept()`: replace author-only check with member-of-narative check (D11) | Medium — changes authorization logic |

---

## 6. Mobile File Structure

### 6.1 Files to Create

**Infra — navigation service:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/infra/services/narativeEntryNavigation.service.ts` | Service | `navigateToNarativeEntry(router, narativeId)` — single function encapsulating the route + param construction. All navigation to entry route goes through here. |

**Infra — queries:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.gql` | GQL | `narativeEntryScreen` query — all fields from §3.1 |
| `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.hooks.ts` | Codegen | Auto-generated typed hook |

**Infra — mutations:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/toggleNarativeSubscriptions.mutation.gql` | GQL | `toggleNarativeSubscriptions(narativeId, enabled)` |
| `mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/toggleNarativeSubscriptions.mutation.hooks.ts` | Codegen | Auto-generated typed hook |

**View — screens:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/NarativeEntryScreen.tsx` | Screen (pure) | Full-screen layout: blurred thumbnail background, bottom sheet card with title / location / dates / members / momentCount / conditional CTAs |
| `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/index.ts` | Barrel | Re-exports |

**View — components:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/view/components/SubscriptionToggleRow/SubscriptionToggleRow.tsx` | Component | Row with toggle + subscriber count label. Returns null when `narativeVisibility === 'private'` (D7) |
| `mobile/src/domains/narative/subscription/view/components/SubscriptionToggleRow/index.ts` | Barrel | Re-exports |
| `mobile/src/domains/narative/subscription/view/components/PrivacyChangeWarningBanner/PrivacyChangeWarningBanner.tsx` | Component | Inline warning banner: "La confidentialité semble changer · Les {N} abonnements existants seront supprimés !" (D9) |
| `mobile/src/domains/narative/subscription/view/components/PrivacyChangeWarningBanner/index.ts` | Barrel | Re-exports |

**View — containers:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/NarativeEntry.container.tsx` | Container | Fetches `narativeEntryScreen` (network-only), handles subscribe / unsubscribe / membership-request / cancel-request, redirects if already member or subscriber |
| `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/index.ts` | Barrel | Re-exports |
| `mobile/src/domains/narative/subscription/view/containers/SubscriptionToggleRow/SubscriptionToggleRow.container.tsx` | Container | Fetches `subscriptionsEnabled` + `subscriberCount` for a narative; calls `toggleNarativeSubscriptions`; optimistic UI; passes `narativeVisibility` to component |
| `mobile/src/domains/narative/subscription/view/containers/SubscriptionToggleRow/index.ts` | Barrel | Re-exports |

**Routes:**

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/app/(app)/narative/[id]/entry.tsx` | Expo Router route | Reads `id` param via `useLocalSearchParams`; renders `NarativeEntryContainer`. No other logic. |

### 6.2 Files to Modify

| File Path | Change |
|-----------|--------|
| `mobile/src/app/(app)/narative/[id]/_layout.tsx` | Add `<Stack.Screen name="entry" options={{ headerShown: false }} />` — nothing else |
| `mobile/src/app/(app)/narative/[id]/edit/index.tsx` (or NarativeSetup screen — confirm exact path) | Add `<PrivacyChangeWarningBanner subscriberCount={subscriberCount} visible={showPrivacyWarning} />` below the visibility dropdown; add `<SubscriptionToggleRowContainer narativeId={id} narativeVisibility={currentVisibility} />` in member-only settings section |

> **Note on NarativeSetup integration:** The exact file hosting the narative edit form must be confirmed during story implementation (look for where `UpdateNarativeInput` / `confidentiality` dropdown is rendered). The components are created; their integration point is story-level investigation.

### 6.3 GraphQL Operations (exact)

```graphql
# getNarativeEntryScreen.query.gql
query getNarativeEntryScreen($narativeId: ID!) {
  narativeEntryScreen(narativeId: $narativeId) {
    id
    title
    location
    startDate
    endDate
    momentCount
    subscriptionsEnabled
    isAlreadySubscribed
    isMember
    confidentiality
    thumbnail {
      id
      default {
        id
        url
      }
    }
    memberList {
      id
      firstName
      lastName
      profilePicture {
        id
        url
      }
    }
  }
}
```

```graphql
# toggleNarativeSubscriptions.mutation.gql
mutation toggleNarativeSubscriptions($narativeId: ID!, $enabled: Boolean!) {
  toggleNarativeSubscriptions(narativeId: $narativeId, enabled: $enabled) {
    id
    subscriptionsEnabled
  }
}
```

Membership request mutations reused from page domain (no subscription-domain copies needed):
- `sendNarativeMembershipRequest` — `mobile/src/domains/narative/page/infra/mutations/sendNarativeMembershipRequest/`
- `cancelNarativeMembershipRequest` — `mobile/src/domains/narative/page/infra/mutations/cancelNarativeMembershipRequest/`

### 6.4 Navigation

| Route | Screen | Params | Entry Points |
|-------|--------|--------|--------------|
| `/(app)/narative/[id]/entry` | `NarativeEntryContainer` | `{ id: string }` | Deep link handler; auth redirect after unauthenticated user logs in; any notification with a narativeId |

```typescript
// narativeEntryNavigation.service.ts
export function navigateToNarativeEntry(
  router: Router,
  narativeId: string,
): void {
  router.push({
    pathname: '/(app)/narative/[id]/entry',
    params: { id: narativeId },
  });
}
// This is the ONLY function used to navigate toward a narative from outside.
// No caller inspects isMember / isSubscribed before calling this.
// The NarativeEntryContainer owns the access decision.
```

#### Navigation Decision Logic — `NarativeEntryContainer` (D16)

```
External trigger (notification / share link / deep link)
        │
        ▼
navigate to /(app)/narative/[id]/entry
        │
        ▼
NarativeEntryContainer mounts
  → fires narativeEntryScreen query (fetchPolicy: 'network-only')
  → renders <LoadingSpinner /> (no entry screen UI)
        │
        ▼
Query resolves
  ┌─────────────────────────────────────────────────────────┐
  │ isMember === true                                       │
  │   → router.replace('/(app)/narative/[id]')             │
  ├─────────────────────────────────────────────────────────┤
  │ isAlreadySubscribed === true                            │
  │   && subscriptionsEnabled === true                      │
  │   && confidentiality === 'public'                       │
  │   → router.replace('/(app)/narative/[id]')             │
  ├─────────────────────────────────────────────────────────┤
  │ narativeEntryScreen === null (not found)                │
  │   → render error state + retry                         │
  ├─────────────────────────────────────────────────────────┤
  │ else (non-member, non-subscriber, or access revoked)    │
  │   → render NarativeEntryScreen                         │
  └─────────────────────────────────────────────────────────┘
```

> **Key invariant:** The `NarativeEntryScreen` UI component is **never rendered during the loading phase** — only after the query confirms the user has no current access. This prevents any flash-of-wrong-content for members and active subscribers.

---

## 7. State Management

| State | Location | Type | Persistence |
|-------|----------|------|-------------|
| `optimisticSubscribed` | `NarativeEntryContainer` local | `boolean` | Ephemeral |
| `subscribeError` | `NarativeEntryContainer` local | `string \| undefined` | Ephemeral |
| `membershipRequested` | `NarativeEntryContainer` local | `boolean` | Ephemeral — set after `sendNarativeMembershipRequest` success |
| `membershipError` | `NarativeEntryContainer` local | `string \| undefined` | Ephemeral |
| `narativeEntryScreen` query data | Apollo cache | `NarativeEntryScreen` | In-memory; `fetchPolicy: 'network-only'` — never stale |
| `toggleEnabled` (optimistic) | `SubscriptionToggleRowContainer` local | `boolean` | Ephemeral — reverted on error |
| `subscriberCount` (optimistic) | `SubscriptionToggleRowContainer` local | `number` | Ephemeral — set to 0 on disable, restored on error |
| `showPrivacyWarning` | NarativeSetup/edit screen local | `boolean` | Ephemeral — `prevVisibility === 'public' && newVisibility === 'private' && subscriberCount > 0` |

---

## 8. Security

- **Authentication:** All new resolvers (`narativeEntryScreen`, `toggleNarativeSubscriptions`) use `@UseGuards(GqlAuthGuard)`. No anonymous access.
- **Authorization:**
  - `narativeEntryScreen`: Any authenticated user can query. `isMember`, `isAlreadySubscribed`, `confidentiality` computed server-side — never trusted from client.
  - `toggleNarativeSubscriptions`: `NarativeSubscriptionService` guards that `requesterId` is in `narative.memberList`. Throws `ForbiddenException` otherwise.
  - `acceptNarativeMembershipRequest`: Extended to check `callerId` is in `narative.memberList` (any member, not just author — D11).
  - `narative` query subscriber extension: `isSubscriber && confidentiality === 'public' && subscriptionsEnabled` — all server-side via `NarativeAccessService`. Client cannot bypass.
- **PII:** `memberList` in `NarativeEntryScreen` exposes only `id`, `firstName`, `lastName`, `profilePicture.url` — same fields as existing member exposure. `HashedPhoneNumber` never in scope.
- **Input validation:** `narativeId` validated as UUID by Prisma `findUnique`. `enabled: Boolean` — no injection risk.

---

## 9. Performance

- **N+1 — `narativeEntryScreen`:** Narative loaded via `narativeRepository.find(narativeId)` which uses `PRISMA_INCLUDE.Narative` (single query — already includes `memberList`, `likedBy`, `narativeMomentList`, `thumbnail`). `isAlreadySubscribed` computed from `likedBy` in memory — zero extra queries.
- **N+1 — subscriber access in `narative` query:** `NarativeAccessService.resolveAccess()` operates on already-loaded `NarativeEntity` (`likedBy` already in `PRISMA_INCLUDE.Narative`). Zero extra queries.
- **N+1 — `acceptNarativeMembershipRequest` (extended):** Must load narative with `memberList` to check membership. One extra query vs current implementation — acceptable.
- **Mobile — blurred thumbnail:** React Native `Image` `blurRadius` prop (native computation). No JS-side blur. Rendered only when `!isAlreadySubscribed`.
- **Caching — `getNarativeEntryScreen`:** `fetchPolicy: 'network-only'` — ensures fresh `isMember` / `isAlreadySubscribed` on every visit.

---

## 10. Mobile-Specific

- **Offline:** `NarativeEntryScreen` requires network (`network-only`). Error state renders retry button. No offline mode needed.
- **Push notifications:** No new push notifications in this feature (v0).
- **Device permissions:** None required.
- **Network resilience:**
  - Subscribe (like): optimistic update in `NarativeEntryContainer` — button updates immediately, reverts on mutation error with error toast.
  - Unsubscribe (unlike): same optimistic pattern.
  - Membership request: **not** optimistic — await mutation result, update `membershipRequested` state from response, show "Demande envoyée !" or error.
  - Toggle subscription: optimistic in `SubscriptionToggleRowContainer` — reverts on error.

---

## 11. Edge Cases & Handling

| Edge Case | Where | Handling | User-Facing |
|-----------|-------|----------|-------------|
| User is already a member → lands on entry screen | `NarativeEntryContainer` useEffect | `entryScreen.isMember === true` → `router.replace('/(app)/narative/${narativeId}')` | Transparent redirect, no flash |
| User is already subscribed → lands on entry screen | Same useEffect | `entryScreen.isAlreadySubscribed === true && subscriptionsEnabled === true` → `router.replace(...)` | Transparent redirect |
| Subscriptions disabled on a public narative | `NarativeEntryScreen` | No "S'abonner" CTA rendered; only "Demander à devenir membre" | Informational state — no subscribe option |
| Subscriber accesses private narative (likes stored but `confidentiality === 'private'`) | `NarativeAccessService.resolveAccess()` | `canAccess = false` — `assertCanAccess` throws `ForbiddenException` | Access denied |
| Subscriber queries private narative via `narative` resolver | `narative.resolver.ts` after `NarativeAccessService` integration | ForbiddenException — same as non-member | Access denied |
| "Demander l'ajout" — pending request already exists | `NarativeMembershipRequestService.create()` | Unique constraint on `(narativeId, requesterId)` guards against duplicates | "Demande envoyée !" state (button already shows this) |
| Subscriber revocation: toggle disabled | `NarativeSubscriptionService.toggleSubscriptions` | `clearAllSubscribers()` called BEFORE `updateSubscriptionsEnabled(false)` — atomic order ensures no access window | Immediate revocation |
| Subscriber revocation: public → private visibility change | `NarativeService.update()` extended | `clearAllSubscribers(narativeId)` called BEFORE persisting confidentiality change (D8) | Immediate, silent revocation |
| `PrivacyChangeWarningBanner` with `subscriberCount === 0` | NarativeSetup local state | `showPrivacyWarning = false` — no banner | Transparent |
| Subscriber promoted to member | `NarativeRepository.update()` with `memberToAddIdList` | FR7: `likedBy` disconnect for new member IDs (D12) | Transparent — user gains full member access |
| User blocks a member after subscribing | `unlikeFromMemberNaratives(blockingUserId, blockedUserId)` | Called by existing block handler — subscription revoked | Access immediately revoked |
| `narativeEntryScreen` query for unknown narativeId | `NarativeSubscriptionResolver` | `narativeRepository.find()` returns null → resolver returns null | Container shows error state |
| Migration script run twice | Script | `SystemConfig.SUBSCRIPTION_MIGRATION_COMPLETED` guard at entry — exit early | No visible effect |
| Subscriber count when subscriptions disabled (optimistic) | `SubscriptionToggleRowContainer` | Optimistic count = 0 on disable. Server confirms. On error: revert both `enabled` and `count` | Count shows 0 immediately |
| Subscriber is on NarativePage when access is revoked (toggle off or public→private) | `NarativePage` — not in scope to handle real-time | Access revocation is server-side immediate. Client only discovers on next query (no real-time push). If user refreshes / navigates back, `narative` query returns ForbiddenException → app redirects to entry screen or home. No active polling required in v0. | Next navigation triggers GQL error → handled by existing error boundary |
| Authenticated user navigates to `/(app)/narative/[id]` directly (bypassing entry) | `narative` resolver with `NarativeAccessService` | Server-side: `resolveAccess()` returns `canAccess: false` → ForbiddenException. No bypass possible via URL manipulation. | Access denied at API layer |

---

## 12. Testing Requirements

| Unit | Method | Test Cases Required |
|------|--------|---------------------|
| `NarativeAccessService` | `resolveAccess` | Member → `{ canAccess: true, isMember: true, isSubscriber: false }`; subscriber + public + enabled → `canAccess: true`; subscriber + public + disabled → `canAccess: false`; subscriber + private → `canAccess: false`; neither → `canAccess: false` |
| `NarativeAccessService` | `assertCanAccess` | No access → throws `ForbiddenException`; has access → no throw |
| `NarativeSubscriptionService` | `toggleSubscriptions` | Member disables → `clearAllSubscribers()` called THEN `updateSubscriptionsEnabled(false)`; member enables → no `clearAllSubscribers()`; non-member → `ForbiddenException` |
| `NarativeService` | `update` with `confidentiality: 'private'` | Previous `public` → `clearAllSubscribers()` called; previous already `private` → no call; previous `public` → `private` with 0 subscribers → `clearAllSubscribers()` still called (no-op) |
| `NarativeMembershipRequestService` | `accept` | Caller is member → accepted + requester added to memberList; caller is not member → `ForbiddenException`; caller is author but not in memberList → `ForbiddenException` |
| `NarativeRepository` | `update` with `memberToAddIdList` | Confirm `likedBy` disconnect executed for each new member (FR7) |
| `NarativeRepository` | `clearAllSubscribers` | After call, `likedBy` is empty for the narative |
| `NarativeEntryContainer` | `handleSubscribe` | Optimistic update applied; on mutation success → state confirmed; on mutation error → state reverted + error shown |
| `NarativeEntryContainer` | `handleRequestMembership` | Mutation called; on success → `membershipRequested = true`; on error → `membershipError` set |
| `NarativeEntryContainer` | auto-redirect useEffect | `isMember=true` → `router.replace`; `isAlreadySubscribed=true` + `subscriptionsEnabled=true` → `router.replace`; neither → no redirect |
| `SubscriptionToggleRow` | render | `narativeVisibility='private'` → returns null; `narativeVisibility='public'` + `enabled=true` → toggle ON; `narativeVisibility='public'` + `enabled=false` → toggle OFF |
