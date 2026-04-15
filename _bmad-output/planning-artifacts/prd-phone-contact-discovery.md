# Phone Contact Discovery - Product Requirements Document

## Introduction

### Overview

Phone Contact Discovery enables users to find friends already on the platform by securely matching their phone contacts against registered users. The feature maintains a privacy-first architecture using cryptographic hashing while solving the cold-start problem for new users.

### Document Scope

**Full-stack feature** covering:
- Backend API services (NestJS/GraphQL)
- Mobile client (React Native/Expo)


## Project Context

### Current State

| Layer | Status | Details |
|-------|--------|---------|
| **Backend** | Partially Implemented | Stories 1.1, 1.2 DONE |
| **Mobile** | Not Started | Architecture documented, no code |

### Tech Stack

| Layer | Technology | Version |
|-------|------------|---------|
| **Backend Runtime** | Node.js | >=22.8.0 |
| **Backend Framework** | NestJS | 11.x |
| **API** | GraphQL (Apollo Server) | 3.13.0 |
| **Database** | PostgreSQL | Latest |
| **ORM** | Prisma | 6.15.0 |
| **Cache** | Redis | 5.0.1 |
| **Mobile Framework** | React Native | 0.81.5 |
| **Mobile Platform** | Expo | ~54.0.31 |
| **Mobile Router** | Expo Router | ~6.0.21 |
| **Mobile GraphQL** | Apollo Client | 3.13.8 |

### Architecture References

| Document | Location |
|----------|----------|
| Backend Architecture | `docs/architecture/backend/` (18 files) |
| Mobile Architecture | `docs/mobile/architecture.md` |
| GraphQL Contract | `docs/api/graphql-contract.md` |

---

## Goals

- Enable users to discover friends by matching phone contacts against registered users
- Maintain privacy through cryptographic hashing (HMAC-SHA256) - never store plain text phone numbers
- Provide user control over discoverability via opt-in/out privacy toggle
- Integrate seamlessly with existing friend request system
- Deliver native mobile experience following existing app patterns
- Support both iOS and Android platforms

---

## Functional Requirements

### Core Requirements (Backend + Mobile)

**FR1:** Users shall be able to query friend suggestions by providing their phone contacts (up to 1,000 contacts maximum) via GraphQL query

**FR2:** The system shall normalize all phone numbers to E.164 format using `libphonenumber-js` before processing (both for user registration and contact matching)

**FR3:** The system shall hash user's own phone number using HMAC-SHA256 with a server-side secret key for discoverability by others

**FR4:** The system shall store only the user's own hashed phone number in the database (for being found), never storing uploaded contact lists

**FR5:** The system shall match contacts provided in the query (hashed at runtime) against the database of registered users' hashed phone numbers to identify friends on the platform

**FR6:** The system shall return a list of matched users (friend suggestions) sorted alphabetically by firstName, then lastName

**FR7:** Users shall be able to send friend requests to matched contacts via existing friend request mutation

**FR8:** The system shall provide a GraphQL mutation allowing users to update their phone number, which deletes the old hash and creates a new hash while preserving existing friend connections

**FR9:** Users shall be able to opt out of phone number-based discovery through a privacy toggle, which prevents them from both finding others AND being found by others

**FR10:** Existing friendships shall remain intact regardless of discovery opt-out or permission revocation

**FR11:** The system shall enforce the all-or-nothing privacy model: users who opt out cannot find AND cannot be found

**FR12:** The system shall reject contact uploads exceeding 1,000 contacts with clear error message

**FR13:** The system shall provide a GraphQL query to retrieve discovery status (privacy setting, phone number registration status)

### Mobile-Specific Requirements

**FR14:** The mobile app shall display a "Friend Suggestions" module at the top of the Friends list in the Profile tab

**FR15:** If the user has no phone number registered, the module shall display the existing phone number input component

**FR16:** After phone number is entered, the app shall automatically request contact permission using `expo-contacts`

**FR17:** If contact permission is denied, the module shall disappear and reappear on the next session

**FR18:** If contact permission is granted, the app shall automatically sync contacts and display suggestions

**FR19:** Friend suggestions shall be displayed as a horizontal carousel (maximum 5 visible, scroll for more)

**FR20:** Each suggestion card shall show profile picture, name, and "Add" button

