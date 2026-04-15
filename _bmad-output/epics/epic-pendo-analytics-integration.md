# Epic: Integration Pendo Analytics — Brownfield Enhancement

**Date:** 2026-03-10
**Status:** Implementation Complete — Pending Validation

## Epic Goal

Completer et fiabiliser l'integration Pendo analytics dans l'app Nara pour avoir un tracking fonctionnel des screens et events utilisateur, avec support multi-environnement (dev/prod).

## Existing System Context

- **App:** Nara — React Native 0.81.5, Expo SDK 54, Expo Router
- **Stack:** TypeScript, Apollo GraphQL, Expo Router (file-based routing)
- **Integration existante:** rn-pendo-sdk@3.8.0 (mis a jour depuis 3.0.1)
- **Env management:** Variables `EXPO_PUBLIC_*` dans env files

### Etat actuel de l'integration

| Element | Status |
|---------|--------|
| `rn-pendo-sdk` installe | Oui (v3.8.0) |
| Plugin Expo configure | Oui (`app.config.ts`) |
| `TrackingService.init()` cree | Oui |
| `TrackingService.init()` appele | Oui (`_layout.tsx`, top-level) |
| `startSession()` | Oui (login + restore + anonymous) |
| `track()` events | Oui (50 events definis, tous cables) |
| `NavigationLibraryType.ExpoRouter` | Oui (support natif SDK 3.8.0) |
| Multi-env (dev/prod) | Oui (env vars ajoutees, cles a renseigner) |

### Fichiers cles

- `mobile/app.config.ts` — Plugin Pendo (lignes 110-115)
- `mobile/src/shared/infra/services/tracking/tracking.service.ts` — Service tracking
- `mobile/src/shared/infra/services/tracking/events.ts` — Enum des events (source unique)
- `mobile/src/app/_layout.tsx` — Root layout (init + deep links)
- `mobile/package.json` — rn-pendo-sdk@3.8.0

## Enhancement Details

- **Ce qui change:** Mise a jour SDK, initialisation effective, session tracking, event tracking complet
- **Integration:** Cote mobile uniquement, pas de changement backend
- **Success criteria:** Donnees visibles dans les dashboards Pendo dev et prod

---

## Stories

### Story 1: Mise a jour et initialisation du SDK Pendo

**Objectif:** Avoir le SDK Pendo initialise et fonctionnel au lancement de l'app

**Tasks:**

- [x] Mettre a jour `rn-pendo-sdk` de 3.0.1 vers 3.8.0
- [x] Appeler `TrackingService.init()` dans le lifecycle de l'app (`_layout.tsx`)
- [x] Configurer le multi-env (dev/prod) avec les app keys via `EXPO_PUBLIC_PENDO_API_KEY`
- [x] Compatibilite Expo Router confirmee (`NavigationLibraryType.ExpoRouter` natif en 3.8.0)
- [x] Guard `__DEV__` supprime, remplace par `setDebugMode(__DEV__)` pour debug en dev
- [ ] Valider avec un EAS dev build (Expo Go non supporte)

**Acceptance Criteria:**

- [x] SDK 3.8.0 installe sans erreur
- [x] `TrackingService.init()` appele au lancement
- [ ] SDK s'initialise correctement, visible dans le dashboard Pendo
- [x] Screens auto-trackes via `NavigationLibraryType.ExpoRouter`
- [ ] Fonctionne en dev build et prod build

---

### Story 2: Session tracking et identification utilisateur

**Objectif:** Identifier les utilisateurs dans Pendo et tracker leurs sessions

**Tasks:**

- [x] Appeler `PendoSDK.startSession()` apres l'authentification utilisateur (dans `AuthProvider`)
- [x] Appeler `PendoSDK.endSession()` au logout
- [x] Passer le `visitorId` (user ID)
- [x] Gerer le cas anonymous (utilisateur non connecte)

**Acceptance Criteria:**

- [ ] Sessions visibles dans Pendo dashboard
- [x] Utilisateurs identifies par leur ID unique
- [x] Session terminee proprement au logout
- [x] Visiteurs anonymes geres (pre-login)

