# [EPIC] NAR-356 — Complétion du Souvenir

**Identifiant :** NAR-356
**Statut :** Todo
**Priorité :** Medium (2)
**Assigné à :** Theo Rosen
**Équipe :** Nara
**Créé le :** 2026-03-18
**Mis à jour le :** 2026-03-18 *(suite session brainstorming)*

---

## Description

### Problème

Aujourd'hui un Narative peut rester vide ou pauvre indéfiniment — l'utilisateur n'a aucun signal lui indiquant que son souvenir est "incomplet" et aucun trigger externe pour revenir l'enrichir après la fin du voyage.

Résultat : Naratives fantômes, valeur perçue faible, switching cost qui ne se construit jamais.

### Solution

1. **Score de complétude in-app** — composant "Immortalise ce souvenir" avec 5 critères (progress bar + checklist interactive) affiché entre le header et les moments, dès qu'il y a ≥1 moment.
2. **Notification post end-date** — push J+1, J+7 et J+14 après end_date du Narative si score < 4/5.

### Objectifs (60j)

| Métrique | Cible |
| -- | -- |
| % Naratives score ≥3/5 | +40% vs baseline |
| Sessions "revisit" Narative terminé | +25% |
| Open rate push J+1 | ≥35% |
| Conversion post-notif (≥1 critère complété) | ≥20% |

### Les 5 critères V1

| # | Critère | Type | Note |
| -- | -- | -- | -- |
| 1 | Photo de couverture définie | Auto | Obligatoire à la création — toujours validé |
| 2 | Lieu défini | Auto | Obligatoire à la création — toujours validé |
| 3 | ≥3 moments créés | Optionnel | |
| 4 | ≥1 autre membre invité | Optionnel | Option "je voyage seul" pour valider sans action sociale |
| 5 | Partage ton souvenir | Optionnel | Validé à l'ouverture du share sheet natif |

> **Critère retiré du scope V1 :** ≥50% des moments avec description (≥50 chars) — sera réintroduit avec la feature de description vocale (Whisper). Le slot peut rester visible dans l'UI avec label *"Bientôt ✨"*.

> **Note :** Tout le monde démarre à 2/5 (critères 1 et 2 automatiques). L'Endowed Progress Effect est intentionnel.

### UI — États du composant

| Condition | État | Message header |
| -- | -- | -- |
| 0 moment | Masqué | — |
| ≥1 moment, avant end_date, score <5/5 | En cours | *"Continue comme ça ✨"* |
| ≥1 moment, avant end_date, score == maxScore | Badge pendant voyage | 🏆 *"Souvenir immortalisé"* + sous-titre *"Ton souvenir n'est pas fini, continue de le documenter"* |
| ≥1 moment, après end_date, score <5/5 | Complétion | *"Immortalise ce souvenir ✨"* |
| Score 5/5, après end_date | Badge | *"Souvenir immortalisé 🏆"* |

**Comportement de la carte :**
- Dépliée à la première visite post-création
- Repliée par défaut à toutes les visites suivantes
- Header replié affiche le score : *"✨ Immortalise ce souvenir — 3/5 ›"*
- Composant visible jusqu'à end_date + 15 jours, puis disparaît

### Décisions finales

| Question | Décision |
| -- | -- |
| Souvenir privé + critère "Partage" | Si le Narative est privé → critère #5 "Partage" retiré du score → max 4/4 |
| "Je voyage seul" | Retire le critère #4 du score → max 4/4 pour les solos |
| Badge 5/5 pendant le voyage (avant end_date) | Afficher "Souvenir immortalisé 🏆" + sous-titre *"Ton souvenir n'est pas fini, continue de le documenter"* |
| Copy notification J+14 | *"Ton souvenir est presque complet ! Ajoute tes moments pour le finaliser"* |

---

## Stories

### NAR-357 — [P0] Score de complétude — Calcul des 5 critères (client-side)

**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Calculer dynamiquement un score de 0 à 5 pour chaque Narative, à partir des données déjà disponibles dans la query existante.

#### Critères et logique technique

| # | Critère | Condition | Retiré si |
| -- | -- | -- | -- |
| 1 | Photo de couverture | `Narative.thumbnail != null` | — |
| 2 | Lieu défini | `Narative.location != null` | — |
| 3 | ≥3 moments | `Narative.moments.count >= 3` | — |
| 4 | ≥1 autre membre | `Narative.members.count >= 2` | `soloMode == true` → critère retiré du score |
| 5 | Partage effectué | `shareSheetOpened == true` *(flag local)* | `Narative.isPrivate == true` → critère retiré du score |

