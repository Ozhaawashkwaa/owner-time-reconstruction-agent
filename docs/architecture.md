# Architecture & Spécifications

## Vision

L'agent doit fonctionner comme un **analyste financier rigoureux**, pas comme un automate paresseux. Il fait le travail de recherche (parsing AW, corrélation signaux, search ClickUp) mais **valide TOUJOURS avec Franck** avant de poster du temps facturable.

## Les 20 Spécifications du Projet

### Capture de signaux (1-5)

1. **ActivityWatch comme source primaire** — Capture passive de toutes les fenêtres, apps, URLs, périodes AFK. Sauvegarde quotidienne automatique vers Google Drive (cron 2h30 EST).

2. **Données cumulatives** — Le fichier .db SQLite est cumulatif. Le fichier du 28 avril contient TOUS les events depuis l'installation, y compris ceux du 27 avril. Toujours utiliser le .db le plus récent ≥ target_date+1.

3. **Multi-watchers ActivityWatch** — Au minimum 3 buckets:
   - `aw-watcher-window` — Fenêtre active (app + titre)
   - `aw-watcher-web` — URL navigateur (si extension installée)
   - `aw-watcher-afk` — Périodes inactives (>3 min sans input)

4. **Connecteurs comme sources secondaires** — Gmail, Calendar, Drive, ClickUp, Slack via MCP. Servent à enrichir/valider les blocs ActivityWatch, pas à les remplacer.

5. **Toujours demander le travail off-computer** — Appels téléphoniques, WhatsApp business, réflexion stratégique, déplacements, etc. AW ne capture pas ça. Question obligatoire avant chaque finalisation.

### Analyse & catégorisation (6-10)

6. **Blocs de temps significatifs** — Regrouper events <5 min consécutifs sur même app/projet. Bloc minimum reconnu = 15 min.

7. **Confidence scoring** — Chaque bloc a un score 0-1:
   - 1.0 — Calendar event direct + AW match
   - 0.9 — App + window title sans ambiguïté (ex: VS Code + repo identifié)
   - 0.7 — Plusieurs signaux concordants
   - <0.5 — Demander clarification à Franck

8. **Corrélation cross-source** — Pour chaque bloc:
   - 2h Gmail → query Gmail pour les emails envoyés/lus pendant ce créneau → identifier client
   - 2h VS Code → window title indique repo → search ClickUp pour tâches liées
   - 1h Zoom → croiser avec Calendar pour identifier participants

9. **Réconciliation explicite des totaux** — Toujours présenter:
   ```
   ActivityWatch total: X heures
   + Off-computer estimé: Y heures (validé par Franck)
   = Total journée: Z heures
   
   Total proposé en time entries: W heures
   Gap: (Z - W) — flagger si > 5 min
   ```

10. **Catégorisation client/projet** — Hiérarchie ClickUp à respecter:
    - Workspace > Space > Folder > List > Task
    - Mapping connu dans `docs/clickup_workspace_mapping.md`

### Mapping ClickUp (11-15)

11. **Search avant create** — Toujours chercher tâches existantes par keywords avant de proposer création nouvelle tâche.

12. **Fit quality scoring** — Pour chaque proposed entry:
    - **Strong fit** — Tâche existe, contexte évident
    - **Partial fit** — Tâche existe mais pourrait être ambiguë
    - **Weak fit** — Aucune tâche pertinente, recommander création

13. **State management dans Google Sheet** — Pas de DB locale persistente. Tout l'état dans le Google Sheet `1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM`:
    - `daily_status` — Statut de chaque journée processée
    - `proposed_entries` — Entrées proposées en attente de validation
    - `clickup_mapping` — Apprentissage continu des mappings

14. **Apprentissage du mapping** — Quand Franck valide ou corrige un mapping, l'enregistrer dans le tab `clickup_mapping` avec `last_used` et `franck_confirmed=true`. Augmenter la confidence pour usages futurs.

15. **Format ClickUp time entries** — Utiliser start + end_time (calculé) pour éviter problèmes de formatage de duration. Format ISO `YYYY-MM-DD HH:MM`.

