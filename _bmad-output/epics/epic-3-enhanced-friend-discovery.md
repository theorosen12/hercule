# Epic 3: Enhanced Friend Discovery Screen

## Epic Overview

**Epic Goal:** Enhance the post-registration Friend Discovery screen with a two-section layout: friend suggestions (contacts on Nara) with narative count, and device contacts to invite - both with "Voir plus" pagination (3 at a time).

**Status:** Draft

**Prerequisites:** Epic 2 (Friend Discovery in Signup Flow) - Complete

---

## Context

### Current State

The Friend Discovery screen (post-registration) displays:
- A single content area showing either suggestions OR "no suggestions" state
- NoSuggestionsState handles device contact invites with "Voir plus" (3 by 3)
- SuggestionRow shows user info but NOT narative membership count

### Target State

Split the screen into two distinct sections:

1. **Section "Amis sur Nara"** (top)
   - Display matched contacts already on the platform
   - Show **narative count** for each suggestion (total naratives they're member of)
   - Load 3 at a time with "Voir plus" button
   - Reuse `SuggestionRow` component (enhanced with narative count)

2. **Section "Inviter des contacts"** (bottom)
   - Display device contacts NOT on the platform
   - Allow SMS invite for each contact
   - Load 3 at a time with "Voir plus" button
   - Reuse logic from `NoSuggestionsState` component

---

## User Flow

```
Friend Discovery Screen
    │
    ├─→ Section 1: "Amis sur Nara"
    │       ├─→ Shows 3 suggestions initially
    │       ├─→ Each card shows: avatar, name, narative count, "Ajouter" button
    │       ├─→ "Voir plus" loads next 3
    │       └─→ Empty state if no matches
    │
    └─→ Section 2: "Inviter des contacts"
            ├─→ Shows 3 device contacts initially
            ├─→ Each card shows: avatar placeholder, name, phone, "Inviter" button
            ├─→ "Voir plus" loads next 3
            └─→ Hidden if no device contacts available
```

---

## Stories

### Story 3.1: Backend - Add Narative Count to Contact Suggestions

**As a** mobile app,
**I want** the `getContactSuggestions` query to return narative membership count for each user,
**So that** I can display how active each suggested friend is on the platform.

**Acceptance Criteria:**

1. Modify `getContactSuggestions` query response to include `narativeCount: Int!` field
2. Count ALL naratives where the user is a member (not just public)
3. Optimize query to avoid N+1 problem (use aggregation or dataloader)
4. Return 0 if user is not member of any narative

**Technical Notes:**
- Field: `narativeCount` on User type or inline in suggestion response
- Query should remain performant (<2s for 1000 contacts)

---

### Story 3.2: Two-Section Layout for Friend Discovery

**As a** new user on the Friend Discovery screen,
**I want** to see two distinct sections: friends on Nara and contacts to invite,
**So that** I can both add existing users and invite new ones.

**Acceptance Criteria:**

1. Split content area into two scrollable sections
2. Section 1: "Amis sur Nara" header with suggestions list
3. Section 2: "Inviter des contacts" header with device contacts list
4. Both sections visible simultaneously (vertical scroll)
5. Section 2 hidden if no device contacts available
6. Section 1 shows empty state message if no suggestions found

**Technical Notes:**
- Location: `FriendDiscovery.container.tsx`
- Use ScrollView with two sections
- Reuse section header styling from existing components

---

### Story 3.3: Paginated Suggestions with "Voir plus"

**As a** new user viewing friend suggestions,
**I want** to see suggestions loaded 3 at a time with a "Voir plus" button,
**So that** I can browse through all my contacts on the platform progressively.

**Acceptance Criteria:**

1. Initially display maximum 3 suggestions
2. Show "Voir plus" button if more than 3 suggestions exist
3. Clicking "Voir plus" reveals next 3 suggestions
4. Button disappears when all suggestions are shown
5. Show count indicator: "Voir plus (X restants)"
6. Smooth animation when revealing new items

**Technical Notes:**
- Implement pagination state in container
- No API pagination needed (client-side slicing of full results)
- Consider using `LayoutAnimation` for smooth reveal

---

### Story 3.4: Display Narative Count on Suggestion Row

**As a** new user viewing a friend suggestion,
**I want** to see how many Naratives the person is member of,
**So that** I can gauge how active they are on the platform.

**Acceptance Criteria:**

1. Display narative count on `SuggestionRow` component
2. Format: "{count} Nara" or "{count} Naras" (pluralization)
3. Show "Nouveau" badge if count is 0 (new user)
4. Position: below the user's name
5. Use secondary text styling

**Technical Notes:**
- Modify `SuggestionRow.component.tsx`
- Add `narativeCount` to `ContactSuggestion` type
- Update GraphQL query to fetch this field

---

### Story 3.5: Device Contacts Invite Section

**As a** new user on the Friend Discovery screen,
**I want** to see my device contacts who are not on Nara,
**So that** I can invite them to join the platform.

**Acceptance Criteria:**

1. Display device contacts in "Inviter des contacts" section
2. Initially show maximum 3 contacts
3. Show "Voir plus" button if more than 3 contacts exist
4. Each contact shows: name, phone number, "Inviter" button
5. "Inviter" button opens SMS with personalized invite message
6. Reuse invite logic from `NoSuggestionsState`

**Technical Notes:**
- Extract device contacts logic from `NoSuggestionsState` into reusable hook or component
- Create `InviteContactRow` component (or reuse/adapt existing)
- Use existing SMS invite mechanism with personalized message

---

### Story 3.6: Empty States for Each Section

**As a** new user on the Friend Discovery screen,
**I want** to see appropriate empty states for each section,
**So that** I understand when no contacts are available.

**Acceptance Criteria:**

1. Section "Amis sur Nara" empty state: "Aucun ami trouvé sur Nara"
2. Section "Inviter des contacts": hidden entirely if no device contacts
3. Both sections empty: show combined message with generic invite option
4. Permission denied: show message for both sections

**Technical Notes:**
- Handle edge cases gracefully
- Maintain clean UI when one or both sections are empty

---

## Story Dependencies

```
Story 3.1 (Backend: Narative Count)
    ↓
Story 3.4 (Display Narative Count)
    ↓
Story 3.2 (Two-Section Layout) ──→ Story 3.3 (Paginated Suggestions)
                               ──→ Story 3.5 (Invite Section)
                                        ↓
                                Story 3.6 (Empty States)
```

---

## Implementation Order

### Phase 1: Backend Enhancement
- [ ] Story 3.1: Backend - Add Narative Count to Contact Suggestions

### Phase 2: UI Structure
- [ ] Story 3.2: Two-Section Layout for Friend Discovery

### Phase 3: Features
- [ ] Story 3.3: Paginated Suggestions with "Voir plus"
- [ ] Story 3.4: Display Narative Count on Suggestion Row
- [ ] Story 3.5: Device Contacts Invite Section

### Phase 4: Polish
- [ ] Story 3.6: Empty States for Each Section

---

## UI Layout

### Enhanced Friend Discovery Screen

```
┌─────────────────────────────────────┐
│                              [X]    │
│                                     │
│              🤩                      │
│     Tes amis sont sur Nara !        │
│                                     │
│  ┌───────────────────────────────┐  │
│  │  AMIS SUR NARA                │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │ 👤 Marie Dupont         │  │  │
│  │  │    12 Naras    [Ajouter]│  │  │
│  │  └─────────────────────────┘  │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │ 👤 Jean Martin          │  │  │
│  │  │    5 Naras     [Ajouter]│  │  │
│  │  └─────────────────────────┘  │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │ 👤 Sophie Bernard       │  │  │
│  │  │    Nouveau     [Ajouter]│  │  │
│  │  └─────────────────────────┘  │  │
│  │                               │  │
│  │     [ Voir plus (4 restants) ]│  │
│  └───────────────────────────────┘  │
│                                     │
│  ┌───────────────────────────────┐  │
│  │  INVITER DES CONTACTS         │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │ 👤 Paul Durand          │  │  │
│  │  │    +33 6 12...  [Inviter]│  │  │
│  │  └─────────────────────────┘  │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │ 👤 Claire Petit         │  │  │
│  │  │    +33 7 89...  [Inviter]│  │  │
│  │  └─────────────────────────┘  │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │ 👤 Marc Leroy           │  │  │
│  │  │    +33 6 45...  [Inviter]│  │  │
│  │  └─────────────────────────┘  │  │
│  │                               │  │
│  │     [ Voir plus (12 restants)]│  │
│  └───────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

---

## Components to Create/Modify

| Component | Action | Description |
|-----------|--------|-------------|
| `SuggestionRow` | MODIFY | Add narative count display |
| `InviteContactRow` | CREATE | Row for device contact with invite button |
| `PaginatedSection` | CREATE | Reusable section with "Voir plus" logic |
| `FriendDiscoveryContainer` | MODIFY | Implement two-section layout |
| `getContactSuggestions` | MODIFY | Add narativeCount field (backend) |

---

## GraphQL Changes

### Modified Query Response

```graphql
type ContactSuggestion {
  id: ID!
  firstName: String!
  lastName: String!
  username: String
  profilePictureMediaId: String
  narativeCount: Int!  # NEW FIELD
}
```

---

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-02-01 | 1.0 | Initial epic creation | BMad Orchestrator |
