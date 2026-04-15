# NAR-357 — [P0] Score de complétude — Calcul des 5 critères (client-side)

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Calculer dynamiquement un score de 0 à 5 pour chaque Narative, à partir des données déjà disponibles dans la query existante.

## Critères et logique technique

| # | Critère | Condition | Retiré si |
| -- | -- | -- | -- |
| 1 | Photo de couverture | `Narative.thumbnail != null` | — |
| 2 | Lieu défini | `Narative.location != null` | — |
| 3 | ≥3 moments | `Narative.moments.count >= 3` | — |
| 4 | ≥1 autre membre | `Narative.members.count >= 2` | `soloMode == true` → critère retiré du score |
| 5 | Partage effectué | `shareSheetOpened == true` *(flag local)* | `Narative.isPrivate == true` → critère retiré du score |

> **Score max variable :** 5/5 (par défaut) · 4/4 (solo OU privé) · 3/3 (solo ET privé)

## Implémentation

- Calcul **client-side** à partir des données de la query Narative existante
- Pas de nouvel endpoint nécessaire
- Le score est recalculé à chaque update — **régression autorisée** (calcul pur, pas de persistance)
- Exposer un hook/helper `useNarativeCompletionScore(narative)` → retourne `{ score, maxScore, criteria[], isOngoing }`
- `isOngoing: boolean` = `currentDate < narative.endDate` — utilisé par l'UI pour le wording date-aware
- `maxScore` = nombre de critères actifs selon profil (solo/privé)

## Acceptance criteria

- [ ] Le score est correct pour tous les critères actifs selon le profil
- [ ] Le score se met à jour en temps réel après une action
- [ ] `isOngoing` est correctement calculé selon la date du jour vs end_date
- [ ] `soloMode == true` retire le critère #4 → maxScore = 4
- [ ] `isPrivate == true` retire le critère #5 → maxScore = 4
- [ ] Les deux simultanément → maxScore = 3
- [ ] Le score peut régresser si un critère devient non satisfait (comportement normal)
