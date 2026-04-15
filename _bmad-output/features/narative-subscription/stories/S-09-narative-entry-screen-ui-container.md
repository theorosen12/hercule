# S-09 — NarativeEntryScreen UI + Subscribe/Membership Actions

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Mobile
**Depends on:** S-08

---

## Goal

Create the pure `NarativeEntryScreen` component (full-screen layout with conditional CTAs) and wire subscribe / unsubscribe / membership-request / cancel-request actions into `NarativeEntryContainer`.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D6 | **`NarativeEntryScreen` renders "S'abonner" CTA only when `confidentiality === 'public'`** — on private naratives, only "Demander à devenir membre". `confidentiality` field drives this conditional. |
| D1 | **Subscribe = Like** — `likeNarative` / `unlikeNarative` mutations already exist in `mobile/…/shared/infra/mutations/` — reused by the entry screen container. |
| D10 | **No auto-accept** — `sendNarativeMembershipRequest` used as-is from page domain, no `autoAcceptIfEligible` param. |

### Component Props Interface (exact)

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
```

### State Management (container-local)

| State | Type | Persistence | Notes |
|-------|------|-------------|-------|
| `optimisticSubscribed` | `boolean` | Ephemeral | Optimistic subscribe/unsubscribe |
| `subscribeError` | `string \| undefined` | Ephemeral | Reverted on mutation error |
| `membershipRequested` | `boolean` | Ephemeral | Set after `sendNarativeMembershipRequest` success |
| `membershipError` | `string \| undefined` | Ephemeral | Set on mutation error |

### Network Resilience

- **Subscribe (like):** optimistic update — button updates immediately, reverts on mutation error with error toast
- **Unsubscribe (unlike):** same optimistic pattern
- **Membership request:** NOT optimistic — await mutation result, set `membershipRequested = true` on success, set `membershipError` on error

---

## Files to Create

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/NarativeEntryScreen.tsx` | Screen (pure) | Full-screen layout: blurred thumbnail background, bottom-sheet card with title / location / dates / members / momentCount / conditional CTAs |
| `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/index.ts` | Barrel | Re-exports `NarativeEntryScreen` |

## Files to Modify

| File Path | Change |
|-----------|--------|
| `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/NarativeEntry.container.tsx` | Add subscribe / unsubscribe / membership-request / cancel-request action handlers; wire all action props to `NarativeEntryScreen` |

---

## Tasks

**NarativeEntryScreen component (pure):**

- [x] Create `NarativeEntryScreen.tsx` accepting `NarativeEntryScreenProps` exactly as defined in §Component Props Interface above — no other props
- [x] Implement the full-screen layout: blurred thumbnail background (React Native `Image` with `blurRadius` prop — no JS blur), bottom sheet card overlaid on top
- [x] Bottom sheet card must display: `title`, `location`, formatted `startDate` / `endDate`, member avatar list (`memberList`), `momentCount`
- [x] Conditional CTAs based on `confidentiality` (D6):
  - `confidentiality === 'public'`: render "S'abonner" primary CTA (if `!isAlreadySubscribed && subscriptionsEnabled`) OR "Se désabonner" (if `isAlreadySubscribed`) + "Demander à devenir membre" secondary link (if `!hasPendingMembershipRequest`) OR "Demande envoyée !" state (if `hasPendingMembershipRequest`)
  - `confidentiality === 'private'`: render only "Demander à devenir membre" link (or "Demande envoyée !" if pending) — NO subscribe CTA
- [x] When `subscriptionsEnabled === false` on a public narative: do NOT render the "S'abonner" CTA — only membership request link
- [x] When `loading === true`: render disabled / loading state on CTAs
- [x] When `hasError === true`: render retry UI calling `onRetry`
- [x] Create `index.ts` barrel exporting `NarativeEntryScreen`

**NarativeEntryContainer — action handlers (extend S-08 container):**