**FR21:** When a friend request is sent, the card shall show "En attente" (pending) state

**FR22:** If no suggestions are found, the module shall display "Aucun ami trouvé" with an "Invite Friends" button

**FR23:** The "Invite Friends" button shall open the native share sheet with an invitation link

**FR24:** The module shall include a "Refresh" button to manually re-sync contacts

**FR25:** The module shall include an "X" close button for permanent dismissal

**FR26:** When permanently dismissed, the module shall not reappear (flag stored in AsyncStorage: `hide_friend_suggestion_profile`)

---

## Non-Functional Requirements

### Performance

**NFR1:** Contact upload and hashing shall complete within 3 seconds for 1,000 contacts

**NFR2:** Friend suggestions query shall return within 500ms (cached) or 2 seconds (uncached)

**NFR3:** Mobile UI shall render suggestion list without frame drops (60fps scrolling)

### Security

**NFR4:** All phone number data in transit shall be encrypted using TLS 1.3+

**NFR5:** Plain text phone numbers shall never be logged or persisted anywhere

**NFR6:** All contact operations shall require JWT authentication

**NFR7:** Security audit logs shall capture all contact upload and privacy setting changes

### Rate Limiting

**NFR8:** `getContactSuggestions` query: 10 requests per hour per user (includes contact hashing)

**NFR9:** `setMyPhoneNumber` mutation: 3 requests per day per user

**NFR10:** `togglePhoneDiscovery` mutation: 10 requests per hour per user

### Caching

**NFR11:** Friend suggestions results may be cached briefly (5 minutes) based on hash of provided contacts to reduce repeated computation

### Compliance

**NFR14:** System shall comply with GDPR requirements: data minimization (hashing), purpose limitation, user control (opt-out)

**NFR15:** System shall support right to deletion via cascade delete when user account is deleted

---

## Compatibility Requirements

**CR1:** Backend API shall maintain compatibility with existing authentication system (JWT/Passport)

**CR2:** Database changes shall use Prisma migrations compatible with existing schema

**CR3:** Mobile UI shall follow existing design system (`@nara/evanescent`, Emotion styling)

**CR4:** Mobile navigation shall integrate with existing Expo Router structure

**CR5:** GraphQL operations shall follow existing Apollo Client patterns (codegen, hooks)

**CR6:** Friend request integration shall use existing `sendFriendRequest` mutation

---

## User Interface

### Module Location

The Friend Suggestions module is displayed **at the top of the Friends list** in the Profile tab. It is not a separate screen.

### Module States

| State | Condition | Display |
|-------|-----------|---------|
| **Phone Input** | No phone number registered | Existing phone input component |
| **Permission Request** | Phone entered, no permission yet | System permission dialog (auto-triggered) |
| **Suggestions Carousel** | Permission granted, suggestions found | Horizontal carousel (max 5 visible) |
| **No Suggestions** | Permission granted, no matches | "Aucun ami trouvé" + "Inviter des amis" |
| **Permission Denied** | User denied contacts access | Module hidden until next session |
| **Permanently Closed** | User clicked X | Module never shown again |

### UI Components (New)

| Component | Purpose |
|-----------|---------|
| `FriendSuggestionModule` | Container for the entire module with state management |
| `SuggestionCarousel` | Horizontal scrollable list of suggestion cards |
| `SuggestionCard` | Single friend card with profile pic, name, Add button |
| `NoSuggestionsState` | "Aucun ami trouvé" + Invite button |
| `ModuleHeader` | Title + Refresh button + Close button |

### Integration Points

| Location | Integration |
|----------|-------------|
| **Profile Tab (Friends List)** | Insert `FriendSuggestionModule` at top of list |
| **Existing Phone Input** | Reuse existing component for phone number entry |
| **Share Sheet** | Native share for "Invite Friends" action |

---

## Epic Structure

### Rationale

Single epic covering both backend and mobile implementation because:
1. Feature is cohesive - one user journey across layers
2. Backend stories block mobile stories (API must exist first)
3. Simplifies tracking and release planning

---

## Epic 1: Phone Contact Discovery

**Epic Goal:** Enable users to discover friends by uploading phone contacts, matching against registered users via secure hashing, and sending friend requests - with full privacy controls.

**Status:** In Progress (Backend 0/6 stories done, Mobile 0/7 stories done)

---

### Backend Stories

