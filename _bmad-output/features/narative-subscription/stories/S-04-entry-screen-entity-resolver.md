# S-04 — NarativeEntryScreen Entity + Resolver

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Backend
**Depends on:** S-03

---

## Goal

Create the `NarativeEntryScreen` GraphQL `@ObjectType` entity and the `NarativeSubscriptionResolver` exposing the `narativeEntryScreen` query and `toggleNarativeSubscriptions` mutation.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D14 | **`narativeEntryScreen` is a new query** — not reusing `narative`. Entry screen exposes computed fields (`isAlreadySubscribed`, `isMember`, `momentCount`, `confidentiality`) that are entry-screen-specific. Avoids polluting the existing `narative` type. |
| D6 | **`NarativeEntryScreen` renders "S'abonner" CTA only when `confidentiality === 'public'`** — `confidentiality` field returned by `narativeEntryScreen` query drives this conditional on mobile. |
| D15 | **`NarativeSubscriptionService` manages toggle** — `toggleNarativeSubscriptions` mutation delegates to `NarativeSubscriptionService.toggleSubscriptions()`. |

### GraphQL Types / Operations

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

# New query
query narativeEntryScreen(narativeId: ID!): NarativeEntryScreen   # nullable — null if not found

# New mutation
mutation toggleNarativeSubscriptions(narativeId: ID!, enabled: Boolean!): Narative!
```

### Error Handling

| Error Condition | Error Type | GQL Code | Message |
|-----------------|------------|----------|---------|
| Narative not found (`narativeEntryScreen`) | `null` returned | — | Silent null — mobile container handles |
| Unauthenticated on any new resolver | `ForbiddenError` | UNAUTHENTICATED | Handled by `GqlAuthGuard` |
| Non-member calling `toggleNarativeSubscriptions` | `ForbiddenException` | FORBIDDEN | "Only members can toggle subscriptions" |

---

## Files to Create

| File Path | Class Name | Responsibility |
|-----------|------------|----------------|
| `backend/src/graphql/narative/entities/narative-entry-screen.entity.ts` | `NarativeEntryScreen` (`@ObjectType`) | GraphQL output type — all fields from §GraphQL Types above |
| `backend/src/graphql/narative/resolvers/narative-subscription.resolver.ts` | `NarativeSubscriptionResolver` | `narativeEntryScreen` query + `toggleNarativeSubscriptions` mutation |

## Files to Modify

None in this story — `narative.module.ts` was already updated in S-03.

> Note: Ensure `NarativeSubscriptionResolver` is added to the `providers` array of `narative.module.ts` (or whichever module wires GraphQL resolvers — confirm the pattern used by existing resolvers like `NarativeResolver`).

---

## Tasks

- [x] Create `backend/src/graphql/narative/entities/narative-entry-screen.entity.ts` with `@ObjectType() export class NarativeEntryScreen` containing all fields from §GraphQL Types above using NestJS GraphQL decorators (`@Field()`, `@ObjectType()`)
- [x] Map each GraphQL field to its TypeScript type: `id: string`, `title: string`, `location: string`, `startDate: Date`, `endDate: Date | null`, `momentCount: number`, `subscriptionsEnabled: boolean`, `isAlreadySubscribed: boolean`, `isMember: boolean`, `confidentiality: ConfidentialityType` — use `@Field(() => ..., { nullable: true })` for nullable fields
- [x] Create `backend/src/graphql/narative/resolvers/narative-subscription.resolver.ts` with `@Resolver() export class NarativeSubscriptionResolver`
- [x] Decorate the resolver class with `@UseGuards(GqlAuthGuard)` (from `backend/src/jwt-app/graphql-jwt.guard.ts`)
- [x] Implement `@Query(() => NarativeEntryScreen, { nullable: true }) async narativeEntryScreen(@Args('narativeId') narativeId: string, @CurrentUser() user: JwtUser): Promise<NarativeEntryScreen | null>`:
  - Load narative via `narativeRepository.get({ id: narativeId })` — return `null` if not found
  - Compute `isMember = narative.memberList.some(m => m.id === user.id)`
  - Compute `isAlreadySubscribed = narative.likedBy?.some(u => u.id === user.id) ?? false`
  - Compute `momentCount = narative.narativeMomentList?.length ?? 0`
  - Return a `NarativeEntryScreen` object mapping all fields from `narative` + computed fields
- [x] Implement `@Mutation(() => Narative) async toggleNarativeSubscriptions(@Args('narativeId') narativeId: string, @Args('enabled') enabled: boolean, @CurrentUser() user: JwtUser): Promise<Narative>`:
  - Delegate entirely to `NarativeSubscriptionService.toggleSubscriptions(narativeId, user.id, enabled)`
  - Return the result (updated `NarativeEntity`) transformed via `NarativeTransformer`
- [x] Register `NarativeSubscriptionResolver` in `graphql.module.ts` (same as `NarativeResolver`)

---

## Acceptance Criteria

- [x] `narativeEntryScreen` query returns `null` when `narativeId` does not exist in the DB
- [x] `narativeEntryScreen` query returns a `NarativeEntryScreen` with correct `isMember = true` when the authenticated user is in `narative.memberList`
- [x] `narativeEntryScreen` query returns `isAlreadySubscribed = true` when the authenticated user is in `narative.likedBy`
- [x] `narativeEntryScreen` query returns `confidentiality` matching the narative's `ConfidentialityType`
- [x] `toggleNarativeSubscriptions(narativeId, false)` returns the updated narative with `subscriptionsEnabled = false`
- [x] `toggleNarativeSubscriptions` called by non-member throws GQL FORBIDDEN error
- [x] All new resolver methods are protected by `GqlAuthGuard` — unauthenticated calls return UNAUTHENTICATED error

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| `narativeEntryScreen` for unknown `narativeId` | `narativeRepository.find()` returns null → resolver returns null | Mobile container shows error/retry state |
| Non-member calling `toggleNarativeSubscriptions` | `NarativeSubscriptionService.toggleSubscriptions()` throws `ForbiddenException` | GQL FORBIDDEN error |
| Unauthenticated call to any new resolver | `GqlAuthGuard` rejects | GQL UNAUTHENTICATED error |

---

## Architectural Constraints

- Do NOT add `isAlreadySubscribed`, `isMember`, or `momentCount` to the existing `Narative` GQL type (D14 — new type only)
- `narativeEntryScreen` must return `null` (not throw) when narative not found
- Do NOT compute access authorization in this resolver — the `narativeEntryScreen` query is intentionally open to any authenticated user (access decision is client-side via D16)
- Do NOT use `any` type anywhere

---

## Dev Notes

- Look at `backend/src/graphql/narative/entities/narative.entity.ts` for the pattern for defining `@ObjectType()` — follow the same decorator style
- Look at `backend/src/graphql/narative/resolvers/narative.resolver.ts` for `@UseGuards(GqlAuthGuard)`, `@CurrentUser()`, and `@Args()` usage patterns
- `NarativeTransformer` pattern exists at `backend/src/graphql/narative/narative.transformer.ts` — consider using it for the `NarativeEntryScreen` field mapping if the pattern fits
- `momentCount` comes from `narative.narativeMomentList?.length` — this relation is already in `PRISMA_INCLUDE.Narative` so no extra query is needed

---

## Completion Notes

**Files created:** `backend/src/graphql/narative/entities/narative-entry-screen.entity.ts`, `backend/src/graphql/narative/resolvers/narative-subscription.resolver.ts`
**Files modified:** `backend/src/graphql/graphql.module.ts`
**Deviations:** Story says `narativeRepository.find()` but `find()` throws on not-found; used `get({ id: narativeId })` which returns null — correct behavior per spec (query must return null, not throw).
**Escalations:** none
