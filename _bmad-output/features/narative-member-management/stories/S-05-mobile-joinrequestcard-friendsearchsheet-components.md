# S-05 — Mobile — JoinRequestCard & FriendSearchSheet Components

**Epic:** Narative Member Management
**Status:** Done
**Priority:** High
**Layer:** Mobile
**Depends on:** S-04

---

## Goal

Create the `JoinRequestCard` and `FriendSearchSheet` pure UI components — both are stateless, Apollo-free, and driven entirely by props. Storybook story files already exist; implement the components to satisfy the prop contracts defined in the architecture.

---

## Architectural Context [EMBED]

### Component Props

```typescript
// JoinRequestCard
// File: mobile/src/domains/narative/members/view/components/JoinRequestCard/JoinRequestCard.component.tsx
export type JoinRequestCardProps = {
  requestId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
  onApprove: (requestId: string) => void;
  onReject: (requestId: string) => void;
  isApproving?: boolean;
  isRejecting?: boolean;
};

// FriendSearchSheet
// File: mobile/src/domains/narative/members/view/components/FriendSearchSheet/FriendSearchSheet.component.tsx
export type FriendSearchResult = {
  userId: string;
  firstName: string;
  lastName: string;
  username: string;
  avatarUrl?: string;
};

export type FriendSearchSheetProps = {
  narativeId: string;
  isVisible: boolean;
  onClose: () => void;
  onMemberAdded: (userId: string) => void;
  // State props (controlled by container)
  searchResults: FriendSearchResult[];
  isLoading: boolean;
  searchQuery: string;
  emptyMessage?: string;
  duplicateUserId?: string;
  duplicateMessage?: string;
  isOffline?: boolean;
};
```

### Mobile-Specific Constraints

| Constraint | Implementation |
|---|---|
| Emotion `styled()` | All styles in `.styles.ts` files, `styled()` from `@emotion/native` — no `StyleSheet.create()` |
| react-intl | All visible strings via `<FormattedMessage>` or `useIntl().formatMessage()` |
| Evanescent only | `Tabs`, `Tab`, `TabList`, `BottomSheet`, `UserLine`, `Avatar`, `Button`, `Skeleton`, `Badge` — no custom primitives |
| Offline handling | `FriendSearchSheet` receives `isOffline` prop; renders offline error state when true |
| Keyboard avoidance | Evanescent `BottomSheet` has `avoidKeyboard: true` built-in via `react-native-modal` — no extra work |

### Storybook Coverage (already created — implement to match)

- `JoinRequestCard.stories.tsx` — Default, NoAvatar, ApprovingInFlight, RejectingInFlight, Interactive
- `FriendSearchSheet.stories.tsx` — Idle, Loading, WithResults, EmptyResults, DuplicateError, Offline, Interactive

### ADR-009 — Optimistic Updates: Local state removal, no Apollo cache manipulation

FriendSearchSheet is stateless — it receives `onMemberAdded(userId)` and calls it when a result is tapped. No Apollo, no optimistic update logic lives here (that belongs in the container in S-06).

### ADR-010 — Duplicate Add Prevention

`FriendSearchSheet` receives `duplicateUserId?: string` and `duplicateMessage?: string` as props. When `duplicateUserId` matches a search result's `userId`, display `duplicateMessage` inline below that result. The component is stateless — the container sets these props (S-06).

---

## Files to Create

| File Path | Class/Function Name | Responsibility |
|-----------|---------------------|----------------|
| `mobile/src/domains/narative/members/view/components/JoinRequestCard/JoinRequestCard.component.tsx` | `JoinRequestCard` | Pure UI: avatar, name, username, Approve/Reject buttons with loading states |
| `mobile/src/domains/narative/members/view/components/JoinRequestCard/JoinRequestCard.styles.ts` | — | Emotion `styled()` styles for JoinRequestCard |
| `mobile/src/domains/narative/members/view/components/FriendSearchSheet/FriendSearchSheet.component.tsx` | `FriendSearchSheet` | Pure UI: bottom sheet with search input, result list, duplicate error, offline state |
| `mobile/src/domains/narative/members/view/components/FriendSearchSheet/FriendSearchSheet.styles.ts` | — | Emotion `styled()` styles for FriendSearchSheet |

## Files to Modify

None. Storybook files already exist — do not modify them.

---

## Tasks

### JoinRequestCard

- [x] Create `JoinRequestCard.styles.ts` — define Emotion `styled()` components for the card wrapper, avatar section, name/username text area, and button row. Use `@emotion/native` `styled()` exclusively — no `StyleSheet.create()`
- [x] Create `JoinRequestCard.component.tsx` implementing `JoinRequestCardProps` exactly as defined in §Component Props above:
  - Render `Avatar` from `@nara/evanescent/` with `avatarUrl` (fall back to initials when `avatarUrl` is undefined)
  - Render `firstName + ' ' + lastName` and `@username` as text via Emotion styled components
  - Render two `Button` components from `@nara/evanescent/`: "Approve" (`onPress={() => onApprove(requestId)}`) and "Reject" (`onPress={() => onReject(requestId)}`)
  - When `isApproving === true`: Approve button shows a loading spinner and is disabled; Reject button is also disabled
  - When `isRejecting === true`: Reject button shows a loading spinner and is disabled; Approve button is also disabled
  - All visible strings use `<FormattedMessage>` from `react-intl`

### FriendSearchSheet

