# Epic 5: Soft Update App Mobile — Brownfield Enhancement

## Epic Goal

Créer une query dédiée `checkAppVersion` dans le module `SystemConfig` : le mobile envoie sa version, le backend valide et retourne un boolean, le mobile affiche un modal de soft update si nécessaire. La query `getUser` n'est pas modifiée.

## Epic Description

**Existing System Context:**

- Query existante : `getUser($id: ID!)` dans `CurrentUserProvider.tsx` — **non modifiée**
- Backend : `SystemConfigModule` (nouveau) — NestJS/GraphQL
- Stack : React Native/Expo + NestJS/Prisma/PostgreSQL

**Flux cible :**

```
Mobile (lancement)                    Backend
       │                                │
       │── checkAppVersion(appVersion) ─→│ lit recommendedVersion depuis table SystemConfig
       │                                │  compare appVersion vs recommendedVersion (semver)
       │                                │
       │←── { updateAvailable: true } ──│
       │                                │
       │ si true → affiche modal        │
```

**Enhancement Details:**

- Le mobile appelle une query dédiée `checkAppVersion(appVersion: String!)` au lancement
- Le backend lit la version minimale recommandée depuis une table dédiée `SystemConfig` en DB
- Le backend compare `appVersion` vs `recommendedVersion` en semver
- Le backend retourne `updateAvailable: Boolean` via un type dédié `AppVersionCheck`
- Le mobile affiche un modal non-bloquant si `updateAvailable === true`
- La query `getUser` reste inchangée — séparation des responsabilités

---

## Stories

### Story 5.1: Backend — Query `checkAppVersion` dans le module `SystemConfig`

**As a** mobile app,
**I want** to call a dedicated query with my version and receive a boolean indicating if an update is available,
**so that** the app can prompt the user to update.

**Acceptance Criteria:**

1. Créer une nouvelle table Prisma `SystemConfig` dédiée aux paramètres système :
   ```prisma
   model SystemConfig {
     id        String   @id @default(cuid())
     key       String   @unique
     value     String
     createdAt DateTime @default(now())
     updatedAt DateTime @updatedAt
   }
   ```
2. Migration Prisma pour créer la table + seed avec une entrée `key: "recommendedAppVersion"`, `value: "1.0.0"`
3. Créer un module `SystemConfigModule` (service + resolver) pour gérer les paramètres système
4. Créer un type GraphQL `AppVersionCheck` :
   ```graphql
   type AppVersionCheck {
     updateAvailable: Boolean!
   }
   ```
5. Créer une query `checkAppVersion(appVersion: String!): AppVersionCheck!` dans `SystemConfigResolver`
6. Le `SystemConfigService` lit la valeur `recommendedAppVersion` depuis `SystemConfig`
7. Comparer `appVersion` vs `recommendedAppVersion` (semver : `appVersion < recommended` → `true`)
8. Si aucune entrée `recommendedAppVersion` en DB → `updateAvailable` retourne `false`
9. Tests unitaires : version inférieure → `true`, version égale/supérieure → `false`, entrée DB absente → `false`

---

### Story 5.2: Mobile — Appel `checkAppVersion` + modal de soft update

**As a** user opening the app,
**I want** to be notified if a newer version is available,
**so that** I can update from the store.

**Acceptance Criteria:**

1. Créer `checkAppVersion.query.gql` :
   ```graphql
   query checkAppVersion($appVersion: String!) {
     checkAppVersion(appVersion: $appVersion) {
       updateAvailable
     }
   }
   ```
2. Run `npm run codegen` après création du `.gql`
3. Appeler `checkAppVersion` au lancement de l'app (dans `CurrentUserProvider` ou composant dédié), en envoyant la version via `expo-constants` (`Constants.expoConfig.version`)
4. Si `updateAvailable === true` → afficher un modal non-bloquant `AppUpdateModal` :
   - Titre : "Mise à jour disponible"
   - Message : "Une nouvelle version est disponible. Mettez à jour pour profiter des dernières améliorations."
   - Bouton primaire : "Mettre à jour" → `Linking.openURL()` vers le store selon `Platform.OS` (URLs hardcodées côté mobile)
   - Bouton secondaire : "Plus tard" → ferme le modal
5. Le modal s'affiche **une seule fois par session**
6. Si la query échoue ou `updateAvailable` est `null`/`false` → rien ne s'affiche
7. Suivre les patterns UI existants (Emotion, design system)

---

## Compatibility Requirements

- [x] Query `getUser` **non modifiée** — zéro risque de régression
- [x] Schema DB : nouvelle table `SystemConfig` (clé/valeur)
- [x] UI suit les patterns existants
- [x] Query séparée — séparation des responsabilités claire

## Risk Mitigation

- **Risque principal :** Entrée `recommendedAppVersion` absente en DB
- **Mitigation :** Retourne `false` par défaut — jamais de faux positif
- **Avantage :** La version recommandée est modifiable en DB sans redéploiement
- **Rollback :** Retirer l'appel `checkAppVersion` côté mobile — aucun impact sur `getUser`

## Definition of Done

- [ ] Query `checkAppVersion` retourne `updateAvailable`
- [ ] Table `SystemConfig` créée avec entrée `recommendedAppVersion`
- [ ] Comparaison semver correcte côté backend
- [ ] Modal affiché quand `updateAvailable === true`
- [ ] Bouton store ouvre la bonne URL selon plateforme
- [ ] Fail silently si erreur
- [ ] Tests unitaires backend
- [ ] Aucune régression sur `getUser`
