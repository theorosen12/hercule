---
stepsCompleted: [step-01-init, step-02-discovery, step-02b-vision, step-02c-executive-summary, step-03-success, step-04-journeys, step-05-domain, step-06-innovation, step-07-project-type, step-08-scoping, step-09-functional, step-10-nonfunctional, step-11-polish]
inputDocuments:
  - _bmad-output/features/narative-subscription/context.md
  - _bmad-output/features/narative-subscription/brainstorm.md
  - _bmad-output/features/narative-subscription/data-brief.md
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/product-brief-DEVZONE-2026-03-14.md
workflowType: 'prd'
classification:
  projectType: Mobile + Backend feature
  domain: Consumer Social / Growth & Retention
  complexity: Medium-High
  projectContext: brownfield
---

# Product Requirements Document — Narative Subscription

**Author:** Matthieu
**Date:** 2026-04-10 (révisé — brainstorming session)
**Stack:** React Native 0.81.5 + Expo ~54.0.31 (mobile) · NestJS 11 + GraphQL + Prisma (backend)
**Context:** Brownfield — narative, like button, lien de partage et flow d'onboarding existent

---

## Executive Summary

Le lien de partage Nara génère aujourd'hui des clics qui aboutissent sur un écran blanc pour les non-membres. La feature Narative Subscription transforme ce cul-de-sac en portail : tout utilisateur recevant un lien peut s'abonner au narative en un geste, accéder à l'intégralité de son historique en lecture seule, et démarrer son aventure sociale sur Nara. Les membres partagent leurs souvenirs sans diluer l'intimité du groupe ; les destinataires accèdent au contenu sans s'engager comme membres.

**Utilisateurs cibles :** Non-membres recevant un lien de partage, ou accédant à un narative public depuis l'app (si ami d'un membre).

**Problème résolu :** Absence de tier d'accès intermédiaire entre "membre" (accès complet + contribution) et "exclu" (écran blanc). Chaque lien partagé représente aujourd'hui de la valeur perdue.

**Valeur émotionnelle :** Le Narative Entry Screen est un portail vers les souvenirs de quelqu'un — nostalgie, connexion humaine, appartenance. Pour le membre qui partage, c'est la certitude que ses souvenirs seront vus sans compromettre l'intimité du groupe.

**Valeur croissance :** Le lien de partage existant est un canal d'acquisition inexploité. Chaque partage devient un vecteur d'acquisition : visiteur → abonné → potentiel membre → potentiel créateur de narative. La migration des likes existants en abonnements crée une base d'abonnés immédiate au launch — sous réserve d'une communication in-app proactive.

**Impact architectural :** `NarativeAccessService` centralisé requis — tous les resolvers accédant au contenu d'un narative doivent gérer le nouveau tier `isSubscriber`.

---

## Success Criteria

### User Success
- Un non-membre accède au contenu du narative en ≤ 2 taps depuis un lien de partage
- Un utilisateur sans compte crée son compte, s'abonne, et accède au narative en une session continue
- Un abonné navigue l'intégralité de l'historique via le storyteller sans blocage
- Les utilisateurs dont les likes existants sont migrés comprennent le changement (0 plainte de "je ne me suis pas abonné")

### Business Success
- **Launch :** Migration 100% des likes existants → abonnements sans incident
- **Semaine 1 :** Baseline de conversion lien de partage → abonnement établie via Pendo
- **3 mois :** Taux de conversion lien → abonnement mesurable (aujourd'hui 0% — écran blanc)
- **3 mois :** Nombre d'abonnés actifs (≥ 1 ouverture narative en tant qu'abonné)
- **6 mois :** % d'abonnés demandant à devenir membres (progression dans le funnel)

