# NAR-367 — [P0] Notification J+14 — Avertissement disparition de la carte

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Todo | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Envoyer une notification à J+14 pour avertir l'utilisateur que la carte de complétion va disparaître le lendemain, afin de créer une dernière fenêtre d'engagement.

## Conditions de déclenchement

```
À end_date + 14j :
  Si score < 5/5 (carte encore active)
→ Envoyer push J+14
```

## Copy

- Titre : *"Ton souvenir est presque complet !"*
- Corps : *"Ajoute tes moments pour le finaliser avant que la carte disparaisse demain."*
- Action : deeplink `naraapp://narative?id={narativeId}&source=completion_notif`

## Acceptance criteria

- [ ] Push envoyée uniquement à end_date + 14j si score < 5/5
- [ ] Deeplink avec `source=completion_notif` (même comportement que J+1/J+7)
- [ ] Copy validée avec Théo avant dev
- [ ] Event `COMPLETION_NOTIF_J14_SENT` émis