### Conversation & validation (16-20)

16. **Message matinal conversationnel naturel** — Pas robotique. Voir `docs/morning_message_prompt.md` pour le template. Inclure résumé + questions + propositions + offre de validation.

17. **Approval gate obligatoire** — JAMAIS créer de time entry sans `APPROVE` explicite de Franck. La commande `APPROVE` est sur la dernière version validée des proposed entries.

18. **Gestion des corrections** — Si Franck corrige ("non, ce n'était pas pour client X mais Y"), mettre à jour proposed_entries + tab `clickup_mapping`. Régénérer la table d'approbation.

19. **Commande SKIP** — Franck peut skip une journée entière (`SKIP "raison"`) ou une portion. Logger la raison dans `daily_status`.

20. **Robustesse & error recovery**:
    - Si .db AW pas encore uploadé à 7h → retry à 7h30, 8h, 8h30
    - Si MCP connector échoue → continuer avec les autres + flagger ce qui manque
    - Si Slack Bridge down → retry POST 3x, puis logger erreur visible

---

## Principes architecturaux

### Séparation des responsabilités

```
routine/
├── download_aw_db.py       ← I/O Drive
├── parse_aw.py             ← I/O SQLite
├── correlate_signals.py    ← Logique métier (corrélation)
├── map_to_clickup.py       ← Logique métier (mapping)
├── handle_franck_response.py ← Logique métier (conversation)
├── post_to_clickup.py      ← I/O ClickUp
├── google_sheet_state.py   ← I/O Google Sheet
├── slack_bridge_client.py  ← I/O Slack Bridge HTTP
└── main.py                 ← Orchestration
```

Chaque module est testable isolément.

### Données qui circulent

- **Raw events** (sortie de `parse_aw.py`) — events SQLite parsés
- **Time blocks** (sortie de `correlate_signals.py`) — blocs enrichis avec contexte
- **Proposed entries** (sortie de `map_to_clickup.py`) — propositions formatées pour Franck
- **Validated entries** (sortie de `handle_franck_response.py`) — entrées prêtes pour ClickUp

### Communication routine ↔ Slack Bridge

**Routine → Slack Bridge:** POST `/morning-message`
```json
{
  "date": "2026-04-27",
  "message": "Bonjour Franck! Voici l'analyse...",
  "analysis": {
    "aw_total_minutes": 408,
    "proposed_entries": [...],
    "questions": [...]
  }
}
```

**Slack Bridge → Routine:** webhook ou polling
- Format à définir (voir Phase 2 du dev)

---

## Phase de développement

### Phase 1 (CURRENT) — MVP rétroactif

Objectif: faire fonctionner pour le 27 avril (puis 28, 29, 30) en mode dry-run.

- [x] Setup repo + docs
- [ ] `download_aw_db.py`
- [ ] `parse_aw.py` 
- [ ] `correlate_signals.py` (version basique)
- [ ] `map_to_clickup.py`
- [ ] `main.py` avec --dry-run + --target-date
- [ ] Test bout en bout sur 27 avril

### Phase 2 — Intégration Slack

- [ ] `slack_bridge_client.py` — POST vers Railway
- [ ] Test message matinal → Slack
- [ ] `handle_franck_response.py`
- [ ] Test conversation complète

### Phase 3 — ClickUp posting

- [ ] `post_to_clickup.py`
- [ ] Test création real time entries
- [ ] `google_sheet_state.py` — update state après posting

### Phase 4 — Production

- [ ] `scheduler.py` — cron 7h EST
- [ ] Monitoring + alertes
- [ ] Run quotidien automatique

---

## Anti-patterns à éviter

❌ **Hardcoder les task IDs ClickUp** — Toujours search dynamiquement
❌ **Stocker des secrets en clair** — Toujours .env + .gitignore
❌ **Skipper la validation Franck** — Jamais de création directe sans APPROVE
❌ **Ignorer les conflits** — Si AW total ≠ proposed total, flagger explicitement
❌ **Faire des assumptions silencieuses** — Toujours demander quand ambigu
