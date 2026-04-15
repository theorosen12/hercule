# Linear Context — Bug Deeplink Post-Login Redirect

## Source
- Issue: NAR-459 — Bug — Deeplink post-login ne redirige pas vers NarativeEntryScreen
- Status: Backlog | Priority: High
- Assignee(s): matthieu.gouverith@gmail.com
- Labels: Bug

## Epic / Project
- Epic: NAR-448 — Narative Subscription — Epic
- Project: Narative-subscription

## Feature Description

Quand un utilisateur **non connecté** clique sur un lien de partage narative, se connecte, il atterrit sur l'écran d'accueil (`/`) au lieu de `NarativeEntryScreen`.

## Comportement observé

1. Utilisateur reçoit un lien `nara-app.com/narative/[id]`
2. L'utilisateur n'est pas connecté
3. Il clique sur le lien → redirigé vers `/welcome`
4. Il se connecte
5. Il arrive sur `/` (home) au lieu de `/narative/[id]/entry` ❌

## Comportement attendu

Après login, l'utilisateur doit être redirigé vers `NarativeEntryScreen` (`/narative/[id]/entry`).

## Root Cause (identifié par analyse codebase)

Dans `mobile/src/app/_layout.tsx`, le deeplink `/narative/[id]` est immédiatement converti en `/narative/[id]/entry` via `resolveHref()` et `router.push()` est appelé.

Le `(app)/_layout.tsx` détecte que l'utilisateur n'est pas authentifié → redirect vers `/welcome`. Le narative ID est **perdu** — il n'est jamais stocké dans `TARGET_NARATIVE_ID`.

La fonction `navigateAfterAuth()` dans `navigation.helper.ts` **existe et est fonctionnelle**, mais ne se déclenche jamais car `TARGET_NARATIVE_ID` n'a jamais été renseigné.

### Flow actuel (cassé)
```
deeplink /narative/123
  → _layout.tsx: resolveHref() → /narative/123/entry
  → router.push('/narative/123/entry')
  → (app)/_layout.tsx: !isAuthenticated → Redirect href="/welcome"
  → narative ID perdu ❌
  → user login → navigateAfterAuth() → TARGET_NARATIVE_ID vide → push('/') ❌
```

### Flow attendu (à implémenter)
```
deeplink /narative/123
  → _layout.tsx: resolveHref() → /narative/123/entry
  → !isAuthenticated → StorageService.setString(TARGET_NARATIVE_ID, '123')
  → router.push('/narative/123/preview') ou '/welcome'
  → user login → navigateAfterAuth() → TARGET_NARATIVE_ID = '123' → push('/narative/123/entry') ✅
```

## Key Decisions & Constraints

- La constante `TARGET_NARATIVE_ID` existe déjà dans `mobile/src/shared/constants/localStorage.constant.ts`
- `navigateAfterAuth()` dans `mobile/src/domains/auth/shared/helpers/navigation.helper.ts` est déjà implémentée et lit `TARGET_NARATIVE_ID` — pas besoin de la modifier
- Le fix est **minimal et localisé** : uniquement `mobile/src/app/_layout.tsx`
- Il faut couvrir les deux cas : login + signup (création de compte)
- Zéro régression pour les utilisateurs déjà connectés

## Open Questions

- Faut-il router l'utilisateur non-auth vers `/narative/[id]/preview` (expérience enrichie) ou directement vers `/welcome` (plus simple) avant le login ?
- Le cas "création de compte" est-il déjà couvert par `navigateAfterAuth()` ou nécessite-t-il un hook séparé ?

## Linked Resources

- Linear Project: https://linear.app/naraa/project/narative-subscription-652a79e379c0/overview
- Issue: https://linear.app/naraa/issue/NAR-459/bug-deeplink-post-login-ne-redirige-pas-vers-narativeentryscreen

## Fichiers clés

| Fichier | Rôle | Action |
|---------|------|--------|
| `mobile/src/app/_layout.tsx` | Deeplink processing (lignes 46-55) | **À modifier** — stocker TARGET_NARATIVE_ID si !auth |
| `mobile/src/app/(app)/_layout.tsx` | Auth gate | Lecture seule |
| `mobile/src/domains/auth/shared/helpers/navigation.helper.ts` | `navigateAfterAuth()` | Fonctionnel, pas de modification |
| `mobile/src/shared/constants/localStorage.constant.ts` | `TARGET_NARATIVE_ID` constant | Déjà définie |
| `mobile/src/app/narative/[id]/preview.tsx` | Route preview unauthenticated | Destination potentielle |

## Sibling Issues (même epic NAR-448)

| Issue ID | Title | Status |
|----------|-------|--------|
| NAR-449 | S-01 — Prisma Schema Migration | — |
| NAR-450 | S-02 — Narative Repository Extensions | — |
| NAR-451 | S-03 — Access & Subscription Services | — |
| NAR-452 | S-04 — Entry Screen Entity Resolver | — |
| NAR-453 | S-05 — Membership Request Auto-Accept | — |
| NAR-454 | S-06 — Narative Service Resolver Integration | — |
| NAR-455 | S-08 — Mobile Navigation Service + Entry Route (D16) | Pull request |
| NAR-456 | S-09 — NarativeEntryScreen UI + Subscribe/Membership Actions | Pull request |
| NAR-457 | S-10 — SubscriptionToggleRow + PrivacyChangeWarningBanner | Pull request |

## Acceptance Criteria

- [ ] Un utilisateur non connecté qui clique sur un lien narative, puis se connecte, est redirigé vers `NarativeEntryScreen` du bon narative
- [ ] Un utilisateur non connecté qui clique sur un lien narative, puis **crée un compte**, est redirigé vers `NarativeEntryScreen`
- [ ] Un utilisateur déjà connecté qui clique sur un lien narative est toujours redirigé directement vers `NarativeEntryScreen` (pas de régression)
- [ ] Si le narative ID est invalide / narative supprimé, le fallback est `/` (home) sans crash
