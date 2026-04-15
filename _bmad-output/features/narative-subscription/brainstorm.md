# Brainstorm — Narative Subscription

_Session du 2026-04-10 — Challengé et révisé en profondeur_

## Feature Concept

Le lien de partage Nara droppe actuellement les non-membres sur un écran blanc. **Narative Subscription** transforme ce cul-de-sac en portail d'accueil via la **Narative Entry Screen**.

N'importe quel utilisateur recevant un lien vers un narative **public** peut s'abonner en un tap (via le bouton like existant) et accéder à l'historique complet en mode lecture seule. Pour un narative **privé**, seule la demande de membership est possible — pas d'abonnement. Les membres partagent leurs souvenirs sans compromettre l'intimité du groupe.

**Distinction sémantique fondamentale :**
- **Membre** = quelqu'un qui a vécu ou va vivre le narative (voyage, événement). Appartenance physique réelle.
- **Abonné** = quelqu'un qui suit le narative pour recevoir des updates, sans en faire partie. Observateur read-only.

## Pendo Signals

- Aucune donnée Pendo disponible — authentification non effectuée lors de la session
- Aucun signal feedback sur le partage de narative ou l'accès non-membre
- Brainstorming conduit sans ancrage data quantitatif

## Ideation Summary

### Ideas explored

**Techniques utilisées : Premortem (avocat du diable) + User Journey Safari**

- **Table `NarativeSubscription` dédiée** — sémantique propre, évolutif, trackable. Écarté : `likedBy` maintenu par choix délibéré (coût immédiat vs dette future acceptée).
- **Auto-accept si ami avec un membre** — réduire friction demande de membership. Écarté : trop permissif pour des groupes de vie intime.
- **Deep link automatique** (Universal Links / Branch.io) — éliminer le flow manuel. Écarté en scope Subscription, projet dédié créé.
- **Abonnement sur naratives privés** — permettre l'abonnement read-only même sur privé. Écarté : contredit l'intention de confidentialité.
- **Réactions abonnés** (likes, commentaires) — engagement des abonnés. Écarté pour v0, à considérer en v1.
- **Accès partiel** (fenêtre temporelle, moments tageables) — granularité du contenu visible. Écarté : complexité inutile pour v0.
- **Web viewer** (landing page web sans app) — zéro friction d'install. Écarté : hors scope, projet futur potentiel.

### Selected approach

**Entry Screen conditionnelle + abonnement public uniquement + demande de membership avec validation explicite.**

- La **Narative Entry Screen** s'affiche dans tous les cas (lien partagé), mais les actions disponibles dépendent de la visibilité du narative
- **Narative public** : bouton "S'abonner" (like → abonné immédiat) + lien "Demander à devenir membre"
- **Narative privé** : lien "Demander à devenir membre" uniquement — pas d'abonnement possible
- **Subscribe = Like** (`likedBy` réutilisé) : choix maintenu délibérément malgré la dette sémantique
- **Validation explicite** pour toute demande de membership, dans tous les cas, sans exception
- **Tous les membres** peuvent accepter une demande (pas seulement l'auteur), depuis la page notification

### Risks & anti-patterns identified

- **Dette sémantique `likedBy`** — le champ signifie "abonné" mais s'appelle "liked". Risque si un vrai système de réactions est introduit plus tard. Documenté et accepté.
- **Conversion post-install cassée** — le flow "coller un lien nara" est le maillon faible du funnel mais hors scope. Risque de drop massif entre l'install et l'arrivée sur la Entry Screen.
- **Contenu sensible exposé** — l'abonné voit tout l'historique y compris les moments anciens. À documenter dans les edge cases PRD.
- **Révocation brutale sans notification** — désactiver les abonnements coupe l'accès immédiatement sans prévenir les abonnés. Comportement validé et assumé.
- **Membre inactif bloquant** — si tous les membres sont inactifs, les demandes de membership restent en attente indéfiniment. Non traité en v0.

### Adjacent opportunities

- **Deep link automatique** — projet Linear créé : "Deep Link — Paste a Nara Link"
- **Interactions abonnés v1** — réactions, commentaires en lecture
- **Liste d'abonnés visible aux membres** — analytics léger
- **Onboarding lite pour arrivants via lien** — réduire le funnel post-install
- **Écran "Mes demandes en attente"** — visibilité sur les demandes de membership envoyées
- **Web viewer** — accès narative sans app pour les non-utilisateurs Nara

## Research Findings

None — skipped.

## Decisions for the PRD

### Sémantique des rôles
1. **Membre** = participant réel du narative (a vécu ou va vivre l'expérience). Peut contribuer du contenu.
2. **Abonné** = observateur read-only. Suit le narative pour les updates. N'a pas vécu l'expérience.
3. **Statuts exclusifs** — abonné promu membre → retiré de `likedBy` automatiquement.

### Modèle d'abonnement
4. **Subscribe = Like** — `likedBy` réutilisé comme liste d'abonnés. Pas de nouvelle table.
5. **Abonnement uniquement sur naratives publics** — les naratives privés ne permettent pas l'abonnement.
6. **Unlike = désabonnement.**
7. **Désactivation toggle → perte d'accès immédiate** pour tous les abonnés existants. Pas de notification, pas de grace period.
8. **Toggle "Autoriser les abonnements" masqué/désactivé** quand le narative est Privé.
9. **Pas de warning** lors du changement de visibilité — révocation immédiate et silencieuse.

### Entry Screen
10. **Narative public** : "S'abonner" (CTA principal) + "Demander à devenir membre" (lien secondaire).
11. **Narative privé** : "Demander à devenir membre" uniquement.
12. **Pas de sous-titre** sur la Entry Screen — supprimé (incohérent entre public et privé).
13. **Métadonnées visibles** sur les deux types : titre, cover, moments, journées, membres, dates.
14. **Membres visibles** même sur narative privé — intentionnel.
15. **"Ajouter en ami les membres"** — in scope MVP, affiché sur la Entry Screen.

### Demande de membership
16. **Validation explicite obligatoire** dans tous les cas — narative public ou privé, amis ou non.
17. **Tous les membres peuvent accepter** une demande, pas seulement l'auteur.
18. **Validation depuis la page notification.**
19. **Post-demande** : afficher "Les membres du Nara seront notifiés" sous l'état "Demande envoyée !".

### Migration
20. **Likes existants → abonnements** au launch. Migration idempotente via guard `SystemConfig`.
21. **Notification in-app** aux utilisateurs migrés.

### Hors scope — projet dédié
22. **"Coller un lien nara"** — flow post-onboarding retiré du scope. Projet Linear créé : [Deep Link — Paste a Nara Link](https://linear.app/naraa/project/deep-link-paste-a-nara-link-9ee92a6ce950).

## Out of scope (decided here)

- **Deep link automatique / paste link page** — projet dédié créé, hors scope Subscription
- **Notifications abonnés** (révocation, nouveaux moments) — v1
- **Interactions abonnés** (réactions, commentaires) — v1 ou plus
- **Auto-accept de membership** (même si ami avec un membre) — supprimé, trop permissif
- **Onboarding lite pour arrivants via lien** — optimisation future
- **Écran "Mes demandes en attente"** — optimisation future
- **Granularité du contenu partagé** (moments tageables) — trop complexe pour v0
- **Web viewer pour non-utilisateurs Nara** — adjacent, pas v0
- **Feed de découverte publique** — adjacent, pas v0
- **Notification abonné lors de révocation** — comportement assumé : pas de notification
