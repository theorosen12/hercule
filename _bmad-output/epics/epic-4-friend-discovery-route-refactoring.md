# Epic 4: Friend Discovery Route Refactoring

## Epic Overview

**Epic Goal:** Refactor the post-registration friend discovery flow by splitting the single `/register/friend-discovery` route into two dedicated routes (`/register/phone-number` and `/register/friend-discovery`), simplifying state management by using `useQuery` instead of lazy queries, deriving UI state from query results, and leveraging Apollo cache for friend request status instead of local state.

**Status:** Draft

**Prerequisites:** Epic 2 (Friend Discovery in Signup Flow) - Complete

---

## Context

### Current State

The friend discovery flow lives in a single route `/register/friend-discovery` that handles:
- Phone number entry via a BottomSheet (auto-opens if no phone)
- Contact permission request (triggered after phone submission)
- Contact extraction and `getContactSuggestions` lazy query
- Friend request status tracking via local `Map<string, FriendRequestStatus>` state
- Complex state machine: `initial → loading → suggestions/empty/denied`

**Pain Points:**
1. **Single route does too much** - phone entry, permission, discovery all in one screen
2. **Lazy query** needed because we don't know if we have permission/phone yet at mount time
3. **Local state for friend request status** - `requestStatuses` Map duplicates what the server knows
4. **Complex state machine** - multiple states to manage (`initial`, `loading`, `suggestions`, `empty`, `denied`) instead of deriving from query

### Target State

Split into two routes with cleaner separation of concerns:

1. **`/register/phone-number`** - Dedicated phone entry screen
   - On submit: ask contact permission + submit phone form
   - If permission granted + phone saved → navigate to `/register/friend-discovery`
   - Else → navigate to home (`/`)

2. **`/register/friend-discovery`** - Pure friend suggestions screen
   - Uses `useQuery` (not lazy) since we know permission is granted when arriving
   - Derives UI state from query results (`loading`, `data`, `error`)
   - `getContactSuggestions` returns `friendRequest` field (server-side status)
   - `sendFriendRequest` mutation auto-updates Apollo cache (no local state)

---

## User Flow

```
Terms & Conditions (Step 4)
    |
[User accepts & registers]
    |
Phone Number Screen (NEW - /register/phone-number)
    |
[User enters phone & submits]
    |
    |--- 1. Ask contact permission (system dialog)
    |--- 2. Submit phone number (setMyPhoneNumber mutation)
    |
    +---> Permission GRANTED + Phone saved
    |         |
    |         v
    |     Friend Discovery Screen (/register/friend-discovery)
    |         |
    |         +---> useQuery: getContactSuggestions
    |         |         (re-extracts contacts, queries with phoneNumbers)
    |         |
    |         +---> Suggestions displayed (friendRequest status from server)
    |         |         +---> "Ajouter" → sendFriendRequest → cache auto-update
    |         |         +---> Close (X) → Home
    |         |
    |         +---> No suggestions → NoSuggestionsState → Close (X) → Home
    |
    +---> Permission DENIED or Phone skipped
              |
              v
          Home (/)
```

---

## Stories

### Story 4.1: Split Routes - Create Phone-Number Screen with Permission & Navigation

**As a** newly registered user,
**I want** a dedicated phone number entry screen,
**So that** the flow is clearer and each screen has a single responsibility.

**Key Changes:**
- New route `/register/phone-number` with full-screen phone entry (no BottomSheet)
- On submit: ask contact permission + submit phone
- Conditional navigation based on permission result
- Update `handlePostRegistration` to navigate to phone-number route

---

### Story 4.2: Backend - Return friendRequest Field in getContactSuggestions

**As a** mobile client,
**I want** the `getContactSuggestions` query to include the `friendRequest` field with status,
**So that** I don't need local state to track friend request status.

**Key Changes:**
- Add `friendRequest { id status }` to `getContactSuggestions` response
- Returns `null` if no friend request exists, or the request with its status

---

### Story 4.3: Refactor Friend-Discovery Route with useQuery and Derived State

**As a** developer,
**I want** to simplify the friend-discovery screen using `useQuery` and derived state,
**So that** the code is simpler and the UI state is always in sync with the server.

**Key Changes:**
- Use `useQuery` instead of lazy query (permission is guaranteed at this point)
- Re-extract contacts on mount (permission already granted)
- Derive UI state from query `{ loading, data, error }` instead of state machine
- Remove `PhoneEntryBottomSheet` from friend-discovery
- Remove complex state machine (`initial/loading/suggestions/empty/denied`)

---

### Story 4.4: Apollo Cache Auto-Update for sendFriendRequest

**As a** user adding friends from the discovery screen,
**I want** the friend request status to update automatically in the UI after sending a request,
**So that** the UI stays in sync without manual local state tracking.

**Key Changes:**
- Remove local `requestStatuses` Map state
- Read friend request status from `friendRequest` field in cached query data
- `sendFriendRequest` mutation auto-updates Apollo cache
- `SuggestionRow` reads status from Apollo cache (no prop drilling of statuses)

