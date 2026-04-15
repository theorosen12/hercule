# S-08 — Mobile Navigation Service + Entry Route (D16)

**Epic:** narative-subscription
**Status:** Done
**Priority:** High
**Layer:** Mobile
**Depends on:** S-07

---

## Goal

Create `narativeEntryNavigation.service.ts` (the universal navigation function), the `/(app)/narative/[id]/entry.tsx` Expo Router route, and modify `_layout.tsx` to register the new screen — implementing the D16 navigation decision gate in `NarativeEntry.container.tsx`.

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D16 | **`/(app)/narative/[id]/entry` is the universal entry point** — all external navigation (notifications, share links, deep links) always targets `/entry`, never directly `/(app)/narative/[id]`. The `NarativeEntryContainer` is the access gate: queries `narativeEntryScreen`, renders nothing while loading, then either redirects (member / active subscriber) or displays the entry screen. Prevents flash-of-wrong-content. Single decision point — no caller needs to know the user's relationship before navigating. |
| D13 | **No paste link / deep link flow** — out of scope. Dedicated project: Deep Link — Paste a Nara Link. |

### Navigation

| Route | Screen | Params | Entry Points |
|-------|--------|--------|--------------|
| `/(app)/narative/[id]/entry` | `NarativeEntryContainer` | `{ id: string }` | Notification handlers; auth redirect after login; any external narativeId reference |

### Navigation Service (exact)

```typescript
// mobile/src/domains/narative/subscription/infra/services/narativeEntryNavigation.service.ts

export function navigateToNarativeEntry(
  router: Router,
  narativeId: string,
): void {
  router.push({
    pathname: '/(app)/narative/[id]/entry',
    params: { id: narativeId },
  });
}
// This is the ONLY function used to navigate toward a narative from outside.
// No caller inspects isMember / isSubscribed before calling this.
// The NarativeEntryContainer owns the access decision.
```

### Navigation Decision Logic — D16

```
External trigger (notification / share link / deep link)
        │
        ▼
navigate to /(app)/narative/[id]/entry
        │
        ▼
NarativeEntryContainer mounts
  → fires narativeEntryScreen query (fetchPolicy: 'network-only')
  → renders <LoadingSpinner /> (no entry screen UI)
        │
        ▼
Query resolves
  ┌─────────────────────────────────────────────────────────┐
  │ isMember === true                                       │
  │   → router.replace('/(app)/narative/[id]')             │
  ├─────────────────────────────────────────────────────────┤
  │ isAlreadySubscribed === true                            │
  │   && subscriptionsEnabled === true                      │
  │   && confidentiality === 'public'                       │
  │   → router.replace('/(app)/narative/[id]')             │
  ├─────────────────────────────────────────────────────────┤
  │ narativeEntryScreen === null (not found)                │
  │   → render error state + retry                         │
  ├─────────────────────────────────────────────────────────┤
  │ else (non-member, non-subscriber, or access revoked)    │
  │   → render NarativeEntryScreen                         │
  └─────────────────────────────────────────────────────────┘
```

> **Key invariant:** The `NarativeEntryScreen` UI component is **never rendered during the loading phase** — only after the query confirms the user has no current access.

### Container Props Interface

```typescript
interface NarativeEntryContainerProps {
  narativeId: string;
}
```

---