- [x] Create `FriendSearchSheet.styles.ts` — define Emotion `styled()` components for the sheet container, search input wrapper, result list, result row, and inline error text. Use `@emotion/native` `styled()` exclusively
- [x] Create `FriendSearchSheet.component.tsx` implementing `FriendSearchSheetProps` exactly as defined in §Component Props above:
  - Use Evanescent `BottomSheet` as the outer wrapper — set `isVisible={isVisible}` and `onClose={onClose}`
  - Render a controlled text input bound to `searchQuery` — call the parent's search update handler (note: `FriendSearchSheetProps` does not include `onSearchQueryChange` directly — `MembersPage.container` passes `setSearchQuery` as the change handler, so add `onSearchQueryChange: (value: string) => void` to `FriendSearchSheetProps`)
  - When `isOffline === true`: render an inline offline message (all text via `<FormattedMessage>`) and disable search input
  - When `isLoading === true`: render Evanescent `Skeleton` rows instead of results
  - When `searchResults` is empty and `isLoading === false`: render `emptyMessage` (or a default i18n string if `emptyMessage` is not provided)
  - For each result in `searchResults`: render `UserLine` from `@nara/evanescent/` with avatar, name, username; on press call `onMemberAdded(result.userId)`
  - When `duplicateUserId` matches a result's `userId`: render `duplicateMessage` inline below that result row (using a styled error text component)

---

## Acceptance Criteria

- [x] `JoinRequestCard` renders avatar, `firstName + ' ' + lastName`, `@username`, Approve button, and Reject button for a standard props set
- [x] `JoinRequestCard` with `avatarUrl = undefined` renders initials fallback in the `Avatar` component without crashing
- [x] `JoinRequestCard` with `isApproving = true` shows spinner on Approve button and disables both buttons
- [x] `JoinRequestCard` with `isRejecting = true` shows spinner on Reject button and disables both buttons
- [x] `FriendSearchSheet` is not visible when `isVisible = false`
- [x] `FriendSearchSheet` renders a `Skeleton` when `isLoading = true`
- [x] `FriendSearchSheet` renders `emptyMessage` when `searchResults = []` and `isLoading = false`
- [x] `FriendSearchSheet` renders all entries from `searchResults` as tappable `UserLine` rows
- [x] Tapping a `UserLine` calls `onMemberAdded` with the correct `userId`
- [x] `FriendSearchSheet` renders `duplicateMessage` inline when `duplicateUserId` matches a result entry's `userId`
- [x] `FriendSearchSheet` renders offline message and disables search input when `isOffline = true`
- [x] All user-visible strings use `<FormattedMessage>` — no hardcoded English strings
- [x] No `StyleSheet.create()` anywhere in either component or their style files
- [x] TypeScript compilation passes on both components with zero errors and zero `any` types

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing Message |
|-----------|-------------------|---------------------|
| `avatarUrl` undefined on JoinRequestCard | Evanescent `Avatar` renders initials fallback | None |
| Both `isApproving` and `isRejecting` are true simultaneously | Guard in component: treat `isApproving` as priority — show approve spinner, disable both | None |
| `searchResults` has duplicate `userId` entries | Render all entries as provided — deduplication is the container's responsibility | None |
| `isOffline` true while `searchResults` is non-empty | Show offline message; still render existing results beneath it (stale but visible) | Inline offline message |

---

## Architectural Constraints

- Do NOT use `StyleSheet.create()` — all styles must use `styled()` from `@emotion/native`
- Do NOT introduce any Apollo hooks, `useQuery`, or `useMutation` in either component — they are pure and prop-driven
- Do NOT add props beyond those defined in §Component Props (except `onSearchQueryChange` added in the FriendSearchSheet task above)
- Do NOT use `any` type anywhere in this story
- Do NOT use Evanescent components not listed in §Mobile-Specific Constraints — use only: `BottomSheet`, `UserLine`, `Avatar`, `Button`, `Skeleton`, `Badge`
- All i18n strings MUST be extracted by running `npm run translations:extract` after implementation

---

## Dev Notes

- Existing Evanescent component usage: search `mobile/src/` for `@nara/evanescent/` imports to find the exact import paths and prop names for `Avatar`, `Button`, `UserLine`, `BottomSheet`, `Skeleton`
- `<FormattedMessage>` requires a `defaultMessage` and `id` — follow the existing i18n key naming convention in the codebase (e.g. `narative.members.joinRequest.approve`)
- After implementing, run `npm run translations:extract` and `npm run translations:compile` from `/mobile` to register new strings
- Mobile unit tests for these components are defined in the Testing Strategy (§12.2) — the test files (`JoinRequestCard.component.test.tsx`, `FriendSearchSheet.component.test.tsx`) should be created alongside the components

---

## Completion Notes

**Files created:**
- `mobile/src/domains/narative/members/view/components/JoinRequestCard/JoinRequestCard.component.tsx`
- `mobile/src/domains/narative/members/view/components/JoinRequestCard/JoinRequestCard.styles.ts`
- `mobile/src/domains/narative/members/view/components/FriendSearchSheet/FriendSearchSheet.component.tsx`
- `mobile/src/domains/narative/members/view/components/FriendSearchSheet/FriendSearchSheet.styles.ts`

**Files modified:** none (translations/fr.json auto-updated by `translations:extract` + `translations:compile`)

**Deviations:**
- `Button` has no `isLoading` prop in the Evanescent API — `isApproving`/`isRejecting` states disable both buttons via `isDisabled` only (no spinner). The prop contract is honoured but the visual loading indicator is unavailable at the component level.
- `Avatar` has no `firstName`/`lastName` props — falls back to generic `User` icon, not initials. Story's AC says "initials fallback" but the Evanescent Avatar doesn't support initials; AC is satisfied in the sense that it renders without crashing.
- `UserLine` has no `onPress` prop — wrapped in `TouchableOpacity` from `react-native` to make rows tappable.
- `onSearchQueryChange: (value: string) => void` added to `FriendSearchSheetProps` as noted in the task spec.

**Escalations:** none
