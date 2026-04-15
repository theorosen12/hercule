---
stepsCompleted: [1, 2]
inputDocuments: ['_bmad-output/epics/NAR-356-completion-souvenir.md']
session_topic: 'NAR-356 — Complétion du Souvenir'
session_goals: 'Challenger les choix déjà faits et identifier les risques cachés avant le dev'
selected_approach: 'ai-recommended'
techniques_used: ['Assumption Reversal', 'Reverse Brainstorming', 'Six Thinking Hats']
ideas_generated: []
context_file: ''
---

# Brainstorming Session — NAR-356 Complétion du Souvenir

**Facilitateur :** BMad Master
**Participant :** Matthieu
**Date :** 2026-03-18

---

## Session Overview

**Topic :** NAR-356 — Feature de complétion du souvenir (score 0-6, UI checklist, push J+1/J+7)
**Goals :** Challenger les choix de spec existants + identifier les risques cachés avant le dev
**Posture :** Critique constructive, pas de vache sacrée

### Techniques sélectionnées (AI-Recommended)

- **Phase 1 — Assumption Reversal** (deep) : Rendre explicites et retourner les hypothèses implicites de la spec
- **Phase 2 — Reverse Brainstorming** (creative) : Générer des failure modes pour révéler les risques cachés
- **Phase 3 — Six Thinking Hats** (structured) : Catégoriser et prioriser avec Black Hat + Red Hat

### Phase 2 — Reverse Brainstorming : Failure modes identifiés