### Technical Success
- `NarativeAccessService` centralisé — aucun resolver n'implémente sa propre logique d'accès narative
- Zéro régression sur les flows existants (membre, ami d'un membre)
- Révocation d'accès effective en temps réel (désactivation, blocage, changement confidentialité)
- Zéro incident de sécurité lié au nouveau tier (abonné accédant à du contenu non autorisé)

---

## Product Scope

### MVP — v0
- Like = abonnement (repurposing sémantique — compteur likes devient compteur d'abonnés)
- Narative Entry Screen (nouveau composant) — actions conditionnelles selon visibilité du narative
- Accès complet read-only à tout l'historique du narative (naratives publics uniquement)
- Unlike = désabonnement
- Abonnement activé par défaut sur naratives publics, désactivable par n'importe quel membre
- Toggle "Autoriser les abonnements" masqué/désactivé sur naratives privés
- Edge cases : désactivation → perte d'accès immédiate · blocage membre → perte d'accès
- Révocation immédiate et silencieuse lors de changement de visibilité public → privé
- Validation explicite obligatoire pour toute demande de membership (pas d'auto-accept)
- N'importe quel membre peut accepter une demande d'ajout (pas seulement l'auteur), depuis la page notification
- Migration des likes existants → abonnements + communication in-app au launch
- `NarativeAccessService` centralisé (backend)

> **Hors scope MVP — projet dédié :** Flow "Coller un lien nara" post-onboarding. Voir projet Linear [Deep Link — Paste a Nara Link](https://linear.app/naraa/project/deep-link-paste-a-nara-link-9ee92a6ce950).

### Growth — v1
- Notifications push lors de l'ajout d'un nouveau moment par un membre

### Vision — Future
- Interactions abonné (réactions, commentaires)
- Vue web pour non-utilisateurs Nara
- Feed de découverte publique de naratives

---

## User Journeys

### Journey 1 — Narative PUBLIC · utilisateur sans compte

Marc reçoit un lien vers un narative public. Il n'a pas l'app. Il est redirigé vers l'App Store, télécharge, crée son compte. Il arrive sur le **Narative Entry Screen**.

- **S'abonner** → redirigé vers la Narative Page en lecture seule.
- **Demander à devenir membre** → le bouton change d'état ("Demande envoyée ! Les membres du Nara seront notifiés"), pas de redirection. Marc reçoit une notification quand un membre accepte.

> **Note :** Le flow de récupération du lien post-onboarding (deep link) est hors scope de cette feature. Voir projet dédié [Deep Link — Paste a Nara Link](https://linear.app/naraa/project/deep-link-paste-a-nara-link-9ee92a6ce950).

---

### Journey 2 — Narative PUBLIC · utilisateur avec compte Nara

Léa reçoit un lien. Elle a l'app. Elle arrive sur le **Narative Entry Screen**.

- **S'abonner** → redirigée vers la Narative Page en lecture seule.
- **Demander à devenir membre** → le bouton change d'état ("Demande envoyée ! Les membres du Nara seront notifiés"), pas de redirection. Validation explicite obligatoire par n'importe quel membre du narative, quelle que soit la relation d'amitié.

---

### Journey 3 — Narative PRIVÉ · utilisateur avec ou sans compte

Thomas reçoit un lien vers un narative privé. Il arrive sur le **Narative Entry Screen** (si non connecté : authentification d'abord).

- **Pas de bouton "S'abonner"** — l'abonnement n'est pas disponible sur les naratives privés.
- **Demander à devenir membre** → le bouton change d'état ("Demande envoyée ! Les membres du Nara seront notifiés"), pas de redirection. Validation explicite obligatoire par n'importe quel membre du narative.

---

### Journey 4 — Antoine désactive l'abonnement (membre, edge case)

Antoine gère un narative familial. Des abonnés inconnus le suivent. Il ouvre les settings et désactive les abonnements. Confirmation : *"Les X abonnés actuels perdront l'accès immédiatement."* Il confirme. Les abonnés sont révoqués en temps réel.

---

### Journey Requirements Summary

| Capability | Journeys |
|-----------|---------|
| Narative Entry Screen | 1, 2, 3 |
| S'abonner (narative public uniquement) | 1, 2 |
| Demander à devenir membre (public et privé) | 1, 2, 3 |
| Validation explicite membre — aucun auto-accept | 1, 2, 3 |
| Accès read-only + storyteller | 1, 2 |
| Toggle abonnement + révocation temps réel | 4 |

---

## Functional Requirements

### Subscription Management
- **FR1 :** Un utilisateur s'abonne à un narative en le likant
- **FR2 :** Un utilisateur se désabonne en retirant son like
- **FR3 :** Le système convertit les likes existants en abonnements au launch
- **FR4 :** Le compteur de likes d'un narative affiche le nombre d'abonnés
- **FR5 :** Un membre active ou désactive les abonnements depuis les settings narative
- **FR6 :** Le système révoque l'accès de tous les abonnés immédiatement lors de la désactivation
- **FR7 :** Un abonné promu membre cesse automatiquement d'être abonné (statuts exclusifs)

### Narative Access Control
- **FR8 :** Un abonné accède à l'intégralité du contenu d'un narative (historique complet)
- **FR9 :** Un abonné ne peut pas modifier, ajouter ou supprimer de contenu
- **FR10 :** Le système bloque l'accès d'un abonné révoqué, y compris via cache ou deep link direct
- **FR11 :** Lorsqu'un membre bloque un abonné, l'abonné perd immédiatement l'accès au narative
- **FR12 :** Lorsque la confidentialité change (public → privé), tous les abonnés perdent l'accès immédiatement et silencieusement — pas de choix proposé au membre, pas de notification aux abonnés
- **FR13 :** L'abonnement est uniquement disponible sur les naratives publics

### Entry Points & Navigation
- **FR14 :** Un utilisateur accède au Narative Entry Screen depuis un lien de partage, quel que soit le type de narative (public ou privé)
- **FR15 :** Un utilisateur non connecté est redirigé vers l'écran d'authentification avant d'accéder au Narative Entry Screen

### Narative Entry Screen
- **FR18 :** Le Narative Entry Screen affiche un aperçu du narative (titre, cover, moments, journées, membres, dates) pour tout visiteur, quel que soit le type de narative
- **FR19 :** Sur un narative **public**, le Narative Entry Screen propose "S'abonner" comme action principale et "Demander à devenir membre" comme action secondaire
- **FR20 :** Sur un narative **privé**, le Narative Entry Screen propose uniquement "Demander à devenir membre" — pas de bouton "S'abonner"
- **FR21 :** Le Narative Entry Screen propose d'envoyer des demandes d'amis aux membres du narative (optionnel, non bloquant), visible dans les deux cas (public et privé)
- **FR22 :** Après abonnement ("S'abonner"), l'utilisateur est redirigé vers la Narative Page en tant qu'abonné
- **FR32 :** Lorsque l'utilisateur clique sur "Demander à devenir membre", le bouton change d'état ("Demande envoyée ! Les membres du Nara seront notifiés"), sans redirection — quelle que soit la visibilité du narative ou la relation d'amitié avec les membres
- **FR33 :** Toute demande de membership requiert une validation explicite par un membre du narative. Aucune auto-acceptation n'est possible
- **FR34 :** N'importe quel membre du narative peut accepter une demande d'ajout depuis la page notification — pas uniquement l'auteur/créateur
- **FR35 :** Le bouton "Demander à devenir membre" en état "Demande envoyée" est non-interactif jusqu'à annulation ou acceptation

### Content Experience
- **FR23 :** Un abonné navigue l'historique complet d'un narative sur la narative page et via le storyteller
- **FR24 :** Un abonné voit la même vue de contenu qu'un membre (sans les fonctionnalités d'édition)

### Settings & Configuration
- **FR25 :** Les settings narative affichent le toggle "Autoriser les abonnements" uniquement lorsque le narative est **public** ; le toggle est masqué sur les naratives privés
- **FR26 :** La désactivation du toggle affiche le nombre d'abonnés qui perdront l'accès avant confirmation
- **FR27 :** Un changement de visibilité public → privé révoque tous les abonnés immédiatement et silencieusement — aucun choix supplémentaire dans le narative form

### Backend & Autorisation
- **FR30 :** Le système centralise la logique d'autorisation narative dans un service unique (`NarativeAccessService`)
- **FR31 :** Le service d'autorisation expose le tier `isSubscriber` en plus des tiers `isMember` et `isFriend` existants

---

## Non-Functional Requirements

### Performance
- Abonnement (tap "S'abonner") confirmé et redirigé en < 2 secondes
- Révocation d'accès (désactivation, blocage) effective en < 1 seconde côté serveur
- Narative Entry Screen chargé en < 2 secondes sur connexion 4G standard
- Narative Entry Screen traite la demande d'abonnement et redirige en < 3 secondes

### Security
- `isSubscriber` vérifié côté serveur sur chaque requête — aucune vérification uniquement côté client
- Abonné révoqué bloqué y compris via token en cache ou deep link direct
- PII des membres (numéros de téléphone hashés) non exposés aux abonnés
- Script de migration exécuté avec log d'audit et idempotence garantie

### Scalability
- `NarativeAccessService` supporte le volume actuel × 10 sans dégradation
### Reliability
- Flow d'abonnement affiche un message d'erreur explicite + option de retry si offline
- Deep link hors scope — projet dédié créé

---

## Security Considerations

- **Tier d'accès :** `isSubscriber` est un état serveur — jamais dérivé du client. Toute requête GraphQL accédant au contenu d'un narative passe par `NarativeAccessService`.
- **Révocation :** La révocation (désactivation narative, blocage, changement confidentialité) invalide l'accès côté serveur ET purge le cache Apollo client mobile pour éviter l'accès offline au contenu post-révocation.
- **PII :** Les numéros de téléphone restent hashés (`HashedPhoneNumber`) — non exposés aux abonnés dans aucune query GraphQL.
- **Deep link :** Le lien de partage ne contient pas de token d'accès — l'abonnement est validé côté serveur après action explicite de l'utilisateur. Le flow de récupération de lien post-install est hors scope (projet dédié).

---

## Pendo Tracking Plan

Les événements suivants mesurent les success metrics définis :

| Événement Pendo | Déclenché quand | Métrique associée |
|----------------|----------------|-------------------|
| `narative_subscribed` | Utilisateur like un narative → abonnement créé | Taux de conversion lien → abonnement |
| `narative_unsubscribed` | Utilisateur unlike un narative → abonnement révoqué | Taux de désabonnement |
| `narative_entry_screen_viewed` | Narative Entry Screen affiché | Reach du funnel |
| `narative_entry_screen_subscribed` | CTA S'abonner tapé depuis Narative Entry Screen | Conversion depuis le lien |
| `narative_entry_screen_member_requested` | CTA Demander à rejoindre tapé | Progression funnel abonné → membre |
| `narative_subscription_disabled` | Membre désactive l'abonnement | Usage du toggle |
| `subscriber_opened_narative` | Abonné ouvre un narative | Abonnés actifs |

---

## Domain-Specific Requirements

### GDPR & Privacy
- L'abonnement constitue une relation entre un utilisateur et le contenu d'un groupe — à documenter dans la politique de confidentialité
- Les profils des membres visibles aux abonnés respectent les paramètres de visibilité existants

### Mobile Constraints
- **Offline :** Contenu déjà chargé accessible offline pour les abonnés (même comportement que les membres). Le Narative Entry Screen requiert une connexion.
- **Deep link :** Hors scope — le flow de récupération de lien post-install est un projet dédié ([Deep Link — Paste a Nara Link](https://linear.app/naraa/project/deep-link-paste-a-nara-link-9ee92a6ce950))
- **Portrait only :** Tous les nouveaux écrans respectent la contrainte portrait de l'app
- **Permissions v0 :** Aucune permission device nouvelle requise
- **Permissions v1 :** `NOTIFICATIONS` iOS + canal Android pour les push

### Risk Mitigations
| Risque | Mitigation |
|--------|-----------|
| NarativeAccessService scope creep | Liste exhaustive des resolvers impactés définie en architecture avant dev |
| Deep link perdu après install | Hors scope — projet dédié créé |
| Abonné révoqué avec cache actif | Révocation serveur + purge cache Apollo client mobile |
| Dette sémantique `likedBy` | Documentée et assumée — à ré-évaluer si un vrai système de réactions est envisagé |
| Demande de membership sans réponse (membres inactifs) | Non traité en v0 — optimisation future |