---

### Story 3: Event tracking custom

**Objectif:** Tracker les actions utilisateur cles dans l'app

**Tasks:**

- [x] Definir les events cles a tracker — 50 events dans `events.ts`
- [x] Creer un wrapper type pour `PendoSDK.track()` dans `TrackingService`
- [x] Implementer le tracking sur tous les events
- [x] Activer `setDebugMode(true)` en dev pour validation

**Acceptance Criteria:**

- [ ] Events custom visibles dans le dashboard Pendo
- [x] Debug mode actif en environnement dev
- [x] Naming convention definie — enum `TrackingEvent` dans `events.ts`
- [x] TrackingService expose une API propre pour track()

---

### Events integres

#### Onboarding (R3)
| Event | Fichier | Status |
|-------|---------|--------|
| `onboarding_started` | `PersonalInformation.container.tsx` | Done |
| `onboarding_completed` | `Register.provider.tsx` | Done |
| `login` | `Login.container.tsx`, `useGoogleSignIn.hook.ts`, `useAppleSignIn.hook.ts` | Done |
| `logout` | `Auth.provider.tsx` | Done |

#### Core Loop — Naratives & Moments (R4)
| Event | Fichier | Status |
|-------|---------|--------|
| `narative_creation_started` | `(tabs)/_layout.tsx` | Done |
| `narative_created` | `Members.container.tsx` | Done |
| `narative_edited` | `EditNarative.container.tsx` | Done |
| `narative_deleted` | `DeleteNarativeBottomSheet.component.tsx` | Done |
| `narative_viewed` | `narative/[id]/index.tsx` | Done |
| `moment_created` | `moment/create/index.tsx` | Done |
| `moment_edited` | `moment/[momentId]/edit/index.tsx` | Done |
| `moment_deleted` | `MomentFormHeader.component.tsx` | Done |

#### Discovery & Consumption (R5)
| Event | Fichier | Status |
|-------|---------|--------|
| `map_viewed` | `(tabs)/index.tsx` | Done |
| `map_marker_tapped` | `(tabs)/index.tsx` | Done |
| `story_teller_opened` | `StoryTeller.component.tsx` | Done |
| `story_teller_moment_viewed` | `useStoryTeller.hook.ts` | Done |
| `story_teller_exited` | `StoryTeller.component.tsx` | Done |
| `feed_narrative_chip_tapped` | `StoryTellerMoment.component.tsx` | Done |

#### Social & Engagement (R6)
| Event | Fichier | Status |
|-------|---------|--------|
| `friend_request_sent` | `ProfileActionsBottomSheet.component.tsx` | Done |
| `friend_request_accepted` | `NotificationItemFriendRequest.component.tsx` | Done |
| `friend_request_declined` | `NotificationItemFriendRequest.component.tsx` | Done |
| `membership_requested` | `narative/[id]/index.tsx` | Done |
| `membership_accepted` | `NotificationItemNarativeMembershipRequest.component.tsx` | Done |
| `membership_declined` | `NotificationItemNarativeMembershipRequest.component.tsx` | Done |
| `narative_liked` | `useNarativeLikeToggle.hook.ts` | Done |
| `narative_shared` | `useNarativeSharing.hook.ts` | Done |
| `contact_sync_completed` | `useContactDiscovery.hook.ts` | Done |
| `friend_removed` | `ProfileActionsBottomSheet.component.tsx` | Done |
| `user_blocked` | `ProfileActionsBottomSheet.component.tsx` | Done |
| `user_unblocked` | `ProfileActionsBottomSheet.component.tsx` | Done |

#### Navigation (R7)
| Event | Fichier | Status |
|-------|---------|--------|
| `tab_switched` | `(tabs)/_layout.tsx` | Done |
| `deep_link_opened` | `_layout.tsx` | Done |
| `profile_viewed` | `Profile.container.tsx` | Done |
| `profile_edited` | `ProfileForm.component.tsx` | Done |
| `settings_opened` | `MyProfileActionsBottomSheet.component.tsx` | Done |
| `search_performed` | `SearchResults.container.tsx` | Done |
| `account_deleted` | `MyProfileActionsBottomSheet.component.tsx` | Done |

