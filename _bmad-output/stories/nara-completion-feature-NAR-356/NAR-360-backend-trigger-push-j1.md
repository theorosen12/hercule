# NAR-360 — [P0] Backend — Trigger push notification J+1 après end_date (score < 4/5)

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Envoyer une push notification aux propriétaires de Naratives terminés dont le score est inférieur à 4/5, 24h après la end_date.

## Logique de déclenchement

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

## Copy notifications

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

## Tracking

- `COMPLETION_NOTIF_J1_SENT { narative_id, score_at_send }`
- `COMPLETION_NOTIF_J7_SENT { narative_id, score_at_send }`
- `COMPLETION_NOTIF_J14_SENT { narative_id, score_at_send }`

## Acceptance criteria

- [ ] Trigger déclenché exactement à end_date + 24h
- [ ] Condition score < 4/5 vérifiée au moment du déclenchement (pas au moment de la création)
- [ ] Pas de push si score ≥ 4/5 au moment du trigger
- [ ] Jamais plus de 3 pushes par Narative
- [ ] Copy dynamique avec Narative.title et compte de critères manquants sur /5
- [ ] Deeplink inclus dans la notification
