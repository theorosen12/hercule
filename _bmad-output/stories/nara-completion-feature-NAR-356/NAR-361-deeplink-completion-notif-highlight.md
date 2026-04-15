# NAR-361 — [P0] Deep link `?source=completion_notif` — scroll auto + highlight composant

**Epic :** NAR-356 — Complétion du Souvenir
**Statut :** Specified | **Priorité :** Medium (2) | **Assigné :** matthieu.gouverith@gmail.com

---

## Objectif

Quand l'utilisateur arrive sur la page Narative depuis la push notification, le composant "Immortalise ce souvenir" est mis en avant automatiquement.

## Comportement attendu

1. La page Narative charge normalement
2. **Après 600ms** → scroll automatique vers le composant de complétude
3. Le composant reçoit une **animation de pulse** (border colorée, 1 fois)
4. Un **toast** apparaît en haut : *"Voici ce qui manque à ce souvenir ✨"* (disparaît après 3s)

## Implémentation

- Parser le param `source` dans le deep link : `naraapp://narative?id=xxx&source=completion_notif`
- Si `source === 'completion_notif'` → déclencher le comportement ci-dessus
- S'assurer que le composant est déplié (état ouvert) à l'arrivée

## Tracking

`COMPLETION_NOTIF_OPENED { narative_id, notif_day, score_at_open }`

## Acceptance criteria

- [ ] Le param `source` est correctement parsé depuis le deep link
- [ ] Scroll automatique vers le composant après 600ms
- [ ] Animation pulse visible (une seule fois)
- [ ] Toast affiché 3s puis disparaît
- [ ] Composant forcé en état déplié si l'utilisateur l'avait réduit
- [ ] Event `COMPLETION_NOTIF_OPENED` émis