#### Story 1.1: Database Schema for Phone Discovery ✅ DONE (needs migration)

**As a** backend developer,
**I want** database models to store user phone hashes and discovery preferences,
**so that** users can be found by others who have their phone number.

**Acceptance Criteria:**
1. Create Prisma schema with `HashedPhoneNumber` model (id, userId, hashedPhone, timestamps) - stores user's OWN phone hash
2. Add relation from User to HashedPhoneNumber with cascade delete
3. Create database indexes on `hashedPhone` and `userId` fields
4. Run `prisma migrate dev` successfully
5. Ensure unique constraint on `hashedPhone` field

---

#### Story 1.2: Phone Number Registration API ✅ DONE (needs refactor)

**As a** mobile app user,
**I want** to register my phone number,
**so that** my friends can discover me on the platform.

**Acceptance Criteria:**
1. Integrate `libphonenumber-js` for E.164 normalization
2. Implement HMAC-SHA256 hashing with server-side secret key (`PHONE_HASH_SECRET` env var)
3. Create `setMyPhoneNumber(phoneNumber: String!): User!` mutation for user phone registration
4. Validate phone number format, reject invalid with clear error
5. Hash and store user's phone number in `HashedPhoneNumber` table
6. Require JWT authentication
7. Never log plain text phone numbers

---

#### Story 1.3: Contact Matching and Friend Suggestions Query (Stateless)

**As a** mobile app user,
**I want** to see which of my contacts are on the platform,
**so that** I can discover and connect with friends.