## Files to Create

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/narative/subscription/infra/services/narativeEntryNavigation.service.ts` | Service | `navigateToNarativeEntry(router, narativeId)` — single exported function |
| `mobile/src/app/(app)/narative/[id]/entry.tsx` | Expo Router route | Reads `id` from `useLocalSearchParams`; renders `NarativeEntryContainer` — no other logic |

## Files to Modify

| File Path | Change |
|-----------|--------|
| `mobile/src/app/(app)/narative/[id]/_layout.tsx` | Add `<Stack.Screen name="entry" options={{ headerShown: false }} />` |

---

## Tasks

- [x] Create `mobile/src/domains/narative/subscription/infra/services/narativeEntryNavigation.service.ts` with the exact exported function from §Navigation Service above — no class, just an exported function
- [x] Create `mobile/src/app/(app)/narative/[id]/entry.tsx` as an Expo Router route file:
  - Use `useLocalSearchParams<{ id: string }>()` to extract the `narativeId`
  - Render `<NarativeEntryContainer narativeId={id} />` — nothing else
  - No logic in the route file itself
- [x] Open `mobile/src/app/(app)/narative/[id]/_layout.tsx` and add `<Stack.Screen name="entry" options={{ headerShown: false }} />` to the Stack navigator
- [x] Create `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/NarativeEntry.container.tsx` with `NarativeEntryContainer`:
  - Accept `NarativeEntryContainerProps` (`{ narativeId: string }`)
  - Call `useGetNarativeEntryScreenQuery({ variables: { narativeId }, fetchPolicy: 'network-only' })`
  - While `loading === true`: render `<Loader />` — do NOT render `NarativeEntryScreen`
  - When `data?.narativeEntryScreen === null`: render `<ApiResult error={...} onRetry={refetch} />`
  - When `isMember === true` OR `isAlreadySubscribed && subscriptionsEnabled && public`: `router.replace` in `useEffect`
  - Otherwise: `null` placeholder (entry screen wired in S-09)
- [x] Create `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/index.ts` barrel re-exporting `NarativeEntryContainer`

---

## Acceptance Criteria

- [x] `navigateToNarativeEntry(router, narativeId)` navigates to `/(app)/narative/[id]/entry` with `{ id: narativeId }` params
- [x] `entry.tsx` renders nothing but `<NarativeEntryContainer narativeId={id} />`
- [x] `_layout.tsx` registers `entry` screen with `headerShown: false`
- [x] `NarativeEntryContainer`: while query is loading, only `<Loader />` is visible — `NarativeEntryScreen` is NOT rendered
- [x] `NarativeEntryContainer`: when `isMember === true` → calls `router.replace` toward `/narative/${narativeId}`
- [x] `NarativeEntryContainer`: when `isAlreadySubscribed && subscriptionsEnabled && confidentiality === 'public'` → calls `router.replace`
- [x] `NarativeEntryContainer`: when `narativeEntryScreen === null` → renders error state with retry
- [x] `NarativeEntryContainer`: when none of the above conditions → renders entry screen (wired in S-09)

---

## Edge Cases & Required Handling

| Edge Case | Handling Strategy | User-Facing |
|-----------|-------------------|-------------|
| User is already a member | `isMember === true` → `router.replace('/(app)/narative/[id]')` | Transparent redirect, no flash |
| User is already subscribed with access | `isAlreadySubscribed && subscriptionsEnabled && confidentiality === 'public'` → `router.replace(...)` | Transparent redirect |
| `narativeEntryScreen === null` (not found) | Render error state + retry button | Error state |
| Query loading | Render `<LoadingSpinner />` only — `NarativeEntryScreen` never rendered during load | Spinner |

---

## Architectural Constraints

- `navigateToNarativeEntry()` must be the ONLY navigation function used to reach the entry route — callers must not construct the pathname directly
- The `NarativeEntryScreen` UI component must NEVER be rendered while `loading === true` (D16 — key invariant)
- `entry.tsx` must contain ZERO logic beyond reading `useLocalSearchParams` and rendering the container
- Do NOT add deep link or paste link handling (D13 — out of scope)
- Do NOT use `any` type anywhere

---

## Dev Notes

- The `router.replace()` redirect belongs in a `useEffect` with `[data]` dependency — do not call router methods directly during render
- `useRouter()` from `expo-router` provides the `router` object in the container
- Follow the existing container pattern in `mobile/src/domains/narative/page/` for hook usage and state management structure
- Unit tests for the redirect `useEffect` are specified in architecture §12: "`isMember=true` → `router.replace`; `isAlreadySubscribed=true` + `subscriptionsEnabled=true` → `router.replace`; neither → no redirect" — implement all 3

---

## Completion Notes

**Files created:** `mobile/src/domains/narative/subscription/infra/services/narativeEntryNavigation.service.ts`, `mobile/src/app/(app)/narative/[id]/entry.tsx`, `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/NarativeEntry.container.tsx`, `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/index.ts`
**Files modified:** `mobile/src/app/(app)/narative/[id]/_layout.tsx`
**Deviations:** `router.replace` uses string path `/narative/${narativeId}` (project convention) rather than object form — both resolve identically in Expo Router. Container returns `null` as placeholder for S-09 entry screen.
**Escalations:** none
