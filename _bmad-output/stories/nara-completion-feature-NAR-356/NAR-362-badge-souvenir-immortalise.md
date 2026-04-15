# NAR-362 — [P1] État "Souvenir immortalisé" — badge 5/5 + feedback visuel

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Backlog | **Priorité :** Medium (3) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Quand le score atteint 5/5 après end_date, remplacer le composant "Immortalise ce souvenir" par un badge compact célébrant l'accomplissement.

## UI — Badge compact "Souvenir immortalisé"

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

## Acceptance criteria

- [ ] Le badge remplace le composant principal uniquement si score == 5/5 ET après end_date
- [ ] Animation confetti au moment du passage à 5/5 (uniquement à la transition, pas au chargement)
- [ ] Badge collapsible
- [ ] Event `COMPLETION_ACHIEVED { narative_id, time_to_complete_days }` émis
