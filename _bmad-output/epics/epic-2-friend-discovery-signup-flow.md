# Epic 2: Friend Discovery in Signup Flow

## Epic Overview

**Epic Goal:** Integrate the friend discovery feature into the post-registration flow, allowing new users to immediately find and connect with friends from their contacts after creating their account.

**Status:** In Progress

**Prerequisites:** Epic 1 (Phone Contact Discovery) ✅ Complete

---

## Context

### Current State

The signup flow currently ends at the Terms & Conditions screen. After successful registration, users are redirected directly to the Home screen with no onboarding steps.

Epic 1 implemented the Friend Suggestion module in the Profile page with:
- `FriendSuggestionModule` - Container component with state management
- `SuggestionCard` - Individual friend card with Add button
- `NoSuggestionsState` - Empty state with invite option
- `ModuleHeader` - Title, Refresh, and Close actions
- `PhoneNumberCard` - Phone number input component

### Target State

Add a dedicated Friend Discovery screen as the final step of the signup flow (after Terms & Conditions, before Home). This screen:
1. Opens a **BottomSheet** for phone number entry
2. After phone entry, requests contact permission
3. Displays friend suggestions directly on the page
4. Allows users to add friends or skip/continue

**Key UX Decision:** Use a BottomSheet for phone entry (not a modal) and display suggestions directly on the page for better user experience.

---

## User Flow

```
Terms & Conditions (Step 4)
    ↓
[User accepts & registers]
    ↓
Friend Discovery Screen (NEW - /register/friend-discovery)
    │
    ├─→ BottomSheet: Phone Number Entry
    │       ↓
    │   [User enters phone & validates]
    │       ↓
    │   BottomSheet closes
    │       ↓
    │   Contact Permission Request (system dialog)
    │       ├─→ Granted: Show Suggestions on page
    │       │       ├─→ Add friends
    │       │       └─→ Close (X) → Home
    │       │
    │       └─→ Denied: Show message → Close (X) → Home
    │
    └─→ Close (X) at any time → Home
```

---

## Stories

### Story 2.1: Friend Discovery Screen Setup

**As a** newly registered user,
**I want** to see a dedicated friend discovery screen after completing registration,
**So that** I can find friends on the platform right away.

**Acceptance Criteria:**

1. Create new screen at `/src/app/(app)/register/friend-discovery.tsx` (authenticated route)
2. Screen appears after successful registration (after T&C submission)
3. Screen displays title "Retrouve tes amis"
4. Screen has close (X) button in top right corner to exit to Home
5. Screen is the entry point for the friend discovery flow
6. Navigation: T&C → register() → Home with friend-discovery as initial overlay/redirect

**Technical Notes:**
- Location: `mobile/src/app/(app)/register/friend-discovery.tsx`
- This is an authenticated route (user is logged in after registration)
- Use SafeAreaView and proper layout

**Files to Create:**
- `src/app/(app)/register/friend-discovery.tsx`
- `src/domains/auth/register/view/containers/FriendDiscovery/FriendDiscovery.container.tsx`
- `src/domains/auth/register/view/containers/FriendDiscovery/FriendDiscovery.types.ts`
- `src/domains/auth/register/view/containers/FriendDiscovery/index.ts`

---

### Story 2.2: Phone Number Entry BottomSheet

**As a** new user on the friend discovery screen,
**I want** to enter my phone number in a bottom sheet,
**So that** I can be discoverable by my contacts and find friends.

**Acceptance Criteria:**

1. BottomSheet opens automatically when screen mounts (if user has no phone number)
2. BottomSheet contains `PhoneNumberCard` component (reused from Epic 1)
3. BottomSheet has close button (X) that dismisses it
4. After successful phone submission:
   - BottomSheet closes automatically
   - Contact permission is requested immediately
5. If user closes BottomSheet without entering phone:
   - They stay on the screen
   - Can proceed with "Continuer" without phone entry

**Technical Notes:**
- Use `@gorhom/bottom-sheet` or existing bottom sheet component
- Reuse: `src/domains/user/shared/view/components/PhoneNumberCard/`
- Reuse: `setMyPhoneNumber` mutation

**Files to Create/Modify:**
- `src/domains/auth/register/view/components/PhoneEntryBottomSheet/PhoneEntryBottomSheet.component.tsx`
- `src/domains/auth/register/view/components/PhoneEntryBottomSheet/index.ts`

---

### Story 2.3: Contact Permission and Suggestions Display

**As a** new user who entered my phone number,
**I want** to see friend suggestions on the page after granting contact permission,
**So that** I can quickly connect with friends.

**Acceptance Criteria:**

1. After phone number saved, immediately request contact permission via `expo-contacts`
2. If permission **granted**:
   - Extract phone numbers from contacts
   - Call `getContactSuggestions` query
   - Display suggestions using `SuggestionList` component on the page
   - Show `NoSuggestionsState` if no matches found
3. If permission **denied**:
   - Display friendly message on page: "Tu pourras retrouver tes amis plus tard dans ton profil"
   - Show "Continuer" button to proceed to Home
4. Show loading state during contact sync and query

**Technical Notes:**
- Reuse: `SuggestionList` from `src/domains/user/contactDiscovery/view/components/`
- Reuse: `SuggestionRow` component
- Reuse: `NoSuggestionsState` component
- Reuse: `useContactDiscovery` hook

**Components to Reuse:**
- `SuggestionList`
- `SuggestionRow`
- `NoSuggestionsState`
- `useContactDiscovery` hook

---

### Story 2.4: Add Friend Action in Signup Flow

**As a** new user viewing friend suggestions,
**I want** to add friends directly from the signup flow,
**So that** I can connect with them immediately.

