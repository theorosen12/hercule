# Pendo Mobile SDK Integration — Deep Research

**Date:** 2026-03-10
**App:** Nara (React Native 0.81.5 + Expo SDK 54)
**Scope:** Analytics only (pas de guides in-app)
**Environnements:** 2 comptes Pendo (dev + prod)
**Backend:** Non impliqué
**RGPD:** Pas de contrainte spécifique

---

## SDK & Compatibilite

| Element | Valeur |
|---------|--------|
| **Package npm** | `rn-pendo-sdk` |
| **Derniere version** | 3.8.0 |
| **Expo supporte** | SDK 41-53 confirme, SDK 54 compatible (RN 0.81) |
| **New Architecture (Fabric)** | Supporte depuis v3.7.2 |
| **Expo Go** | **NON supporte** — necessite un EAS dev build |

> **Point d'attention :** La doc officielle mentionne "Expo SDK 41-53". Expo 54 devrait fonctionner avec `rn-pendo-sdk@3.8.0` (support Fabric), mais a verifier en dev build.

---

## Installation Step-by-Step

### 1. Installer le SDK

```bash
npx expo install rn-pendo-sdk
```

### 2. Configurer le plugin Expo

Dans `app.config.ts`, ajouter le plugin avec les scheme IDs (trouvables dans Pendo UI > Settings > Subscription Settings > App Details) :

```typescript
plugins: [
  [
    "rn-pendo-sdk",
    {
      "ios-scheme": "YOUR_IOS_SCHEME_ID",
      "android-scheme": "YOUR_ANDROID_SCHEME_ID"
    }
  ]
]
```

### 3. Multi-env (dev/prod)

Utiliser une variable d'environnement pour switcher l'app key :

```typescript
// config/pendo.ts
const PENDO_KEYS = {
  development: 'YOUR_DEV_APP_KEY',
  production: 'YOUR_PROD_APP_KEY',
};

export const PENDO_APP_KEY = PENDO_KEYS[process.env.EXPO_PUBLIC_ENV ?? 'development'];
```

### 4. Initialiser le SDK

Dans l'entry file (App.tsx ou equivalent) :

```typescript
import { PendoSDK, NavigationLibraryType } from 'rn-pendo-sdk';

const navigationOptions = {
  library: NavigationLibraryType.ReactNavigation,
};

const initPendo = async () => {
  await PendoSDK.setup(PENDO_APP_KEY, navigationOptions);
};
```

### 5. Wrapper le NavigationContainer

```typescript
import { WithPendoReactNavigation } from 'rn-pendo-sdk';
import { NavigationContainer } from '@react-navigation/native';

const PendoNavigationContainer = WithPendoReactNavigation(NavigationContainer);

// Dans le JSX, remplacer :
// <NavigationContainer>  -->  <PendoNavigationContainer>
```

### 6. Demarrer une session

Quand l'utilisateur est identifie (login, app open) :

```typescript
await PendoSDK.startSession(
  visitorId,    // string | null (null = anonymous)
  accountId,    // string | null
  visitorData,  // { key: value } metadata visiteur
  accountData   // { key: value } metadata account
);
```

---

## API Analytics Disponibles

| Methode | Usage |
|---------|-------|
| `PendoSDK.setup(key, navOptions)` | Initialisation du SDK |
| `PendoSDK.startSession(visitorId, accountId, visitorData, accountData)` | Demarrer une session |
| `PendoSDK.track('EventName', { props })` | Tracker un evenement custom |
| `PendoSDK.setVisitorData({ key: value })` | Mettre a jour les donnees visiteur |
| `PendoSDK.setAccountData({ key: value })` | Mettre a jour les donnees account |
| `PendoSDK.endSession()` | Terminer la session (logout) |
| `PendoSDK.setDebugMode(true)` | Activer le mode debug |

---

## Limitations & Gotchas

- **Expo Go ne fonctionne pas** — utiliser un EAS dev build obligatoirement
- **Screen tracking auto** — fonctionne via le wrapper `WithPendoReactNavigation`, pas besoin de tracker manuellement les screens
- **Scheme IDs vs App Keys** — les scheme IDs sont pour le "pairing mode" (tagging dans Pendo UI), differents des app keys utilises pour `setup()`
- **Compatibilite Expo 54** — non officiellement listee mais devrait fonctionner avec SDK 3.8.0 (support Fabric/New Architecture)

---

## Sources

- [Pendo Expo RN iOS Guide](https://github.com/pendo-io/pendo-mobile-sdk/blob/master/ios/pnddocs/expo_rn-ios.md)
- [Pendo RN API Documentation](https://github.com/pendo-io/pendo-mobile-sdk/blob/master/api-documentation/rn-apis.md)
- [Pendo RN Demo App (React Navigation)](https://github.com/pendo-io/RN-demo-app-React-Navigation)
- [Pendo Mobile SDK Developer Guide](https://support.pendo.io/hc/en-us/articles/16180858202651)
- [Pendo Supported Frameworks](https://support.pendo.io/hc/en-us/articles/360031861572)
- [Pendo Track Events](https://support.pendo.io/hc/en-us/articles/360032294291-Configure-Track-Events)

---

## Next Steps

- [ ] Creer un epic brownfield pour l'integration (2-3 stories)
- [ ] Verifier compatibilite SDK 3.8.0 + Expo 54 en dev build
- [ ] Recuperer les app keys et scheme IDs des 2 comptes Pendo (dev + prod)