> **Score max variable :** 5/5 (par défaut) · 4/4 (solo OU privé) · 3/3 (solo ET privé)

#### Implémentation

- Calcul **client-side** à partir des données de la query Narative existante
- Pas de nouvel endpoint nécessaire
- Le score est recalculé à chaque update — **régression autorisée** (calcul pur, pas de persistance)
- Exposer un hook/helper `useNarativeCompletionScore(narative)` → retourne `{ score, maxScore, criteria[], isOngoing }`
- `isOngoing: boolean` = `currentDate < narative.endDate` — utilisé par l'UI pour le wording date-aware
- `maxScore` = nombre de critères actifs selon profil (solo/privé)

#### Acceptance criteria

- [ ] Le score est correct pour tous les critères actifs selon le profil
- [ ] Le score se met à jour en temps réel après une action
- [ ] `isOngoing` est correctement calculé selon la date du jour vs end_date
- [ ] `soloMode == true` retire le critère #4 → maxScore = 4
- [ ] `isPrivate == true` retire le critère #5 → maxScore = 4
- [ ] Les deux simultanément → maxScore = 3
- [ ] Le score peut régresser si un critère devient non satisfait (comportement normal)

---

### NAR-358 — [P0] Composant UI "Immortalise ce souvenir" — progress bar + checklist 5 critères

**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Créer le composant React Native affiché sur la page Narative entre le header et la section moments, visible uniquement si `moments.count >= 1` ET `score < 5/5` ET `daysSinceEndDate < 15`.

#### Specs UI

**Container**
- Background : `#fff` avec shadow légère
- Border radius : 16px, margin 14px horizontal

**Header (collapsible)**
- État replié : *"✨ [wording] — X/5 ›"* (score visible en permanence dans le header)
- État déplié : emoji + titre + sous-titre dynamique + chevron ∨
- Wording selon `isOngoing` :
  - `isOngoing == true` → *"Continue comme ça"*
  - `isOngoing == false` → *"Immortalise ce souvenir"*
- Sous-titre dynamique selon score (après end_date) :
  - 2/5 → *"Tes souvenirs méritent mieux qu'une galerie photo."*
  - 3/5 → *"Tu y es presque — encore quelques touches !"*
  - 4/5 → *"Un dernier effort pour un souvenir parfait."*

**Comportement d'ouverture**
- **Première visite** : carte dépliée par défaut
- **Visites suivantes** : carte repliée par défaut
- Persister l'état "première visite vue" en local (AsyncStorage ou équivalent)

**Durée de vie**
- Composant visible de la création jusqu'à `end_date + 15 jours`
- Disparaît silencieusement après ce délai si score < 5/5

**Progress bar**
- Gradient `#b060d0 → #e0508a`, height 8px
- Label `X/5` à droite, Semi-bold 20px

**Checklist**
- ✅ Critère validé : texte barré, couleur grisée, checkmark vert `#27b96a`
- ○ Critère manquant : texte normal + chevron `›` (tappable → navigation)
- Critère #4 "Ajoute un ami" : afficher option *"Je voyage seul →"* en sous-texte si non validé
- Critère descriptions : slot visible avec label *"Bientôt — décris tes moments à la voix ✨"* (non interactif)
- Séparation par ligne fine `#f2f2f2`

#### Acceptance criteria

- [ ] Composant affiché uniquement si ≥1 moment ET score < 5/5 ET dans la fenêtre de 15 jours post end_date
- [ ] Progress bar animée à chaque progression du score
- [ ] Header collapsible avec score visible en état replié
- [ ] Carte dépliée à la première visite, repliée par défaut ensuite
- [ ] Wording date-aware (avant/après end_date)
- [ ] Sous-titre change dynamiquement selon score
- [ ] Critères validés : texte barré + checkmark vert
- [ ] Critères manquants : chevron + tappable
- [ ] Option "Je voyage seul" visible sur le critère #4
- [ ] Slot critère "descriptions" affiché comme *"Bientôt ✨"* non interactif

---

### NAR-359 — [P0] Navigation contextuelle depuis chaque critère manquant