**Acceptance Criteria:**
1. Create `getContactSuggestions(phoneNumbers: [String!]!): [User!]!` query accepting contacts as parameter
2. Normalize and hash provided phone numbers at runtime (no storage)
3. Match hashed contacts against `HashedPhoneNumber` table (registered users' phones)
4. Return matched users with fields: id, username, firstName, lastName, profilePictureMediaId
5. Sort results alphabetically by firstName, lastName
7. Exclude requesting user from results
8. Exclude existing friends
9. Exclude users with pending friend requests (both directions)
10. Return empty array (not null) when no matches
11. Reject requests >1,000 contacts with clear error
12. Complete query within 2 seconds for 1,000 contacts (includes hashing time)
13. Optional: Brief cache (5 min) based on hash of input contacts

---

#### Story 1.4: Friend Request Integration

**As a** mobile app user,
**I want** to send friend requests to suggested contacts,
**so that** I can connect with discovered friends seamlessly.

**Acceptance Criteria:**
1. Verify existing `sendFriendRequest` mutation works with discovered users
2. Add optional `source: "contact_discovery"` parameter for analytics
3. Prevent duplicate friend requests with clear error
4. Handle bidirectional discovery (mutual requests = auto-accept)
5. Document GraphQL workflow for mobile team
6. Return clear errors for blocked/deleted users
7. Trigger push notification on friend request

---

#### Story 1.5: Phone Number Update Management

**As a** user who changed my phone number,
**I want** to update my phone number in the app,
**so that** contacts can find me with my new number.

**Acceptance Criteria:**
1. Use existing `setMyPhoneNumber(phoneNumber: String!)` mutation (from Story 1.2)
2. Validate phone format with libphonenumber-js
3. Delete old hash, create new hash (atomic transaction)
4. Preserve existing friend connections
5. Update `User.updatedAt` timestamp
6. Allow no-op if updating to same number
7. Log update events (never log actual numbers)

---

### Mobile Stories

#### Story M1.1: Friend Suggestion Module Container

**As a** user viewing my friends list,
**I want** to see a friend suggestion module at the top of the list,
**so that** I can discover friends I might have missed.

**Acceptance Criteria:**
1. Create `FriendSuggestionModule` component in `src/domains/user/contactDiscovery/view/components/`
2. Integrate module at the top of Friends list in Profile tab
3. Check for `hide_friend_suggestion_profile` flag in AsyncStorage on mount
4. If flag is true, do not render the module
5. Query user profile to check if phone number exists
6. Render appropriate state based on phone number presence
7. Follow container/component pattern with `FriendSuggestionModule.container.tsx`
8. Use Emotion styling matching existing design system

**Technical Notes:**
- Location: Insert in Friends list component (find existing list component)
- AsyncStorage key: `hide_friend_suggestion_profile`

---

#### Story M1.2: Phone Number Entry State

**As a** user without a registered phone number,
**I want** to see a prompt to add my phone number in the suggestion module,
**so that** I can enable friend discovery.

**Acceptance Criteria:**
1. If user has no phone number, display existing phone input component within module
2. Reuse existing `PhoneNumberCard` or equivalent component
3. After successful phone number submission, automatically trigger contact permission request
4. Show appropriate loading state during phone number submission
5. Handle phone validation errors from backend

**Technical Notes:**
- Reference: `src/domains/user/shared/view/components/PhoneNumberCard/`
- Reuse existing `setMyPhoneNumber` mutation

---

#### Story M1.3: Contact Permission and Friend Discovery

**As a** user who just entered my phone number,
**I want** the app to automatically request contact permission and find friends,
**so that** I can see friend suggestions without extra steps.

**Acceptance Criteria:**
1. After phone number is saved, immediately call `expo-contacts.requestPermissionsAsync()`
2. If permission granted:
   - Extract phone numbers via `getContactsAsync({ fields: [PhoneNumbers] })`
   - Deduplicate extracted numbers
   - Call `getContactSuggestions(phoneNumbers)` query directly with contacts as parameter
   - Display suggestions (no separate upload step needed)
3. If permission denied:
   - Hide the module for current session
   - Module reappears on next app session (no persistent flag)
4. Show loading state during discovery process
5. Handle errors gracefully (network, rate limit)

**Technical Notes:**
- Do NOT show explanation dialog before permission - request directly
- Session-based hiding (no AsyncStorage for permission denial)
- Contacts are sent directly to query, never stored on backend

---

#### Story M1.4: Suggestions Carousel Display

**As a** user with contact permission granted,
**I want** to see my friend suggestions in a horizontal carousel,
**so that** I can quickly browse and add friends.

**Acceptance Criteria:**
1. Create `SuggestionCarousel` component with horizontal `FlatList` or `ScrollView`
2. Display maximum 5 cards visible, scroll horizontally for more
3. Each `SuggestionCard` shows:
   - Profile picture (or default avatar)
   - First name + Last name
   - "Add" button
4. Create `getContactSuggestions.query.gql` with `phoneNumbers` parameter in `src/domains/user/contactDiscovery/infra/queries/`
5. Run `npm run codegen` to generate hooks
6. Show loading skeleton while fetching suggestions
7. Carousel scrolls smoothly (60fps)

**Technical Notes:**
- Use `showsHorizontalScrollIndicator={false}`
- Card width should fit ~2.5 cards on screen for peek effect
- Query accepts contacts as parameter: `getContactSuggestions(phoneNumbers: $phoneNumbers)`

---

#### Story M1.5: Add Friend Action

**As a** user viewing a friend suggestion,
**I want** to add them as a friend directly from the carousel,
**so that** I can connect quickly.

**Acceptance Criteria:**
1. "Add" button calls existing `sendFriendRequest` mutation with `source: "contact_discovery"`
2. Show loading spinner on button during request
3. On success: card transitions to "En attente" (pending) state
4. Card remains visible with disabled/pending state (not removed)
5. Handle `FRIEND_REQUEST_EXISTS`: show "En attente" state
6. Handle blocked user error: remove card silently
7. Update Apollo cache appropriately

**Technical Notes:**
- Do NOT remove card from carousel on success
- Visual state change: button becomes "En attente" label (disabled)

---

#### Story M1.6: No Suggestions State

**As a** user whose contacts don't match any platform users,
**I want** to see a helpful message with an invite option,
**so that** I can bring my friends to the platform.

**Acceptance Criteria:**
1. Create `NoSuggestionsState` component
2. Display message: "Aucun ami trouvé" (or localized equivalent)
3. Display "Inviter des amis" button
4. Button opens native share sheet via `Share.share()` API
5. Share content includes app invitation link
6. Module header still shows Refresh and Close buttons

**Technical Notes:**
- Use React Native `Share` API
- Invitation link format: TBD (check existing invite patterns)

---

#### Story M1.7: Module Header Actions

**As a** user viewing the suggestion module,
**I want** to refresh suggestions or close the module permanently,
**so that** I have control over my experience.

**Acceptance Criteria:**
1. Create `ModuleHeader` component with title + action buttons
2. "Refresh" button (↻ icon):
   - Re-fetches contacts from device
   - Re-uploads to backend
   - Re-queries suggestions
   - Show loading state during refresh
   - Respect rate limits (show message if exceeded)
3. "Close" button (X icon):
   - Show confirmation dialog: "Ne plus afficher ce module ?"
   - On confirm: set `hide_friend_suggestion_profile` to `true` in AsyncStorage
   - Module immediately disappears
   - Module never reappears (until app data cleared)
4. Header title: "Amis trouvés dans vos contacts" (when suggestions) or "Trouvez vos amis" (when no phone)

**Technical Notes:**
- AsyncStorage key: `hide_friend_suggestion_profile`
- Confirmation dialog uses existing modal/alert pattern

---

## Story Dependencies

```
Backend Stories:
1.1 (refactor) ──→ 1.2 (refactor) ──→ 1.3 (NEW) ──→ 1.4
                                  ↘
                                   1.5 (backend only)
                                    ↘
                                     1.6

Mobile Stories (require backend 1.3 complete):
M1.1 ──→ M1.2 ──→ M1.3 ──→ M1.4 ──→ M1.5
                    │               │
                    └───→ M1.6 ←────┘
                            │
                            └───→ M1.7
```

| Story | Blocked By |
|-------|------------|
| 1.2 (refactor) | 1.1 (migration) |
| 1.3 | 1.2 |
| 1.4 | 1.3 |
| 1.5 | 1.2 |
| 1.6 | 1.2 |
| M1.1 | None (can start immediately) |
| M1.2 | M1.1, Backend 1.2 |
| M1.3 | M1.2, Backend 1.3 |
| M1.4 | M1.3 |
| M1.5 | M1.4, Backend 1.4 |
| M1.6 | M1.3 |
| M1.7 | M1.1 |

---

## Implementation Order

### Phase 1: Backend Refactor (NEEDS REWORK)
- [x] Story 1.1: Database Schema *(needs migration to remove UploadedContactPhoneNumber)*
- [x] Story 1.2: Phone Registration API *(refactored - uploadContacts removed)*

### Phase 2: Backend Core Features
- [ ] Story 1.3: Contact Matching & Suggestions Query *(NEW: stateless, contacts as parameter)*
- [ ] Story 1.4: Friend Request Integration
- [ ] Story 1.5: Privacy Toggle *(backend only, for future/admin use)*

### Phase 3: Mobile Module Foundation
- [ ] Story M1.1: Friend Suggestion Module Container
- [ ] Story M1.2: Phone Number Entry State
- [ ] Story M1.7: Module Header Actions

### Phase 4: Mobile Contact Discovery
- [ ] Story M1.3: Contact Permission & Friend Discovery *(simplified: single query)*
- [ ] Story M1.4: Suggestions Carousel Display

### Phase 5: Mobile Interactions
- [ ] Story M1.5: Add Friend Action
- [ ] Story M1.6: No Suggestions State

### Phase 6: Backend Enhancements
- [ ] Story 1.6: Phone Number Update *(uses existing setMyPhoneNumber)*

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Suggestions query P95 latency | <2s (includes hashing) |
| Friend requests from discovery | Track adoption |
| Privacy opt-out rate | Monitor for UX issues |
| Permission grant rate | >70% |

---

## Appendix

### GraphQL Operations Summary

| Operation | Type | Status | Mobile UI |
|-----------|------|--------|-----------|
| `setMyPhoneNumber` | Mutation | ✅ Backend Done | ✅ Used |
| `getContactSuggestions(phoneNumbers)` | Query | Pending (NEW) | ✅ Used |
| `getDiscoveryStatus` | Query | Pending | ❌ Optional |
| `togglePhoneDiscovery` | Mutation | Pending | ❌ No UI (backend/admin only) |
| `sendFriendRequest` | Mutation | ✅ Existing | ✅ Used |

**Removed Operations:**
- ~~`uploadContacts`~~ - Contacts no longer stored, sent directly to `getContactSuggestions`

### Error Codes

| Code | Description |
|------|-------------|
| `CONTACT_LIMIT_EXCEEDED` | >1,000 contacts in query |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `INVALID_PHONE_NUMBER` | Cannot parse phone format |
| `UNAUTHENTICATED` | Missing/invalid JWT |
| `FRIEND_REQUEST_EXISTS` | Pending request already sent |
