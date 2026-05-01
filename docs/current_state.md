# Current State (mai 2026)

> Snapshot précis de l'état du projet au moment du handoff vers Claude Code/Windsurf.

---

## ✅ Ce qui fonctionne

### Infrastructure

| Composant | Statut | Détails |
|---|---|---|
| Slack Bridge sur Railway | 🟢 LIVE | URL: voir Railway dashboard |
| GitHub repo Slack Bridge | 🟢 OK | `Ozhaavashkwaa/owner-time-slack-bridge` |
| GitHub repo principal | 🟢 OK | `Ozhaawashkwaa/owner-time-reconstruction-agent` (vide ou en cours) |
| Google Sheet state | 🟢 OK | Schéma 3 onglets en place |
| ActivityWatch Drive backups | 🟢 OK | Sync auto 2h30 EST quotidien |
| MCP connectors (claude.ai) | 🟢 OK | Drive, Gmail, Calendar, ClickUp, Slack tous testés |
| MCP connectors (Claude Code) | ❓ À VALIDER | Premier test à faire |

### Tests confirmés

- ✅ Slack Bridge `/health` répond 200
- ✅ Message dans channel Slack → réponse "No active conversation..." (fallback original fonctionnel)
- ✅ Lecture Google Sheet via MCP
- ✅ Liste fichiers ActivityWatch dans Drive
- ✅ Search ClickUp pour tâches AUpoint
- ✅ List events Calendar pour 27 avril
- ✅ Search Gmail pour 27 avril

---

## ❌ Ce qui ne fonctionne PAS encore

### Code à écrire (priorité ordre)

1. **`routine/download_aw_db.py`** — Code pour télécharger le .db SQLite depuis Drive vers filesystem local Claude Code
2. **`routine/parse_aw.py`** — Code pour parser SQLite et extraire events de target_date
3. **`routine/correlate_signals.py`** — Logique de corrélation cross-source
4. **`routine/map_to_clickup.py`** — Logique de search + fit quality scoring
5. **`routine/main.py`** — Orchestration avec CLI args
6. **`routine/slack_bridge_client.py`** — POST vers Railway
7. **`routine/handle_franck_response.py`** — Cycle de validation
8. **`routine/post_to_clickup.py`** — Création des time entries
9. **`routine/google_sheet_state.py`** — CRUD sur les 3 onglets
10. **`routine/scheduler.py`** — Cron 7h EST

### Tests jamais effectués

- ❌ Téléchargement réel d'un .db ActivityWatch (57 MB) dans Claude Code
- ❌ Parsing SQLite des données ActivityWatch
- ❌ Corrélation cross-source (AW + Gmail + Calendar)
- ❌ Génération message matinal via Anthropic API depuis routine
- ❌ POST de la routine vers Slack Bridge `/morning-message`
- ❌ Création de time entries dans ClickUp via routine
- ❌ Update Google Sheet state depuis routine
- ❌ Cycle complet de validation Franck → APPROVE → ClickUp posting

---

## 🔧 Décisions architecturales prises

### Repos séparés

- **Décision:** Garder le Slack Bridge dans son repo dédié
- **Raison:** Préserver le déploiement Railway. Refactoring = risque inutile.
- **Conséquence:** Le repo principal référence le Slack Bridge via doc + URL Railway, pas via submodule.

### Pas de DB locale

- **Décision:** Tout l'état dans Google Sheet, pas de SQLite/Postgres local
- **Raison:** Routine peut tourner sur n'importe quel environnement (Claude Code local, cloud, etc.) sans setup DB
- **Conséquence:** Le Google Sheet est lu/écrit à chaque run. Possible latency mais acceptable pour usage personnel.

### ClickUp time entries: start + end_time, pas duration

- **Décision:** Toujours utiliser `start` + `end_time` calculés
- **Raison:** Éviter les problèmes de formatage de duration ("1h 30m" vs "1 h 30 m" vs "90m")
- **Source:** Test API ClickUp confirmé fonctionnel avec ce format

### Anthropic API (pas Claude Code natif) pour génération message

- **Décision:** La routine appelle Anthropic API directement pour générer le message Slack via Claude
- **Raison:** Découpler la logique de message generation du runtime Claude Code (portable)
- **Modèle utilisé:** `claude-sonnet-4-20250514` (configurable dans .env)

---

## 🎯 Prochaine action concrète

### Pour Franck (action humaine):

1. Cloner le repo `Ozhaawashkwaa/owner-time-reconstruction-agent` dans `C:\Users\FMunn\repos\`
2. Copier les fichiers du project_kit (que je viens de générer) dans le repo
3. Faire un premier commit
4. Ouvrir le repo dans Windsurf
5. Lancer Claude Code dans Windsurf
6. Donner à Claude Code le prompt initial (voir `scripts/first_prompt.md`)

### Pour Claude Code (dans Windsurf):

1. Lire `CONTEXT.md` en entier
2. Faire la pré-flight checklist (section dans CONTEXT.md)
3. Si tout passe → commencer par `routine/download_aw_db.py` + test
4. Ne pas avancer sans test fonctionnel à chaque étape

---

## 📝 Historique des sessions précédentes

### Session 30 avril (claude.ai)

- Définition des 20 spécifications
- Création du Google Sheet state avec 3 onglets
- Déploiement Slack Bridge sur Railway
- Tests des connecteurs MCP
- Identification du problème: routine pas codée

### Session 1er mai matin (claude.ai)

- Tentatives de modification du Slack Bridge → CASSÉ deux fois
- Restauration de la version originale fonctionnelle
- Décision de migrer le développement vers Claude Code
- Création de ce project kit

### Bugs / leçons

- ⚠️ Modifier le code Slack Bridge en aveugle = casser le déploiement → toujours rollback prêt
- ⚠️ Pas tenter de télécharger des fichiers >5 MB via MCP dans claude.ai (limite tokens contexte)
- ⚠️ Toujours valider le contexte avant de lancer Claude Code (sinon il invente)

---

## 🔮 Roadmap simplifiée

### Cette semaine (Phase 1 — MVP rétroactif)

- [x] Project kit + handoff doc
- [ ] Code routine de base
- [ ] Test bout en bout sur 27 avril en dry-run
- [ ] Test 27 avril → message dans Slack (sans posting ClickUp encore)

### Semaine prochaine (Phase 2 — Intégration complète)

- [ ] Code conversation handler
- [ ] Test cycle complet sur 27 avril
- [ ] Test sur 28, 29, 30 avril (rattrapage)
- [ ] Code ClickUp posting

### Semaine d'après (Phase 3 — Production)

- [ ] Cron scheduler
- [ ] Premier run automatique le matin
- [ ] Itérations sur les corrections de Franck
- [ ] Apprentissage du mapping
