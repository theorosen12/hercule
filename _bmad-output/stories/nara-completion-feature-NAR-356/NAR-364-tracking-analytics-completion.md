# NAR-364 — [P1] Tracking analytics — Events COMPLETION_***

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Backlog | **Priorité :** Medium (3) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Implémenter tous les events de tracking nécessaires pour mesurer l'impact de la feature et itérer.

## Events à implémenter

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

## Métriques de succès à mesurer

- % Naratives score ≥3/5 (G1)
- Sessions "revisit" +25% (G2)
- Open rate notif ≥35% (G3)
- Conversion post-notif ≥20% (G4)

## Acceptance criteria

- [ ] Les 11 events sont implémentés
- [ ] Les payloads sont conformes au tableau ci-dessus
- [ ] `COMPLETION_SCORE_VIEWED` utilise une logique d'impression (visible ≥ 1s dans le viewport)
- [ ] `COMPLETION_NOTIF_CONVERTED` est correctement attribué à la notif (fenêtre de 24h)
