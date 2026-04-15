# NAR-359 — [P0] Navigation contextuelle depuis chaque critère manquant

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Chaque critère manquant (○) dans la checklist est tappable et redirige vers l'écran ou l'action correspondante.

## Mapping critère → destination

| Critère | Action au tap |
| -- | -- |
| Couverture ajoutée | → Édition du Narative (écran edit, focus thumbnail) |
| Lieu défini | → Édition du Narative (focus localisation) |
| ≥3 moments | → Bottom sheet "+ Ajouter un moment" |
| Ajoute au moins un ami | → Écran membres du Narative |
| "Je voyage seul" | → Active `soloMode = true` + valide le critère #4 immédiatement |
| Partage ton souvenir | → Share sheet natif avec lien du Narative (validation à l'ouverture) |

## Comportement du CTA "Compléter maintenant"

- Scroll vers + highlight du premier critère manquant dans la liste
- Si l'utilisateur tape dessus ensuite → navigation contextuelle

## Tracking

Chaque tap sur un critère manquant doit émettre l'event :
`COMPLETION_CRITERIA_TAPPED { narative_id, criterion, score_before }`

## Acceptance criteria

- [ ] Les 5 destinations sont correctement mappées
- [ ] Le tap sur un critère validé (✅) ne déclenche aucune navigation
- [ ] Le CTA scroll + highlight avant navigation
- [ ] "Je voyage seul" valide le critère #4 sans navigation externe
- [ ] Ouverture du share sheet valide le critère #5 immédiatement
- [ ] Event tracking émis au tap
