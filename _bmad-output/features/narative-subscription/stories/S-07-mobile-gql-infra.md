# S-07 — Mobile GQL Infra (Queries + Mutations + Codegen)

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Mobile
**Depends on:** S-04

---

## Goal

Create the `.gql` operation files for `getNarativeEntryScreen` and `toggleNarativeSubscriptions`, then run codegen to generate the typed React hooks.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D14 | **`narativeEntryScreen` is a new query** — not reusing `narative`. Exposes computed fields (`isAlreadySubscribed`, `isMember`, `momentCount`, `confidentiality`) that are entry-screen-specific. |
| D1 | **Subscribe = Like** — `likeNarative` / `unlikeNarative` mutations already exist in `mobile/…/shared/infra/mutations/` — reused as-is. No subscription-domain copy needed. |

### GraphQL Operations (exact)

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

### Mobile Hooks (codegen output shape)

```typescript
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
    confidentiality: 'public' | 'private';
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
// fetchPolicy: 'network-only' — set in container (S-09), not here

function useToggleNarativeSubscriptionsMutation(): [
  (options: MutationFunctionOptions<{ narativeId: string; enabled: boolean }>) => Promise<unknown>,
  MutationResult<unknown>,
];
```

### Mutations reused from page domain (no copies needed)

- `sendNarativeMembershipRequest` — `mobile/src/domains/narative/page/infra/mutations/sendNarativeMembershipRequest/`
- `cancelNarativeMembershipRequest` — `mobile/src/domains/narative/page/infra/mutations/cancelNarativeMembershipRequest/`

---

## Files to Create

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.gql` | GQL | `getNarativeEntryScreen` query — exact content from §GraphQL Operations |
| `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.hooks.ts` | Codegen | Auto-generated typed hook — do NOT hand-write |
| `mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/toggleNarativeSubscriptions.mutation.gql` | GQL | `toggleNarativeSubscriptions` mutation — exact content from §GraphQL Operations |
| `mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/toggleNarativeSubscriptions.mutation.hooks.ts` | Codegen | Auto-generated typed hook — do NOT hand-write |

## Files to Modify

None.

---

## Tasks

- [x] Create directory `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/`
- [x] Create `getNarativeEntryScreen.query.gql` with the exact verbatim content from §GraphQL Operations above
- [x] Create directory `mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/`
- [x] Create `toggleNarativeSubscriptions.mutation.gql` with the exact verbatim content from §GraphQL Operations above
- [x] Run `npm run api:generate-types` from the `mobile/` directory (script name differs from story — same codegen)
- [x] Verify `getNarativeEntryScreen.query.hooks.ts` is generated and exports `useGetNarativeEntryScreenQuery`
- [x] Verify `toggleNarativeSubscriptions.mutation.hooks.ts` is generated and exports `useToggleNarativeSubscriptionsMutation`
- [x] `.hooks.ts` files are codegen output — not hand-written

---

## Acceptance Criteria

- [x] `.query.gql` contains all fields: `id`, `title`, `location`, `startDate`, `endDate`, `momentCount`, `subscriptionsEnabled`, `isAlreadySubscribed`, `isMember`, `confidentiality`, `thumbnail { id default { id url } }`, `memberList { id firstName lastName profilePicture { id url } }`
- [x] `.mutation.gql` returns `{ id subscriptionsEnabled }` on the narative
- [x] `npm run api:generate-types` completes without errors
- [x] `useGetNarativeEntryScreenQuery` is importable from the generated hooks file
- [x] `useToggleNarativeSubscriptionsMutation` is importable from the generated hooks file
- [x] No new copies of `sendNarativeMembershipRequest` or `cancelNarativeMembershipRequest` in the subscription domain

---

## Architectural Constraints

- Do NOT add `isAlreadySubscribed`, `isMember`, or `confidentiality` to any existing `.gql` query file (D14)
- Do NOT hand-write the `.hooks.ts` codegen files — always run `npm run codegen`
- Do NOT copy `sendNarativeMembershipRequest` / `cancelNarativeMembershipRequest` into the subscription domain — page domain mutations are reused as-is

---

## Dev Notes

- The `subscription/` domain directory is new — create it from scratch following the pattern of `mobile/src/domains/narative/page/infra/`
- S-08 and S-09 both depend on the hooks generated in this story
- After codegen, verify the `confidentiality` field TypeScript type matches the `ConfidentialityType` enum mapping in the codegen config

---

## Completion Notes

**Files created:** `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.gql`, `getNarativeEntryScreen.query.hooks.ts` (codegen), `mobile/src/domains/narative/subscription/infra/mutations/toggleNarativeSubscriptions/toggleNarativeSubscriptions.mutation.gql`, `toggleNarativeSubscriptions.mutation.hooks.ts` (codegen)
**Files modified:** `backend/src/graphql/narative/entities/narative.entity.ts` (added subscriptionsEnabled field to GQL Narative type — needed for mutation return), `backend/src/graphql/narative/resolvers/narative-subscription.resolver.ts` (fixed Args type to ID)
**Deviations:** Backend GQL Narative entity needed `subscriptionsEnabled` field added for the mutation return type. Resolver args updated to use `{ type: () => ID }` to match spec.
**Escalations:** none
