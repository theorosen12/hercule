# S-10 — SubscriptionToggleRow + PrivacyChangeWarningBanner

**Epic:** narative-subscription
**Status:** Done
**Priority:** Medium
**Layer:** Mobile
**Depends on:** S-07

---

## Goal

Create the `SubscriptionToggleRow` component + container and the `PrivacyChangeWarningBanner` component, then integrate both into the NarativeSetup (edit) screen.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D7 | **`SubscriptionToggleRow` not rendered when `narativeVisibility === 'private'`** — component returns null. The toggle is meaningless on private naratives. Absence is cleaner than a disabled state. |
| D9 | **`PrivacyChangeWarningBanner` — inline banner, no modal** in `NarativeSetup` screen. Warning is informational, not a decision point. Banner shown below visibility dropdown when `subscriberCount > 0` AND transition to private. Confirmation implicit via "Enregistrer". ConfirmationModal rejected (creates a decision gate where PRD says there is none). |

### Component Props Interfaces (exact)

```typescript
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

// Container props
interface SubscriptionToggleRowContainerProps {
  narativeId: string;
  narativeVisibility: 'public' | 'private'; // passed from settings screen
}
```

### State Management (container-local)

| State | Location | Type | Notes |
|-------|----------|------|-------|
| `toggleEnabled` (optimistic) | `SubscriptionToggleRowContainer` | `boolean` | Reverted on error |
| `subscriberCount` (optimistic) | `SubscriptionToggleRowContainer` | `number` | Set to 0 on disable, restored on error |
| `showPrivacyWarning` | NarativeSetup/edit screen | `boolean` | `prevVisibility === 'public' && newVisibility === 'private' && subscriberCount > 0` |

### Toggle Optimistic Behavior

- Toggle disabled: `toggleEnabled = false` + `subscriberCount = 0` optimistically → call `toggleNarativeSubscriptions(narativeId, false)` → on error: revert both
- Toggle enabled: `toggleEnabled = true` optimistically → call `toggleNarativeSubscriptions(narativeId, true)` → on error: revert

---

