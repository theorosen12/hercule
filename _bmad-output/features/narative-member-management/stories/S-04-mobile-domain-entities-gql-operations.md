# S-04 — Mobile — Domain Entities & GQL Operations

**Epic:** Narative Member Management
**Status:** Done
**Priority:** High
**Layer:** Mobile
**Depends on:** S-03

---

## Goal

Create the two domain entity files and all four GQL operation files under `mobile/src/domains/narative/members/`, then run `api:generate-types` to generate the typed Apollo hooks.

---

## Architectural Context [EMBED]

### ADR-008 — Friend Search: Reuse existing `users(isFriend: true)` query
**Decision:** The `FriendSearchSheet` container reuses `listFriendsForNarativeForm.query.gql` pattern — `users(filters: { searchQuery, isFriend: true })` with `useLazyQuery`. No new backend query needed.  
**Rationale:** This query already exists in `shared/infra/queries/listFriendsForNarativeForm` with exactly the fields needed. A new `.gql` in the `members` domain will simply copy this pattern with a different name for isolation.

### Mobile Domain Entities

```typescript
// mobile/src/domains/narative/members/domain/entities/member.entity.ts
export type NarativeMember = {
  userId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
};

// mobile/src/domains/narative/members/domain/entities/join-request.entity.ts
export type NarativeJoinRequest = {
  requestId: string;
  userId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
  date: string; // ISO
};
```

### GQL Operations

**listNarativePendingRequests.query.gql**
```graphql
query listNarativePendingRequests($narativeId: ID!) {
  listNarativePendingRequests(narativeId: $narativeId) {
    id
    requester {
      id
      firstName
      lastName
      username
      profilePicture {
        id
        url
      }
    }
    date
    status
  }
}
```

**addNarativeMember.mutation.gql**
```graphql
mutation addNarativeMember($narativeId: ID!, $userId: ID!) {
  addNarativeMember(narativeId: $narativeId, userId: $userId) {
    id
    memberList {
      id
      firstName
      lastName
      username
      profilePicture {
        id
        url
      }
    }
  }
}
```

**removeNarativeMember.mutation.gql**
```graphql
mutation removeNarativeMember($narativeId: ID!) {
  removeNarativeMember(narativeId: $narativeId) {
    id
  }
}
```

**listFriendsForMembers.query.gql** (friend search — new file, same pattern as `listFriendsForNarativeForm`)
```graphql
query listFriendsForMembers($searchQuery: String) {
  users(filters: { searchQuery: $searchQuery, isFriend: true }) {
    id
    firstName
    lastName
    username
    profilePicture {
      id
      url
    }
  }
}
```

### State Management — Data Dependencies

| Data | Source | Query |
|---|---|---|
| Member list | Apollo, initial load | `getNarativeMemberList` (existing) |
| Pending requests count (badge) | Apollo, initial load | `listNarativePendingRequests` |
| Friend search results | Apollo, lazy on keystroke | `listFriendsForMembers` (new .gql, same pattern as `listFriendsForNarativeForm`) |

---

## Files to Create

| File Path | Class/Function Name | Responsibility |
|-----------|---------------------|----------------|
| `mobile/src/domains/narative/members/domain/entities/member.entity.ts` | `NarativeMember` | Domain type for a narative member |
| `mobile/src/domains/narative/members/domain/entities/join-request.entity.ts` | `NarativeJoinRequest` | Domain type for a pending join request |
| `mobile/src/domains/narative/members/infra/queries/listNarativePendingRequests/listNarativePendingRequests.query.gql` | — | GQL query — list pending requests for a narative |
| `mobile/src/domains/narative/members/infra/mutations/addNarativeMember/addNarativeMember.mutation.gql` | — | GQL mutation — add a friend to a narative |
| `mobile/src/domains/narative/members/infra/mutations/removeNarativeMember/removeNarativeMember.mutation.gql` | — | GQL mutation — remove self from a narative |
| `mobile/src/domains/narative/members/infra/queries/listFriendsForMembers/listFriendsForMembers.query.gql` | — | GQL query — lazy friend search scoped to isFriend: true |

## Files to Modify

None. The generated hook files (`*.hooks.ts`) are produced by `api:generate-types` — do not write them manually.

---

## Tasks

