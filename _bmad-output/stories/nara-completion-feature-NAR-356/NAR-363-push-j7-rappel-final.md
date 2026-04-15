# NAR-363 — [P1] Push J+7 — Rappel final si score < 4/5 et non converti

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Backlog | **Priorité :** Medium (3) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Envoyer un second rappel 7 jours après la end_date, si l'utilisateur n'a pas réagi à la push J+1.

## Conditions de déclenchement

```
À end_date + 7j :
  ET push J+1 envoyée
  ET score encore < 4/5
  ET aucun critère complété depuis la push J+1 ("pas de conversion")
→ Envoyer push J+7
```

## Copy

- Titre : `Les souvenirs s'effacent vite...`
- Corps : `Ton {Narative.title} attend encore d'être immortalisé. C'est le dernier rappel avant que la carte disparaisse.`
- Même deeplink que J+1 : `naraapp://narative?id={narativeId}&source=completion_notif`

## Tracking

`COMPLETION_NOTIF_J7_SENT { narative_id, score_at_send }`
`COMPLETION_NOTIF_CONVERTED { narative_id, notif_day, criteria_completed }` si conversion

## Acceptance criteria

- [ ] Push J+7 envoyée uniquement si J+1 déjà envoyée + score encore < 4/5 + pas de conversion
- [ ] Tracking correct