## Files to Create

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/view/components/SubscriptionToggleRow/SubscriptionToggleRow.tsx` | Component (pure) | Toggle row + subscriber count label. Returns `null` when `narativeVisibility === 'private'` (D7) |
| `mobile/src/domains/narative/subscription/view/components/SubscriptionToggleRow/index.ts` | Barrel | Re-exports |
| `mobile/src/domains/narative/subscription/view/components/PrivacyChangeWarningBanner/PrivacyChangeWarningBanner.tsx` | Component (pure) | Inline warning banner: "La confidentialité semble changer · Les {N} abonnements existants seront supprimés !" |
| `mobile/src/domains/narative/subscription/view/components/PrivacyChangeWarningBanner/index.ts` | Barrel | Re-exports |
| `mobile/src/domains/narative/subscription/view/containers/SubscriptionToggleRow/SubscriptionToggleRow.container.tsx` | Container | Fetches `subscriptionsEnabled` + `subscriberCount`; calls `toggleNarativeSubscriptions`; optimistic UI; passes `narativeVisibility` to component |
| `mobile/src/domains/narative/subscription/view/containers/SubscriptionToggleRow/index.ts` | Barrel | Re-exports |

## Files to Modify

| File Path | Change |
|-----------|--------|
| `mobile/src/app/(app)/narative/[id]/edit/index.tsx` (or the NarativeSetup screen — confirm exact path by finding where `confidentiality` dropdown / `UpdateNarativeInput` is rendered) | Add `<PrivacyChangeWarningBanner subscriberCount={subscriberCount} visible={showPrivacyWarning} />` below visibility dropdown; add `<SubscriptionToggleRowContainer narativeId={id} narativeVisibility={currentVisibility} />` in member-only settings section |

---

## Tasks

**SubscriptionToggleRow component (pure):**

- [x] Create `SubscriptionToggleRow.tsx` accepting `SubscriptionToggleRowProps` exactly as defined in §Component Props Interfaces above
- [x] If `narativeVisibility === 'private'`: return `null` immediately (D7) — no toggle rendered
- [x] If `narativeVisibility === 'public'`: render a toggle switch row with:
  - Toggle control wired to `onToggle(newValue)` callback
  - `enabled` prop drives the toggle state
  - Subscriber count label: e.g., "{subscriberCount} abonné(s)" visible when `enabled === true`
  - `loading` prop disables the toggle during pending mutation
- [x] Create `index.ts` barrel

**PrivacyChangeWarningBanner component (pure):**

- [x] Create `PrivacyChangeWarningBanner.tsx` accepting `PrivacyChangeWarningBannerProps`
- [x] If `visible === false`: return `null`
- [x] If `visible === true`: render an inline warning banner (NOT a modal, NOT a bottom sheet — D9) with text: "La confidentialité semble changer · Les {subscriberCount} abonnements existants seront supprimés !"
- [x] Banner is purely informational — no confirm/cancel buttons
- [x] Create `index.ts` barrel

**SubscriptionToggleRowContainer:**

- [x] Create `SubscriptionToggleRow.container.tsx` accepting `SubscriptionToggleRowContainerProps`
- [x] Fetch `subscriptionsEnabled` and compute `subscriberCount` for the narative (use `narativeSubscriptionService.getSubscriberCount` via a query, or derive from the narative query — confirm available data source)
- [x] Manage `toggleEnabled` local state (initialized from server `subscriptionsEnabled`)
- [x] Manage `subscriberCount` local state (initialized from server)
- [x] Implement optimistic toggle handler:
  - On disable: set `toggleEnabled = false` + `subscriberCount = 0` locally → call `useToggleNarativeSubscriptionsMutation({ narativeId, enabled: false })` → on error: revert both states
  - On enable: set `toggleEnabled = true` locally → call `useToggleNarativeSubscriptionsMutation({ narativeId, enabled: true })` → on error: revert
- [x] Pass `narativeVisibility` through to `SubscriptionToggleRow` component unchanged
- [x] Create `index.ts` barrel

**NarativeSetup integration:**

- [x] Find the NarativeSetup/edit screen file (search for `confidentiality` dropdown or `UpdateNarativeInput` — confirmed: `mobile/src/domains/narative/edit/view/containers/EditNarative/EditNarative.container.tsx`; confidentiality is in shared `NarativeForm` component)
- [x] Add local state: `showPrivacyWarning: boolean` — computed as `prevVisibility === 'public' && newVisibility === 'private' && subscriberCount > 0`
- [x] Add `<PrivacyChangeWarningBanner subscriberCount={subscriberCount} visible={showPrivacyWarning} />` below the visibility dropdown (not as a modal — D9)
- [x] Add `<SubscriptionToggleRowContainer narativeId={id} narativeVisibility={currentVisibility} />` in the member-only settings section (below the visibility section)

---

## Acceptance Criteria

- [x] `SubscriptionToggleRow` returns `null` when `narativeVisibility === 'private'` (no DOM/RN node rendered)
- [x] `SubscriptionToggleRow` shows the toggle when `narativeVisibility === 'public'`
- [x] Toggle disabled state disables the toggle control when `loading === true`
- [x] `PrivacyChangeWarningBanner` returns `null` when `visible === false`
- [x] `PrivacyChangeWarningBanner` renders the inline warning (not a modal) when `visible === true`
- [x] Warning text includes the subscriber count interpolated: "Les {N} abonnements existants seront supprimés !"
- [x] `SubscriptionToggleRowContainer` applies optimistic updates: toggle fires immediately, reverts on error
- [x] `subscriberCount` shows 0 optimistically when toggle is disabled (reverted on error)
- [x] In NarativeSetup: `PrivacyChangeWarningBanner` appears below the visibility dropdown when switching from public to private with `subscriberCount > 0`
- [x] In NarativeSetup: `SubscriptionToggleRowContainer` is hidden (returns null) when `narativeVisibility === 'private'`

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| Toggle disabled, mutation errors | Revert both `toggleEnabled` and `subscriberCount` to pre-tap values | Error toast, toggle reverts |
| `PrivacyChangeWarningBanner` with `subscriberCount === 0` | `showPrivacyWarning = false` — no banner | Transparent |
| `narativeVisibility === 'private'` | `SubscriptionToggleRow` returns `null` — no toggle rendered (D7) | Toggle absent from UI |
| Subscriber count when subscriptions disabled (optimistic) | Optimistic count = 0 on disable; server confirms; on error: revert both `enabled` and `count` | Count shows 0 immediately |

---

## Architectural Constraints

- `SubscriptionToggleRow` must return `null` (not a hidden/opacity-0 view) when `narativeVisibility === 'private'` (D7)
- `PrivacyChangeWarningBanner` must be an inline banner — NOT a modal, NOT a bottom sheet (D9)
- `PrivacyChangeWarningBanner` must have NO confirm/cancel buttons — purely informational (D9)
- Optimistic update for toggle must revert BOTH `toggleEnabled` AND `subscriberCount` on error
- Do NOT use `any` type anywhere

---

## Dev Notes

- Look at `mobile/src/app/(app)/narative/[id]/edit/index.tsx` for where `confidentiality` is managed — the `showPrivacyWarning` boolean belongs in that screen's local state
- The exact file path for NarativeSetup must be confirmed by searching for `UpdateNarativeInput` or the `confidentiality` field usage in the mobile codebase
- `useToggleNarativeSubscriptionsMutation` comes from the codegen output of S-07

---

## Completion Notes

**Files created:**
- `mobile/src/domains/narative/subscription/view/components/SubscriptionToggleRow/SubscriptionToggleRow.tsx`
- `mobile/src/domains/narative/subscription/view/components/SubscriptionToggleRow/index.ts`
- `mobile/src/domains/narative/subscription/view/components/PrivacyChangeWarningBanner/PrivacyChangeWarningBanner.tsx`
- `mobile/src/domains/narative/subscription/view/components/PrivacyChangeWarningBanner/index.ts`
- `mobile/src/domains/narative/subscription/view/containers/SubscriptionToggleRow/SubscriptionToggleRow.container.tsx`
- `mobile/src/domains/narative/subscription/view/containers/SubscriptionToggleRow/index.ts`

**Files modified:**
- `backend/src/graphql/narative/entities/narative-entry-screen.entity.ts` — added `subscriberCount: Int!` field
- `backend/src/graphql/narative/resolvers/narative-subscription.resolver.ts` — populate `subscriberCount: narative.likedBy?.length ?? 0`
- `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.gql` — added `subscriberCount` field
- `mobile/src/domains/narative/shared/view/components/NarativeForm/NarativeForm.component.tsx` — added `afterConfidentiality?: React.ReactNode` slot prop, rendered after the confidentiality `Select`
- `mobile/src/domains/narative/edit/view/containers/EditNarative/EditNarative.container.tsx` — added `showPrivacyWarning` logic + passes `afterConfidentiality` with `PrivacyChangeWarningBanner` + `SubscriptionToggleRowContainer`

**Deviations:**
- `subscriberCount` added to `NarativeEntryScreen` GQL entity (backend + query) because no other query exposed subscriber count; avoids a new dedicated endpoint
- NarativeSetup integration done via `afterConfidentiality` render slot in `NarativeForm` rather than modifying the route file — the confidentiality field is inside the shared `NarativeForm` component, not in the route file; the render slot pattern keeps the shared form agnostic while allowing the edit container to inject content
- `SubscriptionToggleRowContainer` uses `narativeEntryScreen` query (cache-first) for initial data — member users see `isMember=true` in the response but the data is valid; no redirect occurs because the container doesn't use the routing logic

**Escalations:** none