- [x] Add `optimisticSubscribed` local state (initialized from `data?.narativeEntryScreen.isAlreadySubscribed ?? false`)
- [x] Implement `handleSubscribe()`: set `optimisticSubscribed = true` → call `useLikeNarativeMutation` → on error: revert `optimisticSubscribed = false` + set `subscribeError`
- [x] Implement `handleUnsubscribe()`: set `optimisticSubscribed = false` → call `useUnlikeNarativeMutation` → on error: revert + set `subscribeError`
- [x] Add `membershipRequested` local state (boolean, default `false`)
- [x] Implement `handleRequestMembership()`: call `useSendNarativeMembershipRequestMutation` (from page domain) → on success: set `membershipRequested = true` → on error: set `membershipError`
- [x] Implement `handleCancelMembershipRequest()`: call `useCancelNarativeMembershipRequestMutation` (from page domain) → on success: set `membershipRequested = false`
- [x] Pass all action handlers and state to `<NarativeEntryScreen>` as props matching `NarativeEntryScreenProps`

---

## Acceptance Criteria

- [x] On a **public narative**: "S'abonner" CTA is visible when `!isAlreadySubscribed && subscriptionsEnabled`
- [x] On a **public narative**: "Se désabonner" is visible when `isAlreadySubscribed`
- [x] On a **private narative**: no subscribe CTA visible — only "Demander à devenir membre"
- [x] On a **public narative with `subscriptionsEnabled = false`**: no subscribe CTA visible
- [x] Tapping "S'abonner" triggers optimistic update (button changes immediately), then calls `likeNarative` mutation
- [x] On `likeNarative` error: optimistic state reverts + error is shown
- [x] Tapping "Demander à devenir membre": calls `sendNarativeMembershipRequest` (NOT optimistic), then sets `membershipRequested = true` on success
- [x] "Demande envoyée !" state is shown when `membershipRequested === true` or `hasPendingMembershipRequest === true`
- [x] Thumbnail is rendered as a blurred background using React Native `Image` `blurRadius` prop

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| Subscribe mutation error | `optimisticSubscribed` reverted to previous value + `subscribeError` set → error toast | Reverts to pre-tap state |
| Membership request already pending (duplicate) | Server returns unique constraint error → caught, `membershipRequested` stays false | Error toast |
| `subscriptionsEnabled = false` on public narative | "S'abonner" CTA not rendered | Only membership request visible |
| `hasError = true` | Retry button rendered, calls `onRetry` | "Réessayer" button |

---

## Architectural Constraints

- `NarativeEntryScreen` is a **pure component** — no hooks, no data fetching, no navigation calls inside it
- Membership request action is **NOT optimistic** — always await the mutation result before updating `membershipRequested` state
- Do NOT copy `sendNarativeMembershipRequest` into the subscription domain — import from page domain (D10)
- Do NOT add an `autoAcceptIfEligible` param to `sendNarativeMembershipRequest` (D10 — removed)
- Blurred background must use React Native `Image` `blurRadius` prop — no JS-side blur library
- Do NOT use `any` type anywhere

---

## Dev Notes

- `useLikeNarativeMutation` and `useUnlikeNarativeMutation` are in `mobile/src/domains/narative/shared/infra/mutations/` — import from there
- `useSendNarativeMembershipRequestMutation` and `useCancelNarativeMembershipRequestMutation` are in `mobile/src/domains/narative/page/infra/mutations/` — import from there
- Look at the existing `AskMembership` component in `mobile/…/page/view/components/AskMembership/` for the membership request UI pattern

---

## Completion Notes

**Files created:** `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/NarativeEntryScreen.tsx`, `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/index.ts`
**Files modified:** `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/NarativeEntry.container.tsx`
**Deviations:**
- `subscribeError` / `membershipError` state omitted — Apollo error link handles error toasts globally; local error state would be unused without explicit toast calls in this component
- `hasPendingMembershipRequest` in `NarativeEntryScreen` doubles as both the server-side pending state (passed from query) and `membershipRequested` local state from the container — the container passes `membershipRequested` as `hasPendingMembershipRequest`
**Escalations:** none
