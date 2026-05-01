# Edge Cases & Patterns

> Cas particuliers que l'agent doit reconnaître et gérer correctement.

---

## Apps et signaux ambigus

### Zoho (CRM)

**Pattern:** Franck utilise Zoho pour différents clients selon les projets.

**Comportement:** **TOUJOURS demander** de quel contexte il s'agit. Ne jamais assumer.

```
"⚠️ Détecté 1h12 sur Zoho entre 14h-15h12. Pour quel client/projet?
- Personnel (CRM Munn & Affiliates)
- AUpoint
- Imaginau
- Autre"
```

### ClickUp

**Pattern:** L'usage de ClickUp lui-même est ambigu — c'est soit du **meta-work** (planning, gestion tâches) soit du **travail réel** (commentaires, écriture spec sur une tâche client).

**Heuristique:**
- ClickUp avec `task/`+ID dans URL → probablement travail spécifique
- ClickUp Home / Inbox / Notifications → meta-work (admin/PM)

**Si ambigu:**
```
"45 min ClickUp entre 9h-9h45. C'était:
- Planning/admin de la journée?
- Travail sur une tâche spécifique? (laquelle?)"
```

### Notion

**Pattern:** Plusieurs workspaces Notion (perso, MAIA, clients).

**Heuristique:** Window title contient souvent le nom du workspace ou page → identifier d'abord, demander si pas clair.

### Email générique

