# Morning Message Prompt Template

> Template pour générer le message matinal naturel et conversationnel à poster dans Slack.

---

## Principe

Le message doit lire comme si un assistant humain compétent te récapitulait la journée d'hier. Pas comme un dump JSON. Pas comme un robot. Direct mais chaleureux.

---

## Template (initial — avant validation)

```
Bonjour Franck 👋

Voici l'analyse de ta journée d'hier (lundi 27 avril 2026).

📊 Vue d'ensemble
─────────────────
ActivityWatch a capté 6h48 d'activité active. 
Voici comment elle se décompose:

🕐 6h30-7h00 | Daily Scrum - Solution Design (30 min)
  Calendar: scrum récurrent avec Abdul
  AW: Google Meet
  Mapping suggéré: MAIA > Solution Design > Daily Scrum
  Fit: Strong ⭐⭐⭐⭐

🕘 9h15-11h45 | Coding intensif (2h30)
  AW: VS Code + repo "lacasse-agent" + active typing
  Pas de meeting Calendar pendant cette plage
  Mapping suggéré: MAIA > AI Studio > UX design (existante)
  Fit: Partial ⭐⭐⭐ — peut-être nouvelle tâche dev?

🕑 14h00-15h45 | Gmail + AUpoint (1h45)
  AW: Gmail principalement, Notion ponctuellement
  Gmail: 12 emails échangés avec lfrodrigue@aupoint.ca
  Calendar: invitation reçue pour meeting AUpoint+Franck (Teams)
  Mapping suggéré: AUpoint > Marketing Operations
  Fit: Strong ⭐⭐⭐⭐

[... etc pour chaque bloc significatif]

❓ Questions pour finaliser
──────────────────────────
1. Y a-t-il eu du travail hors-ordinateur? (appels, WhatsApp business, 
   réunions en personne, réflexion stratégique...)

2. ⚠️ Le bloc 11h45-13h30 (1h45 inactif) — c'était pause repas ou autre chose?

3. ⚠️ J'ai vu 23 min de "untitled" dans VS Code à 16h12 — sur quel projet 
   tu travaillais?

📊 Réconciliation préliminaire
──────────────────────────────
ActivityWatch (actif):     6.8 h
+ Off-computer (estimé):   ? h     [À CONFIRMER]
─────────────────
= Total journée:           ? h

Total dans entries proposées: 6.5 h (en attente questions)

Réponds-moi avec tes clarifications, puis je te ferai une 
proposition finale d'entrées ClickUp à approuver.
```

---

## Template (après validation des questions — table d'approbation)

```
Parfait, voici la proposition finale ✅

Total: 7h45 (6h48 ActivityWatch + 57 min off-computer)

| # | Heure       | Durée | Description                        | ClickUp                          |
|---|-------------|-------|------------------------------------|----------------------------------|
| 1 | 6:30-7:00   | 30m   | Daily Scrum avec Abdul             | MAIA > Solution Design > Scrum   |
| 2 | 9:15-11:45  | 2h30  | Dev lacasse-agent UX               | MAIA > AI Studio > UX design     |
| 3 | 13:30-14:00 | 30m   | Appel client (off-computer)        | AUpoint > Strategy               |
| 4 | 14:00-15:45 | 1h45  | Email coordination AUpoint         | AUpoint > Marketing Operations   |
| 5 | 16:00-17:00 | 1h    | Recherche concurrentielle          | Imaginau > SEO > Concurrents     |
| 6 | 17:00-17:30 | 30m   | Admin/planning fin journée         | AI LAB > ADMIN > 1_Planning      |

Réponds APPROVE pour créer ces 6 entrées dans ClickUp.
Réponds avec tes corrections si quelque chose ne va pas.
```

---

## Template (post-approbation)

```
✅ 6/6 entrées créées dans ClickUp pour le 27 avril.

Détails:
- Total tracké: 7h45
- 5 entrées sur tâches existantes
- 1 nouvelle tâche créée: "AUpoint - Strategy preparation"

Tout est dans le tab "proposed_entries" du Google Sheet pour audit.

Bonne journée! 🚀
```

---

## Template (cas d'erreur)

### Données AW manquantes

```
⚠️ Bonjour Franck. Je n'ai pas pu reconstruire le 27 avril:

Le fichier ActivityWatch n'est pas dans Drive. Dernier upload: 26 avril 02h30.

Possible que:
- Ton ordi était hors ligne (pas de sync)
- ActivityWatch ne tournait pas
- Problème avec la sauvegarde automatique

Je réessaie à 9h. Si toujours absent, on devra reconstituer la journée 
manuellement (réponds-moi avec ce que tu te souviens).
```

### Connecteur défaillant

```
Bonjour Franck. J'ai analysé le 27 avril mais avec des limitations:

✅ ActivityWatch: OK
✅ Calendar: OK
❌ Gmail: pas accessible (auth expirée?)
✅ ClickUp: OK

Du coup, les blocs Gmail (~2h total) seront moins précis. Je te propose 
quand même une analyse — tu pourras corriger les mappings si besoin.

[suite du message normal...]
```

---

## Système de variables pour le prompt LLM

Quand on appelle Claude API pour générer le message, le prompt système doit inclure:

```
VARIABLES:
- {target_date}: 2026-04-27
- {day_name_fr}: lundi
- {aw_total_minutes}: 408
- {aw_total_hours}: 6h48
- {time_blocks}: [array of enriched blocks]
- {questions}: [array of questions to ask]
- {proposed_entries}: [array of entries with task_id, fit_quality]
- {reconciliation}: {aw, off_computer, proposed, gap}
- {phase}: "initial" | "after_validation" | "approved" | "error"

INSTRUCTION:
Génère le message Slack matinal en français selon le template approprié 
à la phase. Suit STRICTEMENT les règles de agent_behavior_spec.md.
Ne hallucine pas de données. Si une donnée manque, dis-le.
```

---

## Style guide

### À FAIRE
- Tutoyer Franck
- Direct, factuel
- Émojis avec parcimonie (headers de section, statuts ✅⚠️❌)
- Tables markdown pour les propositions
- Citer la source des signaux (AW, Calendar, Gmail)

### À ÉVITER
- "Je serais ravi de..." / "C'est avec plaisir que..."
- "Excellente question!"
- Trop de variations de couleur/format
- Phrases longues quand une courte suffit
- Hallucinations / suppositions présentées comme des faits