---

## Story Dependencies

```
Story 4.1 (Phone-Number Route)
    |
    +---> Story 4.3 (Refactor Friend-Discovery with useQuery)
              |
Story 4.2 (Backend: friendRequest field) ---+
              |
              v
         Story 4.4 (Apollo Cache Auto-Update)
```

- **4.1** can be done independently
- **4.2** can be done independently (backend)
- **4.3** depends on 4.1 (needs the route split done first)
- **4.4** depends on 4.2 (needs `friendRequest` field from backend) and 4.3 (needs the refactored screen)

---

## Implementation Order

### Phase 1: Independent Work (parallelizable)
- [ ] Story 4.1: Split Routes - Create Phone-Number Screen
- [ ] Story 4.2: Backend - Return friendRequest Field

### Phase 2: Frontend Refactoring
- [ ] Story 4.3: Refactor Friend-Discovery with useQuery
- [ ] Story 4.4: Apollo Cache Auto-Update for sendFriendRequest

---

## UI Layout

### Phone Number Screen (NEW)

```
+-------------------------------------+
|                                      |
|  Retrouve tes amis              [X]  |  <- Header with close/skip
|                                      |
|  +-------------------------------+   |
|  |                               |   |
|  |   [Phone Number Input]        |   |
|  |   +33 6 12 34 56 78           |   |
|  |                               |   |
|  |   [Valider]                   |   |
|  |                               |   |
|  +-------------------------------+   |
|                                      |
+-------------------------------------+
```

### Friend Discovery Screen (SIMPLIFIED)

```
+-------------------------------------+
|                                      |
|  Tes amis sont sur Nara !       [X]  |  <- Header
|                                      |
|  +-------------------------------+   |
|  |  [Suggestion Row]             |   |  <- friendRequest.status from server
|  |  Name           [En attente]  |   |  <- auto-updated via Apollo cache
|  |                               |   |
|  |  [Suggestion Row]             |   |
|  |  Name              [Ajouter]  |   |  <- no friendRequest yet
|  |                               |   |
|  |  [Suggestion Row]             |   |
|  |  Name              [Ajouter]  |   |
|  +-------------------------------+   |
|                                      |
+-------------------------------------+
```

---

## Key Architectural Changes

| Aspect | Before (Epic 2) | After (Epic 4) |
|--------|-----------------|----------------|
| Routes | Single `/register/friend-discovery` | `/register/phone-number` + `/register/friend-discovery` |
| Phone entry | BottomSheet in friend-discovery | Dedicated phone-number screen |
| Permission | Requested after phone in same screen | Requested on phone submit, gates navigation |
| Query type | `useLazyQuery` (manual trigger) | `useQuery` (auto on mount) |
| State management | State machine (initial/loading/etc) | Derived from `{ loading, data, error }` |
| Friend request status | Local `Map<string, FriendRequestStatus>` | Server-side `friendRequest { status }` in query |
| Cache updates | None (local state) | Apollo cache auto-update on mutation |

---

## Components Impact

| Component | Action | Description |
|-----------|--------|-------------|
| `phone-number.tsx` | CREATE | New route for phone entry screen |
| `PhoneNumberScreen` container | CREATE | Phone entry + permission + navigation |
| `FriendDiscovery.container.tsx` | MODIFY | Simplify: remove BottomSheet, use useQuery, derive state |
| `PhoneEntryBottomSheet` | DELETE | No longer needed (phone is its own screen) |
| `useContactDiscovery` hook | MODIFY | Simplify for useQuery pattern |
| `getContactSuggestions` query | MODIFY | Add `friendRequest { id status }` field |
| `SuggestionRow` | MODIFY | Read status from `friendRequest` instead of prop |
| `SuggestionList` | MODIFY | Remove `requestStatuses` prop |
| `Register.provider.tsx` | MODIFY | Navigate to phone-number instead of friend-discovery |

---

## GraphQL Changes

### Modified Query

```graphql
query GetContactSuggestions($phoneNumbers: [String!]!) {
  getContactSuggestions(phoneNumbers: $phoneNumbers) {
    id
    username
    firstName
    lastName
    narativesCount
    profilePicture {
      id
      url
    }
    friendRequest {        # NEW
      id
      status              # PENDING, ACCEPTED, DECLINED, etc.
    }
  }
}
```

### Mutation Cache Behavior

```graphql
mutation sendFriendRequest($userId: String!) {
  sendFriendRequest(requestedUserId: $userId) {
    id
    status               # Returns PENDING
  }
}
```

The mutation response should be configured to update the `friendRequest` field on the matching `ContactSuggestion` in Apollo cache, either via:
- Cache normalization (if `friendRequest` is a normalized type with `__typename + id`)
- Manual `cache.modify` on the parent `ContactSuggestion`
- `update` function in the mutation call

---

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2026-02-07 | 1.0 | Initial epic creation | BMad Orchestrator |