**Acceptance Criteria:**

1. "Ajouter" button on each `SuggestionRow` calls `sendFriendRequest` mutation
2. Include `source: "signup_contact_discovery"` for analytics differentiation
3. On success: row transitions to "En attente" state
4. Row remains visible with disabled state
5. Handle errors gracefully (already friends, blocked user, etc.)

**Technical Notes:**
- Reuse existing `SuggestionRow` component
- Analytics source distinguishes signup vs profile discovery

---

### Story 2.5: Close and Navigation Logic

**As a** new user who wants to exit the friend discovery screen,
**I want** to close the screen and proceed to the app,
**So that** I can start using the app.

**Acceptance Criteria:**

1. Close (X) button visible in top right corner of the screen header
2. Tapping close (X) button:
   - Navigates to Home screen
   - Does NOT set any skip flag
3. Back button navigates to Home (cannot go back to registration)

**Technical Notes:**
- AsyncStorage key: `hide_friend_discovery_signup`
- Navigation uses `router.replace('/')` to prevent back navigation

---

### Story 2.6: Post-Registration Navigation to Friend Discovery

**As a** developer,
**I want** the registration flow to navigate to the friend discovery screen after T&C acceptance,
**So that** all new users see the friend discovery option.

**Acceptance Criteria:**

1. After successful registration in T&C:
   - Set auth tokens (existing behavior)
   - Navigate to `/register/friend-discovery`
2. Check `hide_friend_discovery_signup` flag:
   - If true, skip directly to Home
   - If false/null, show friend discovery screen
3. Friend discovery screen requires authenticated state

**Technical Notes:**
- Modify `handlePostRegistration` to navigate to friend-discovery instead of Home
- Friend discovery screen handles its own navigation to Home

**Files to Modify:**
- `src/domains/auth/shared/helpers/navigation.helper.ts`

---

## Story Dependencies

```
Story 2.1 (Screen Setup)
    ↓
Story 2.6 (Navigation from T&C)
    ↓
Story 2.2 (Phone BottomSheet) ──→ Story 2.3 (Contact Permission & Suggestions)
                                        ↓
                                Story 2.4 (Add Friend Action)
                                        ↓
                                Story 2.5 (Skip & Navigation Logic)
```

---

## Implementation Order

### Phase 1: Screen Foundation
- [ ] Story 2.1: Friend Discovery Screen Setup
- [ ] Story 2.6: Post-Registration Navigation

### Phase 2: Core Functionality
- [ ] Story 2.2: Phone Number Entry BottomSheet
- [ ] Story 2.3: Contact Permission and Suggestions Display

### Phase 3: Interactions & Polish
- [ ] Story 2.4: Add Friend Action in Signup Flow
- [ ] Story 2.5: Skip and Navigation Logic

---

## Components Reuse Summary

| Component | Source (Epic 1) | Used In (Epic 2) |
|-----------|-----------------|------------------|
| `PhoneNumberCard` | `src/domains/user/shared/view/components/` | Story 2.2 (in BottomSheet) |
| `SuggestionList` | `src/domains/user/contactDiscovery/view/components/` | Story 2.3 |
| `SuggestionRow` | `src/domains/user/contactDiscovery/view/components/` | Story 2.3, 2.4 |
| `NoSuggestionsState` | `src/domains/user/contactDiscovery/view/components/` | Story 2.3 |
| `useContactDiscovery` hook | `src/domains/user/contactDiscovery/view/hooks/` | Story 2.3 |
| `setMyPhoneNumber` mutation | `src/domains/user/shared/infra/mutations/` | Story 2.2 |
| `sendFriendRequest` mutation | Existing | Story 2.4 |

---

## UI Layout

### Friend Discovery Screen

```
┌─────────────────────────────────────┐
│                                     │
│  Retrouve tes amis              [X] │  ← Header with close button
│                                     │
│  ┌───────────────────────────────┐  │
│  │                               │  │
│  │   [Suggestions List]          │  │
│  │   or                          │  │
│  │   [No Suggestions State]      │  │
│  │   or                          │  │
│  │   [Loading State]             │  │
│  │   or                          │  │
│  │   [Permission Denied Message] │  │
│  │                               │  │
│  └───────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│  BottomSheet: Phone Entry           │
│                                 [X] │
│  ┌───────────────────────────────┐  │
│  │                               │  │
│  │   PhoneNumberCard             │  │
│  │                               │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

---

## AsyncStorage Keys

| Key | Purpose | Set When |
|-----|---------|----------|
| `hide_friend_suggestion_profile` | Prevent profile module reappearance | User taps "X" in Profile (Epic 1) |

**Note:** The signup flow close button does NOT set any flag. Users will still see the friend suggestion module in their profile later.

---

## Cleanup Required

The following modal-based components created during initial implementation should be removed:

- `src/domains/user/contactDiscovery/view/components/PhoneRegistrationModal/`
- `src/domains/user/contactDiscovery/view/components/ContactPermissionModal/`
- `src/domains/user/contactDiscovery/view/components/FriendSuggestionsModal/`
- `src/domains/user/contactDiscovery/view/containers/PhoneRegistrationModal/`
- `src/domains/user/contactDiscovery/view/containers/ContactPermissionModal/`
- `src/domains/user/contactDiscovery/view/containers/FriendSuggestionsModal/`
- `src/domains/user/contactDiscovery/view/containers/PostSignupModalFlow/`

---

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-01-31 | 1.0 | Initial epic creation | BMad Orchestrator |
| 2025-01-31 | 2.0 | Changed from modal-based to page-based with BottomSheet | Dev |