- [x] Create the directory tree `mobile/src/domains/narative/members/domain/entities/` and write `member.entity.ts` with the `NarativeMember` type exactly as specified in §Mobile Domain Entities above
- [x] Create `join-request.entity.ts` with the `NarativeJoinRequest` type exactly as specified in §Mobile Domain Entities above — `date` is typed as `string` (ISO)
- [x] Create `listNarativePendingRequests.query.gql` at `mobile/src/domains/narative/members/infra/queries/listNarativePendingRequests/` with the exact query body from §GQL Operations above
- [x] Create `addNarativeMember.mutation.gql` at `mobile/src/domains/narative/members/infra/mutations/addNarativeMember/` with the exact mutation body from §GQL Operations above
- [x] Create `removeNarativeMember.mutation.gql` at `mobile/src/domains/narative/members/infra/mutations/removeNarativeMember/` with the exact mutation body from §GQL Operations above
- [x] Create `listFriendsForMembers.query.gql` at `mobile/src/domains/narative/members/infra/queries/listFriendsForMembers/` with the exact query body from §GQL Operations above
- [x] From `/mobile`, run `npm run api:generate-types` — verify that four hook files are generated:
  - `listNarativePendingRequests.query.hooks.ts`
  - `addNarativeMember.mutation.hooks.ts`
  - `removeNarativeMember.mutation.hooks.ts`
  - `listFriendsForMembers.query.hooks.ts`
- [x] Verify that the generated hook files compile without TypeScript errors (`npm run type-check` or equivalent from `/mobile`)

---

## Acceptance Criteria

- [x] `NarativeMember` type is exported from `member.entity.ts` with exactly the fields: `userId`, `firstName`, `lastName`, `username`, `avatarUrl?`
- [x] `NarativeJoinRequest` type is exported from `join-request.entity.ts` with exactly the fields: `requestId`, `userId`, `firstName`, `lastName`, `username`, `avatarUrl?`, `date: string`
- [x] All four `.gql` files exist and contain the exact query/mutation bodies from §GQL Operations
- [x] Running `api:generate-types` produces four `*.hooks.ts` files alongside their respective `.gql` files with no errors
- [x] TypeScript compilation passes with no errors on the new files

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing Message |
|-----------|-------------------|---------------------|
| `api:generate-types` fails due to schema mismatch | S-03 must be merged and backend schema updated first — do not run codegen against a stale schema | N/A (dev workflow) |

---

## Architectural Constraints

- Do NOT write `*.hooks.ts` files manually — they are always generated by `api:generate-types`
- Do NOT add fields to the GQL operations beyond those specified in §GQL Operations — no extra fields
- Do NOT use `any` type in the domain entity files
- Do NOT use `interface` — use `type` for all domain entities (matches existing codebase pattern)
- The `listFriendsForMembers.query.gql` MUST use `isFriend: true` filter — querying the global user table is a security violation (ADR-008, PRD NFR-7)

---

## Dev Notes

- Mirror the folder structure of an existing domain infra — e.g. `mobile/src/domains/narative/shared/infra/queries/listFriendsForNarativeForm/` for the exact file layout and naming convention
- `api:generate-types` is run from `/mobile` — the script is defined in `/mobile/package.json`
- The generated hook names will follow Apollo codegen convention: `useListNarativePendingRequestsQuery`, `useAddNarativeMemberMutation`, `useRemoveNarativeMemberMutation`, `useListFriendsForMembersLazyQuery` — these names will be used in S-05 and S-06

---

## Completion Notes

**Files created:**
- `mobile/src/domains/narative/members/domain/entities/member.entity.ts`
- `mobile/src/domains/narative/members/domain/entities/join-request.entity.ts`
- `mobile/src/domains/narative/members/infra/queries/listNarativePendingRequests/listNarativePendingRequests.query.gql`
- `mobile/src/domains/narative/members/infra/mutations/addNarativeMember/addNarativeMember.mutation.gql`
- `mobile/src/domains/narative/members/infra/mutations/removeNarativeMember/removeNarativeMember.mutation.gql`
- `mobile/src/domains/narative/members/infra/queries/listFriendsForMembers/listFriendsForMembers.query.gql`
- (generated) `*.hooks.ts` files for all four GQL operations

**Files modified:** none

**Deviations:** none

**Escalations:** none