Si bloc Gmail >30 min sans signal client clair (pas d'email entrant/sortant identifié):
```
"32 min sur Gmail entre 11h-11h32 sans email business clair identifié.
Travail sur quel client, ou tri/admin?"
```

### Recherches Google / Browser

Si beaucoup de browsing sans contexte projet:
- Si lié à un repo Github → probablement dev/recherche tech
- Si lié à doc client → contexte client
- Sinon → demander

---

## Patterns temporels

### Daily Scrum (récurrent)

**Détection:** Calendar event "Daily Scrum" + window app Meet/Zoom à l'heure du scrum.

**Mapping par défaut:** Voir `clickup_workspace_mapping.md` — généralement:
- Si avec Abdul → MAIA Solution Design / Daily Scrum
- Si avec autre équipe → identifier projet

**Toujours valider** la première fois pour la journée, puis mémoriser le pattern.

### Meeting prep

**Pattern:** 30-60 min avant un Calendar event = prep meeting. Souvent:
- Lecture documents Drive du client
- Review notes ClickUp
- Préparation slides/agenda

**Mapping:** Même client/projet que le meeting qui suit.

### Déplacements

**Pattern:** Long bloc AFK (>1h) en mileu de journée + Calendar event lieu différent.

**Comportement:** Toujours demander si c'était travail (transit pour meeting client) ou personnel.

### Pauses non capturées

**Pattern:** Trou de >15 min entre deux blocs AW actifs, pas d'AFK enregistré.

**Comportement:** Demander si c'était:
- Pause repas
- Pause déconnectée (téléphone, sortie, café)
- Travail off-computer (ex: réflexion en marchant, lecture papier)

---

## Patterns de projets clients

### AUpoint

**Pattern:** Client majeur avec plusieurs sous-projets:
- Marketing Operations
- EspoCRM
- SEO
- Site web

**Indicateurs:**
- Gmail avec @aupoint.ca
- Drive folder "AUpoint" 
- ClickUp space dédié (ID: `90170935817`)
- Window titles contenant "AUpoint" ou "AU point"

**Heuristique de mapping:**
- Si email AUpoint + work web → Site web AUpoint ou SEO selon contexte
- Si crm/database → EspoCRM AUpoint
- Si marketing/social media → Marketing Operations

### MAIA / AI Lab Agency

**Pattern:** Développement produit interne (lacasse-agent, AI lab tooling, etc.)

**Indicateurs:**
- VS Code/Cursor + repos `lacasse-agent`, `maia-*`, `ai-lab-*`
- ClickUp space MAIA
- Daily scrums avec Abdul

**Heuristique:**
- Coding sessions → tâche AI Studio appropriée
- Daily Scrum → Team Alignment (recurring)

### Imaginau

**Pattern:** Client SEO/Marketing.

**Indicateurs:**
- Email avec @imaginau.* 
- ClickUp space Imaginau
- Travail sur SEO docs, content strategy

### Lacasse

**Pattern:** Client tech, projet AI agent.

**Indicateurs:**
- Repo `lacasse-agent` dans VS Code
- Email avec stakeholders Lacasse

---

## Patterns de travail

### Coding intensif

**Signal AW:** VS Code/Cursor + repo + window changes faibles + active typing.

**Bloc:** Souvent 1-3h continues.

**Mapping:** Identifier projet via repo name → ClickUp task de type "develop X feature".

### Recherche/exploration

**Signal AW:** Browser + multiples onglets/recherches + lecture docs.

**Bloc:** Souvent fragmenté.

**Mapping:** Souvent "Research X" ou phase initiale d'une tâche.

### Communication (emails, Slack, Teams)

**Signal AW:** Mix Gmail/Slack/Teams + window switches fréquents.

**Bloc:** Souvent fragmenté en petites sessions de 5-15 min.

**Mapping:** Identifier les threads principaux → mapper au client correspondant.

### Réunions

**Signal AW:** Zoom/Meet/Teams full-screen ou large window pendant durée Calendar.

**Bloc:** Match avec Calendar event.

**Mapping:** Tâche meeting-spécifique si existante, sinon créer ou mapper à projet parent.

### Admin / Meta-work

**Signal AW:** ClickUp Home, Email tri, Drive organisation, Notion personal.

**Bloc:** Souvent en début/fin de journée.

**Mapping:** AI LAB AGENCY > ADMIN > 1_Planning & PM ou similaire.

---

## Cas particuliers Upwork

⚠️ **Important:** Ne PAS confondre les notifications Upwork.

**Ce qui CONCERNE l'agent:**
- Job posts créés (recrutement)
- Zoom meetings dans Upwork (interviews avec freelancers)
- Messages Upwork (gestion des freelancers en cours)
- Time spent dans plateforme pour gestion équipe

**Mapping:** AI LAB AGENCY > ADMIN > 2.0 Recrutement (généralement).

**Ce qui NE CONCERNE PAS l'agent:**
- Notifications de paiement automatique aux freelancers
- Emails "Payment Received" — ce sont des honoraires de freelancers, pas le travail de Franck
- Reports de revenue freelancers

L'agent doit **filtrer** ces signaux et ne PAS les utiliser comme indicateurs de travail facturable.

---

## Données manquantes ou conflictuelles

### Pas de .db ActivityWatch pour la date

→ Voir `agent_behavior_spec.md` section "Cas d'erreur"

### Données AW incohérentes

**Symptôme:** AW total << temps de travail évident (ex: 1h sur AW mais 5 réunions Calendar de 1h chacune).

**Cause probable:** ActivityWatch pas en marche, ou gros oubli de Franck d'allumer son ordi.

**Comportement:** Flagger explicitement et demander à Franck de confirmer manuellement les blocs.

### Conflit signal AW vs Calendar

**Exemple:** Calendar dit "Meeting 14h-15h avec X" mais AW montre coding sur projet Y.

**Cause possible:**
- Meeting cancelé sans update Calendar
- Multi-tasking pendant meeting
- Meeting effectivement passé mais sur autre device

**Comportement:** Présenter les deux signaux à Franck pour clarification.

```
"⚠️ Conflit 14h-15h:
- Calendar: 'Sync avec Marcel'
- ActivityWatch: VS Code lacasse-agent (45 min actif)

C'était lequel? (Le meeting a-t-il eu lieu?)"
```

### Multi-projets en parallèle

**Symptôme:** Window switches très fréquents entre apps de différents projets.

**Heuristique:** Si <5 min par switch, considérer que c'est multi-tasking inefficace → demander si garder un seul mapping ou splitter.