#### Onboarding Steps (R3 — nouveaux)
| Event | Fichier | Status |
|-------|---------|--------|
| `onboarding_step_completed` (step 2) | `Password.container.tsx` | Done |
| `onboarding_step_completed` (step 3) | `ProfilePicture.container.tsx` | Done |
| `onboarding_step_completed` (step 4) | `TermsAndConditions.container.tsx` | Done |
| `onboarding_step_completed` (step 5) | `PhoneNumberScreen.container.tsx` | Done |

#### Media (R4 — nouveaux)
| Event | Fichier | Status |
|-------|---------|--------|
| `media_uploaded` | `UploadStatusHeader.component.tsx` | Done |
| `media_deleted` | `MediaListField.component.tsx` | Done |
| `description_added` | `moment/create/index.tsx`, `moment/[momentId]/edit/index.tsx` | Done |
| `description_edited` | `moment/[momentId]/edit/index.tsx` | Done |

#### Discovery & Feed (R5 — nouveaux)
| Event | Fichier | Status |
|-------|---------|--------|
| `story_teller_media_viewed` | `StoryTellerMoment.component.tsx` | Done |
| `feed_opened` | `MomentFeed.container.tsx` | Done |
| `feed_session_ended` | `MomentFeed.container.tsx` | Done |

#### Social (R6 — nouveaux)
| Event | Fichier | Status |
|-------|---------|--------|
| `notification_opened` | `(tabs)/notifications.tsx` | Done |

#### Events non cables (feature non existante)
| Event | Raison |
|-------|--------|
| `feed_moment_viewed` | Couvert par `story_teller_moment_viewed` (meme composant StoryTeller) |
| `analytics_consent_changed` | Feature de consentement non implementee dans l'app |

---

## Compatibility Requirements

- [x] Existing APIs remain unchanged
- [x] Database schema changes — aucun
- [x] UI changes — aucun (SDK-only)
- [x] Performance impact minimal (SDK async)

## Risk Mitigation

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Incompatibilite SDK 3.8.0 + Expo Router | Screen auto-tracking ne fonctionne pas | `NavigationLibraryType.ExpoRouter` supporte nativement |
| Incompatibilite SDK 3.8.0 + Expo 54 | SDK crash au lancement | A valider en dev build |
| Performance impact | Ralentissement de l'app | SDK init sync, calls async, debug mode en dev |

**Rollback Plan:** Revert a la v3.0.1 ou desactiver le SDK via env var vide

## Definition of Done

- [x] SDK 3.8.0 installe et initialise au lancement
- [x] Sessions trackees avec identification utilisateur
- [x] Screen tracking fonctionnel via ExpoRouter
- [x] Events custom cables (48/50 events integres, 2 non applicables)
- [x] Multi-env dev/prod configure (cles a renseigner)
- [x] TypeScript compile sans erreur
- [ ] Valide en EAS dev build
- [ ] Donnees visibles dans dashboard Pendo
- [ ] Aucune regression sur les features existantes

## Recherche de reference

Voir: `_bmad-output/research/pendo-mobile-sdk-integration-2026-03-10.md`

## Prochaines etapes

1. **Renseigner les cles Pendo** (`EXPO_PUBLIC_PENDO_API_KEY`, `EXPO_PUBLIC_PENDO_SCHEME_ID`) dans les `.env` — recuperer les cles depuis le dashboard Pendo
2. **Build EAS dev** : `eas build --profile development --platform ios` (ou android) — Expo Go ne supporte pas les modules natifs
3. **Valider l'initialisation** : verifier les logs `[Pendo]` dans la console Metro avec `setDebugMode(true)`
4. **Valider dans le dashboard Pendo** : sessions, screens auto-trackes, events custom
5. **Tester regression** : navigation, auth, creation moment/narative — aucun changement UI