**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Chaque critère manquant (○) dans la checklist est tappable et redirige vers l'écran ou l'action correspondante.

#### Mapping critère → destination

| Critère | Action au tap |
| -- | -- |
| Couverture ajoutée | → Édition du Narative (écran edit, focus thumbnail) |
| Lieu défini | → Édition du Narative (focus localisation) |
| ≥3 moments | → Bottom sheet "+ Ajouter un moment" |
| Ajoute au moins un ami | → Écran membres du Narative |
| "Je voyage seul" | → Active `soloMode = true` + valide le critère #4 immédiatement |
| Partage ton souvenir | → Share sheet natif avec lien du Narative (validation à l'ouverture) |

#### Comportement du CTA "Compléter maintenant"

- Scroll vers + highlight du premier critère manquant dans la liste
- Si l'utilisateur tape dessus ensuite → navigation contextuelle

#### Tracking

Chaque tap sur un critère manquant doit émettre l'event :
`COMPLETION_CRITERIA_TAPPED { narative_id, criterion, score_before }`

#### Acceptance criteria

- [ ] Les 5 destinations sont correctement mappées
- [ ] Le tap sur un critère validé (✅) ne déclenche aucune navigation
- [ ] Le CTA scroll + highlight avant navigation
- [ ] "Je voyage seul" valide le critère #4 sans navigation externe
- [ ] Ouverture du share sheet valide le critère #5 immédiatement
- [ ] Event tracking émis au tap

---

### NAR-360 — [P0] Backend — Trigger push notification J+1 après end_date (score < 4/5)

**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Envoyer une push notification aux propriétaires de Naratives terminés dont le score est inférieur à 4/5, 24h après la end_date.

#### Logique de déclenchement

```
À end_date + 24h :
  Si score < 4/5 → Envoyer push J+1
  Sinon → Rien

À end_date + 7j :
  Si score encore < 4/5 ET push J+1 envoyée ET pas de conversion → Envoyer push J+7

À end_date + 14j :
  Si score encore < 4/5 ET carte encore visible → Envoyer push J+14 (avertissement disparition)
  (max 3 pushes par Narative, jamais de 4ème)
```

#### Copy notifications

**J+1**
- Titre : `Ton {Narative.title} est terminé ! ✨`
- Corps : `Il reste {5 - score} chose(s) pour immortaliser ce souvenir.`
- Thumbnail : image de couverture du Narative (si définie)
- Action : deeplink `naraapp://narative?id={narativeId}&source=completion_notif`

**J+7**
- Titre : `Les souvenirs s'effacent vite...`
- Corps : `Ton {Narative.title} attend encore d'être immortalisé. C'est le dernier rappel.`
- Action : deeplink `naraapp://narative?id={narativeId}&source=completion_notif`

**J+14** *(voir NAR-367)*
- Copy à définir avec Théo — ton urgence douce, dernière chance avant disparition de la carte

#### Tracking

- `COMPLETION_NOTIF_J1_SENT { narative_id, score_at_send }`
- `COMPLETION_NOTIF_J7_SENT { narative_id, score_at_send }`
- `COMPLETION_NOTIF_J14_SENT { narative_id, score_at_send }`

#### Acceptance criteria

- [ ] Trigger déclenché exactement à end_date + 24h
- [ ] Condition score < 4/5 vérifiée au moment du déclenchement (pas au moment de la création)
- [ ] Pas de push si score ≥ 4/5 au moment du trigger
- [ ] Jamais plus de 3 pushes par Narative
- [ ] Copy dynamique avec Narative.title et compte de critères manquants sur /5
- [ ] Deeplink inclus dans la notification

---

### NAR-361 — [P0] Deep link `?source=completion_notif` — scroll auto + highlight composant

**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Quand l'utilisateur arrive sur la page Narative depuis la push notification, le composant "Immortalise ce souvenir" est mis en avant automatiquement.

#### Comportement attendu

1. La page Narative charge normalement
2. **Après 600ms** → scroll automatique vers le composant de complétude
3. Le composant reçoit une **animation de pulse** (border colorée, 1 fois)
4. Un **toast** apparaît en haut : *"Voici ce qui manque à ce souvenir ✨"* (disparaît après 3s)

#### Implémentation

- Parser le param `source` dans le deep link : `naraapp://narative?id=xxx&source=completion_notif`
- Si `source === 'completion_notif'` → déclencher le comportement ci-dessus
- S'assurer que le composant est déplié (état ouvert) à l'arrivée

#### Tracking

`COMPLETION_NOTIF_OPENED { narative_id, notif_day, score_at_open }`

#### Acceptance criteria

- [ ] Le param `source` est correctement parsé depuis le deep link
- [ ] Scroll automatique vers le composant après 600ms
- [ ] Animation pulse visible (une seule fois)
- [ ] Toast affiché 3s puis disparaît
- [ ] Composant forcé en état déplié si l'utilisateur l'avait réduit
- [ ] Event `COMPLETION_NOTIF_OPENED` émis

---

### NAR-362 — [P1] État "Souvenir immortalisé" — badge 5/5 + feedback visuel

**Statut :** Backlog | **Priorité :** Medium (3) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Quand le score atteint 5/5 après end_date, remplacer le composant "Immortalise ce souvenir" par un badge compact célébrant l'accomplissement.

#### UI — Badge compact "Souvenir immortalisé"

**Container**
- Background : `#edfaee` (vert très désaturé)
- Border radius : 16px, même position que le composant principal

**Contenu**
- 🏆 + "Souvenir immortalisé !" — Semi-bold 16px
- Sous-titre : *"Ce souvenir vivra pour toujours. ✨"*
- Progress bar verte full (gradient `#3dd68c → #27b96a`)
- Label `5/5`
- Chevron `›` → composant collapsible (tap → réduit)

**Animation au passage 4→5**
- Légère animation confetti (durée ~1.5s, non bloquante)
- Le composant principal "pulse" brièvement en vert avant de laisser place au badge

> **Note :** Le badge n'apparaît qu'après end_date. Si score atteint 5/5 pendant le voyage, afficher *"Parfait pour l'instant 🎉"* à la place (wording date-aware géré par NAR-358).

#### Acceptance criteria

- [ ] Le badge remplace le composant principal uniquement si score == 5/5 ET après end_date
- [ ] Animation confetti au moment du passage à 5/5 (uniquement à la transition, pas au chargement)
- [ ] Badge collapsible
- [ ] Event `COMPLETION_ACHIEVED { narative_id, time_to_complete_days }` émis

---

### NAR-363 — [P1] Push J+7 — Rappel final si score < 4/5 et non converti

**Statut :** Backlog | **Priorité :** Medium (3) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Envoyer un second rappel 7 jours après la end_date, si l'utilisateur n'a pas réagi à la push J+1.

#### Conditions de déclenchement

```
À end_date + 7j :
  ET push J+1 envoyée
  ET score encore < 4/5
  ET aucun critère complété depuis la push J+1 ("pas de conversion")
→ Envoyer push J+7
```

#### Copy

- Titre : `Les souvenirs s'effacent vite...`
- Corps : `Ton {Narative.title} attend encore d'être immortalisé. C'est le dernier rappel avant que la carte disparaisse.`
- Même deeplink que J+1 : `naraapp://narative?id={narativeId}&source=completion_notif`

#### Tracking

`COMPLETION_NOTIF_J7_SENT { narative_id, score_at_send }`
`COMPLETION_NOTIF_CONVERTED { narative_id, notif_day, criteria_completed }` si conversion

#### Acceptance criteria

- [ ] Push J+7 envoyée uniquement si J+1 déjà envoyée + score encore < 4/5 + pas de conversion
- [ ] Tracking correct

---

### NAR-364 — [P1] Tracking analytics — Events COMPLETION_***

**Statut :** Backlog | **Priorité :** Medium (3) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Implémenter tous les events de tracking nécessaires pour mesurer l'impact de la feature et itérer.

#### Events à implémenter

| Event | Payload | Déclencheur |
| -- | -- | -- |
| `COMPLETION_SCORE_VIEWED` | `{ narative_id, score, source }` | Composant visible à l'écran (impression) |
| `COMPLETION_CRITERIA_TAPPED` | `{ narative_id, criterion, score_before }` | Tap sur un critère manquant ○ |
| `COMPLETION_CTA_TAPPED` | `{ narative_id, score }` | Tap "Compléter maintenant" |
| `COMPLETION_SCORE_INCREASED` | `{ narative_id, old_score, new_score, criterion }` | Score progresse d'un point |
| `COMPLETION_ACHIEVED` | `{ narative_id, time_to_complete_days }` | Score atteint 5/5 après end_date |
| `COMPLETION_NOTIF_J1_SENT` | `{ narative_id, score_at_send }` | Push J+1 envoyée |
| `COMPLETION_NOTIF_J7_SENT` | `{ narative_id, score_at_send }` | Push J+7 envoyée |
| `COMPLETION_NOTIF_J14_SENT` | `{ narative_id, score_at_send }` | Push J+14 envoyée |
| `COMPLETION_NOTIF_OPENED` | `{ narative_id, notif_day, score_at_open }` | Notification tappée |
| `COMPLETION_NOTIF_CONVERTED` | `{ narative_id, notif_day, criteria_completed }` | Critère validé dans les 24h suivant une notif |
| `COMPLETION_SOLO_MODE_ACTIVATED` | `{ narative_id }` | Utilisateur active "je voyage seul" |

#### Métriques de succès à mesurer

- % Naratives score ≥3/5 (G1)
- Sessions "revisit" +25% (G2)
- Open rate notif ≥35% (G3)
- Conversion post-notif ≥20% (G4)

#### Acceptance criteria

- [ ] Les 11 events sont implémentés
- [ ] Les payloads sont conformes au tableau ci-dessus
- [ ] `COMPLETION_SCORE_VIEWED` utilise une logique d'impression (visible ≥ 1s dans le viewport)
- [ ] `COMPLETION_NOTIF_CONVERTED` est correctement attribué à la notif (fenêtre de 24h)

---

### NAR-365 — ~~[BLOQUANT] GraphQL — Exposer statut conversation IA~~ **ANNULÉ**

**Statut :** Annulé | **Raison :** Le critère #6 (chat IA complété) a été retiré de la spec V1 et remplacé par "Partage ton souvenir". Cette story n'a plus d'objet.

---

### NAR-366 — ~~[BLOQUANT] Infra push schedulée~~ **ANNULÉ**

**Statut :** Annulé | **Raison :** L'infrastructure de notifications schedulées est déjà en place. Pas de travail d'architecture nécessaire avant de démarrer NAR-360.

---

### NAR-367 — [P0] Notification J+14 — Avertissement disparition de la carte

**Statut :** Todo | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

#### Objectif

Envoyer une notification à J+14 pour avertir l'utilisateur que la carte de complétion va disparaître le lendemain, afin de créer une dernière fenêtre d'engagement.

#### Conditions de déclenchement

```
À end_date + 14j :
  Si score < 5/5 (carte encore active)
→ Envoyer push J+14
```

#### Copy

- Titre : *"Ton souvenir est presque complet !"*
- Corps : *"Ajoute tes moments pour le finaliser avant que la carte disparaisse demain."*
- Action : deeplink `naraapp://narative?id={narativeId}&source=completion_notif`

#### Acceptance criteria

- [ ] Push envoyée uniquement à end_date + 14j si score < 5/5
- [ ] Deeplink avec `source=completion_notif` (même comportement que J+1/J+7)
- [ ] Copy validée avec Théo avant dev
- [ ] Event `COMPLETION_NOTIF_J14_SENT` émis

---

## Récapitulatif des stories

| ID | Titre | Priorité | Statut | Assigné |
| -- | -- | -- | -- | -- |
| NAR-357 | Score de complétude — Calcul 5 critères + isOngoing | P0 | Specified | matthieu |
| NAR-358 | Composant UI date-aware, collapsible, 15 jours | P0 | Specified | matthieu |
| NAR-359 | Navigation contextuelle + solo mode + share | P0 | Specified | matthieu |
| NAR-360 | Backend — Trigger push J+1/J+7/J+14 | P0 | Specified | matthieu |
| NAR-361 | Deep link completion_notif + highlight | P0 | Specified | matthieu |
| NAR-362 | Badge "Souvenir immortalisé" 5/5 post end_date | P1 | Backlog | matthieu |
| NAR-363 | Push J+7 rappel final | P1 | Backlog | matthieu |
| NAR-364 | Tracking analytics COMPLETION_*** (11 events) | P1 | Backlog | matthieu |
| ~~NAR-365~~ | ~~GraphQL statut conv. IA~~ | — | **Annulé** | — |
| ~~NAR-366~~ | ~~Infra push schedulée~~ | — | **Annulé** | — |
| NAR-367 | Notification J+14 — avertissement disparition carte | P0 | Todo | matthieu |
