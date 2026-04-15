# Linear Context — Narative Subscription

## Source
- Issue: NAR-388 — Narative Subscription — Epic
- Status: Backlog | Priority: High
- Assignee(s): Matthieu Gouverith
- Labels: Feature, v2.6
- Project: Narative-subscription (In Progress)
- Initiatives: "Improve user onboarding on Nara" · "Improve user engagement and retention"

## Feature Description

Le lien de partage Nara drop actuellement les non-membres sur un écran blanc. **Narative Subscription** transforme ce cul-de-sac en portail d'accueil via la **Narative Entry Screen**.

N'importe quel utilisateur recevant un lien de partage peut s'abonner en un tap (via le bouton like existant) et accéder à l'historique complet du narative en mode lecture seule. Les membres partagent leurs souvenirs sans compromettre l'intimité du groupe.

**Mécanisme central :** Subscribe = Like. La relation `likedBy` est sémantiquement réutilisée comme liste d'abonnés. Pas de nouvelle join table.

### Scope v0 MVP
- Narative Entry Screen (landing non-membre)
- Abonnement via like button (un tap, UI optimiste)
- Accès lecture seule complet pour les abonnés
- Écran "Coller un lien nara" post-onboarding (flow manuel uniquement)
- Toggle d'abonnement dans les paramètres du narative (contrôle membre)
- Révocation : désactiver → tous les abonnés perdent l'accès immédiatement
- Migration : likes existants → abonnements + notification in-app

### Success Metrics
- Taux de conversion lien de partage → abonnement mesurable via Pendo (baseline à établir)
- Migration 100% des likes existants → abonnements au launch sans incident
- Zéro régression sur les flows existants (membre, ami d'un membre)

## Key Decisions & Constraints

- **Subscribe = Like** : `likedBy` est la liste d'abonnés — pas de nouvelle table
- **`NarativeAccessService` centralisé** : toute la logique d'accès passe par ce service
- **`subscriptionsEnabled` sur le modèle `Narative`** : flag booléen `@default(true)`
- **Révocation immédiate** : disable → `likedBy.deleteMany` immédiat, pas de grace period
- **Migration idempotente** : guard via `SystemConfig` key `SUBSCRIPTION_MIGRATION_COMPLETED`
- **Pas de deferred deep link automatique** : Universal Links / App Links post-install App Store non maîtrisé — flow manuel "Coller un lien nara" uniquement
- **Accès read-only strict** : le subscriber voit le narative complet sans contrôles d'édition
- **Statuts exclusifs** : abonné promu membre → auto-retiré de `likedBy`

## Scénarios d'entrée (décisions actées)

| Situation | "S'abonner" | "Demander l'ajout" |
|-----------|-------------|-------------------|
| Sans compte → crée compte → colle lien | → Narative Page | → bouton change état, notification à l'acceptation |
| Connecté + ami avec un membre | → Narative Page | → Narative Page (auto-accepté) |
| Non connecté + narative public + ami avec un membre | → Narative Page | → Narative Page (auto-accepté) |
| Non connecté + narative privé + ami avec un membre | → Narative Page | → bouton change état, notification à l'acceptation |
| Connecté ou non + aucun ami parmi les membres | → Narative Page | → bouton change état, notification à l'acceptation |

> L'auto-accept s'applique uniquement quand l'utilisateur est déjà connecté OU sur un narative public, et est ami avec au moins un membre. Le narative privé impose une validation explicite même pour les amis.

## Linked Resources
- PRD : `https://linear.app/naraa/document/prd-narative-subscription-dbcac53eaf9f`
- UX Spec : `https://linear.app/naraa/document/ux-spec-narative-subscription-25ccf8532d0c`
- Architecture : `https://linear.app/naraa/document/architecture-narative-subscription-v2-mobile-rewrite-2aaece3428e2`
- Scénarios : `https://linear.app/naraa/document/scenarios-90ab81716167`