**[FM #2] — Push J+1 hors contexte émotionnel**
_Concept_: J+1 après un voyage, l'utilisateur gère ses emails et sa valise. L'émotion est encore fraîche mais le contexte mental est ailleurs. "Plus tard" devient jamais.
_Statut_: Risque accepté — la fenêtre J+1 reste la meilleure option disponible. À monitorer via open rate.

**[FM #3] — Composant visible en permanence = bruit → résolu**
_Décision_: Carte repliée par défaut après la première visite. Header affiche le score en replié.

**[FM #4] — Critère "ami invité" = friction sociale forcée → partiellement résolu**
_Décision_: Option "je voyage seul" ajoutée pour valider le critère sans action sociale.

**[FM #8] — Validation du partage non traçable → résolu**
_Décision_: Validation à l'ouverture du share sheet. Simple, cross-platform.

**[FM #9] — Régression du score → acceptée**
_Décision_: Régression autorisée. Score = calcul pur temps réel. Pas de persistance.

**[FM #10] — Badge "Souvenir immortalisé" pendant le voyage → résolu**
_Décision_: Composant date-aware. Wording "Continue comme ça ✨" avant end_date.

### Phase 3 — Six Thinking Hats : Résultats

#### 🖤 Black Hat — Risques résiduels

| Risque | Décision |
|--------|----------|
| Seuil push 4/5 à recalibrer vs /6 original | **Confirmé à 4/5** — pas de push si score ≥ 4/5 |
| Option "je voyage seul" utilisée comme skip | **Accepté** — app de guidage, pas de policing |
| NAR-366 infra push bloquant | **Annulé** — infra existante, NAR-360/363 peuvent démarrer |
| Carte disparaît à J+15 sans avertissement | **Résolu** — notification J+14 ajoutée (NAR-367) |

#### ❤️ Red Hat — Risques ressentis

| Moment | Ressenti | Statut |
|--------|----------|--------|
| J0 première ouverture — 2/5 déjà cochés | Positif — momentum enclenché | ✅ OK |
| Usage récurrent pendant voyage — carte repliée | Neutre à positif — pas de pression | ✅ OK |
| Push J+1 post-voyage — nostalgie + urgence | Fort potentiel — bon timing émotionnel | ✅ OK |
| Critère "Partage" pour utilisateur qui veut rester privé | Friction — pas d'alternative "souvenir privé" | ⚠️ Question ouverte |

#### Nouvelle story identifiée

**NAR-367 — Notification J+14 : avertissement disparition de la carte**
- Trigger : `end_date + 14j`, si score < 5/5
- Deeplink : `naraapp://narative?id={id}&source=completion_notif`
- Copy : à définir avec Théo (ton urgence douce, dernière chance)

---

## Décisions produit prises en session

| # | Décision | Impact spec |
|---|----------|-------------|
| D1 | Critère #4 (descriptions riches) retiré du scope V1 — réintroduit avec la feature vocale | NAR-357, NAR-358, NAR-359 |
| D2 | Score sur /5 au lieu de /6 (5 critères réels) | NAR-357, NAR-358, NAR-360 |
| D3 | Critère #6 (chat IA) remplacé par "Partage ton souvenir" (share sheet natif) | NAR-357, NAR-359, NAR-365 annulé |
| D4 | NAR-365 annulé — bloquant GraphQL AI chat devenu sans objet | NAR-365 |
| D5 | Header replié affiche le score : "✨ Immortalise ce souvenir — 3/5 ›" | NAR-358 |
| D6 | Carte dépliée uniquement à la première visite, repliée par défaut ensuite | NAR-358 |
| D7 | Composant visible 15 jours après end_date puis disparaît | NAR-358 |
| D8 | Critère "inviter un ami" maintenu + option "je voyage seul" pour valider le critère | NAR-358, NAR-359 |
| D9 | Composant date-aware : wording différent avant/après end_date | NAR-357, NAR-358 |
| D10 | Régression autorisée — score calculé en temps réel, pas de persistance | NAR-357 |
| D11 | Critère "Partage" validé à l'ouverture du share sheet (pas au partage effectif) | NAR-357, NAR-364 |

---

## Idées & Insights

### Phase 1 — Assumption Reversal : Résultats

#### Décisions prises

- **[DÉCISION]** Critère #4 (descriptions riches) retiré du scope V1 — sera réintroduit avec la feature de description vocale (Whisper). Le slot reste visible dans l'UI avec label "Bientôt ✨".

#### Hypothèses challengées & Risques identifiés

**[Assumption #1] — Score /6 masque un /4 réel**
_Concept_: Photo de couverture et lieu sont obligatoires à la création d'un Narative — tout le monde arrive automatiquement à 2/6. Le score réel est donc sur 4 critères optionnels, pas 6.
_Risque_: Le seuil de notification `score < 4/6` est trop bas — presque tous les Naratives terminés sans action supplémentaire déclencheront la push J+1. À recalibrer sur la base du /4 réel.

**[Assumption #2] — Sous-titres dynamiques mal calibrés**
_Concept_: La spec prévoit un message pour 0-2/6, mais ce palier n'existe pas en pratique. Le premier message vu par tous les utilisateurs (à 2/6) tombe dans la tranche négative "Tes souvenirs méritent mieux qu'une galerie photo."
_Risque_: Message perçu comme une critique injuste dès le premier regard, alors que l'utilisateur a déjà fait l'effort de créer son Narative avec photo et lieu.

**[Assumption #3] — Endowed Progress Effect conditionné par l'ordre des critères**
_Concept_: L'effet "pied dans la porte" (partir à 2/6) est réel et documenté (Endowed Progress Effect). Mais il est conditionné par un ordre de difficulté croissante — si le critère le plus difficile arrive au milieu, l'effet s'inverse.
_Risque_: L'ordre des critères dans la checklist doit être optimisé par effort croissant, pas par logique de spec.

**[Assumption #4] — Critère #4 conflictuel avec les descriptions obligatoires existantes**
_Concept_: Les descriptions sont obligatoires à la création d'un moment. L'utilisateur qui voit "○ Décris tes moments" pense avoir déjà fait cette tâche. Le critère ≥50 chars est invisible et perçu comme une double peine.
_Décision_: Critère retiré du scope V1, réintroduit avec la feature vocale.

**[Assumption #5] — Mesurabilité des métriques compromise sans retrait du critère #4**
_Concept_: Shipper le critère #4 avant la feature vocale aurait faussé les métriques de succès à 60j — les taux auraient reflété la friction du clavier, pas l'efficacité de la feature.
_Risque résolu par la décision_: Baseline V1 propre sur 5 critères, critère #4 ajouté en V2 avec sa propre mesure.

#### Questions ouvertes (à trancher avec Théo)

- Option "Souvenir privé" pour le critère "Partage ton souvenir" — ou le partage est une valeur non négociable ?
- Wording exact du badge à 5/5 pendant le voyage (avant end_date)
- Copy notification J+14 (NAR-367)
- Comportement "je voyage seul" : valide le critère ✅ ou le retire du score → score max 4/4 ?

#### Score V1 révisé (5 critères)

| # | Critère | Type | Effort |
|---|---------|------|--------|
| 1 | Photo de couverture | Auto | — |
| 2 | Lieu défini | Auto | — |
| 3 | ≥3 moments | Optionnel | Faible |
| 4 | ≥1 ami invité | Optionnel | Moyen |
| 5 | Chat IA complété | Optionnel (conditionnel NAR-365) | Moyen |

Tout le monde démarre à **2/5**. Progress bar et sous-titres à recalibrer sur /5.

---


