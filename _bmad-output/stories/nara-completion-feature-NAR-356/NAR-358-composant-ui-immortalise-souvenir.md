# NAR-358 — [P0] Composant UI "Immortalise ce souvenir" — progress bar + checklist 5 critères

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Créer le composant React Native affiché sur la page Narative entre le header et la section moments, visible uniquement si `moments.count >= 1` ET `score < 5/5` ET `daysSinceEndDate < 15`.

## Specs UI

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

## Acceptance criteria

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
