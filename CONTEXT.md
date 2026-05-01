# CONTEXT.md — Handoff document pour Claude Code

> **À LIRE EN PREMIER.** Ce document contient tout le contexte nécessaire pour reprendre le développement de ce projet sans devoir relire des heures de discussion antérieure.

---

## 🎯 MISSION DU PROJET

Construire un **Owner Time Reconstruction Agent** qui automatise la capture de temps facturable de Franck (owner de Munn & Affiliates / AI Lab Agency) en analysant les données ActivityWatch + connecteurs (Gmail, Calendar, Drive, ClickUp, Slack) pour générer des entrées de temps précises dans ClickUp.

**Pourquoi:** Franck a souvent 3 jours de retard sur son time tracking. Reconstruire manuellement = 1-2h par jour. L'agent doit faire 90% du travail, Franck valide en quelques minutes.

---

## 🏗️ ARCHITECTURE GLOBALE

```
┌──────────────────────────────────────────────────────────────┐
│  ACTIVITYWATCH (Windows)                                     │
│  - Capture passive activité ordinateur                       │
│  - Auto-upload .db SQLite vers Google Drive (cron 2h30 EST)  │
└───────────────────────┬──────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────────┐
│  CLAUDE CODE ROUTINE (déclenchée 7h00 EST)  ← CE QU'ON CODE  │
│                                                              │
│  1. Télécharge .db ActivityWatch depuis Drive                │
│  2. SQL queries pour extraire events de target_date          │
│  3. Corrèle avec Gmail/Calendar/Drive/ClickUp via MCP        │
│  4. Catégorise blocs de temps (client, projet, tâche)        │
│  5. Mappe vers ClickUp (tâches existantes ou nouvelles)      │
│  6. Calcule réconciliation (AW total + off-computer = day)   │
│  7. Génère message conversationnel naturel                   │
│  8. POST vers Slack Bridge → message dans Slack              │
└───────────────────────┬──────────────────────────────────────┘
                        │ POST /morning-message
                        ▼
┌──────────────────────────────────────────────────────────────┐
│  SLACK BRIDGE (Railway.app)  ← DÉJÀ DÉPLOYÉ, NE PAS TOUCHER  │
│  Repo: Ozhaavashkwaa/owner-time-slack-bridge                 │
│  - Reçoit message matinal de la routine                      │
│  - Post dans channel Slack #time-tracking                    │
│  - Reçoit réponses Franck → relay vers routine               │
│  - Gère commandes APPROVE / SKIP                             │
└───────────────────────┬──────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────────┐
│  FRANCK (dans Slack)                                         │
│  - Lit le message matinal                                    │
│  - Répond aux questions (off-computer, ambiguïtés)           │
│  - APPROVE → entrées créées dans ClickUp                     │
└──────────────────────────────────────────────────────────────┘

         ┌────────────────────────────────────────┐
         │  GOOGLE SHEET STATE                    │
         │  ID: 1XHrhhoQIbPNVvsH6-kQMcNTa52rt...  │
         │  - Tab daily_status                    │
         │  - Tab proposed_entries                │
         │  - Tab clickup_mapping (apprentissage) │
         └────────────────────────────────────────┘
```

---

## ✅ CE QUI EXISTE DÉJÀ (NE PAS RECRÉER)

### Infrastructure déployée et fonctionnelle:

1. **Slack Bridge sur Railway** ✅
   - Repo: `https://github.com/Ozhaavashkwaa/owner-time-slack-bridge`
   - Endpoint: `/slack/events` (reçoit messages Franck), `/morning-message` (reçoit msg de la routine), `/health`
   - Variables d'env configurées dans Railway: `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET`, `ANTHROPIC_API_KEY`, `TIME_TRACKING_CHANNEL`
   - **TEST CONFIRMÉ FONCTIONNEL:** message "hello" dans Slack → réponse "No active time reconstruction conversation..."
   - **NE PAS MODIFIER ce repo sans plan rollback. Voir `docs/slack_bridge_reference.md`**

2. **Google Sheet State Store** ✅
   - ID: `1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM`
   - URL: https://docs.google.com/spreadsheets/d/1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM/edit
   - 3 onglets: `daily_status`, `proposed_entries`, `clickup_mapping`
   - Schéma confirmé conforme aux 20 spécifications

3. **Google Drive — ActivityWatch backups** ✅
   - Folder: `Franck Activity Tracking` (ID: `1f4XFaYCrUDzu5MmUcu_ZVAh4x1hqOujS`)
   - Path: `Shared drives/I_Financial/OPERATIONAL/Franck Activity Tracking/`
   - Pattern de fichiers: `peewee-sqlite.v2_YYYY-MM-DD_HH-MM-SS.db` (~57 MB chacun)
   - **Note importante:** Ces fichiers sont **cumulatifs** — celui du 28 avril contient les events du 27 avril aussi
   - Sauvegarde Microsoft automation: cron 2h30 EST chaque matin

4. **MCP Connectors actifs** ✅ (à valider dans Claude Code)
   - Google Drive (fichiers + ActivityWatch .db)
   - Gmail (correspondance)
   - Google Calendar (meetings)
   - ClickUp (tasks + time tracking)
   - Slack (messaging)

### Documentation/Specs déjà rédigées:

- `docs/architecture.md` — Vision + 20 spécifications du projet
- `docs/agent_behavior_spec.md` — Règles strictes du comportement de l'agent
- `docs/edge_cases.md` — Cas particuliers (Zoho ambigu, off-computer, etc.)
- `docs/morning_message_prompt.md` — Template du message matinal
- `docs/clickup_workspace_mapping.md` — Mapping connu des projets/tâches

---

## ❌ CE QUI EST À CONSTRUIRE (TON TRAVAIL)

### 🔴 P0 — Critique pour test du 27 avril:

1. **`routine/download_aw_db.py`** — Télécharge la dernière .db depuis Drive
   - Input: target_date (ex: "2026-04-27")
   - Logique: trouve le .db dont la date est ≥ target_date+1 (cumulatif), le télécharge localement
   - Output: chemin local du fichier .db

2. **`routine/parse_aw.py`** — Parse SQLite ActivityWatch
   - Input: chemin .db, target_date
   - Logique: SQL queries sur tables `eventmodel` + `bucketmodel`
     - Window watcher (fenêtres actives) → app + titre
     - Web watcher (URLs visitées)
     - AFK watcher (périodes inactives)
   - Output: liste d'events bruts pour la journée + total minutes actives

3. **`routine/correlate_signals.py`** — Croise avec Gmail/Calendar/Drive/ClickUp
   - Input: events bruts + target_date
   - Pour chaque bloc de temps significatif (>15min):
     - Si app=Gmail → query Gmail sur les emails envoyés/reçus pendant ce bloc → identifier client/projet
     - Si app=VS Code/Cursor → identifier repo via window title → corréler avec ClickUp
     - Si app=Notion/Drive → identifier document → corréler avec projet
   - Output: blocs enrichis avec client/projet probables + confidence score

4. **`routine/map_to_clickup.py`** — Trouve les tâches ClickUp correspondantes
   - Pour chaque bloc enrichi:
     - Search ClickUp par keywords (client, projet, type de travail)
     - Évalue fit quality (existing task vs new task needed)
     - Génère recommandation
   - Output: liste de proposed_entries avec task_id ou new_task_recommendation

5. **`routine/main.py`** — Orchestre le tout
   - CLI: `python main.py --target-date 2026-04-27 [--debug] [--dry-run]`
   - Charge config depuis `config/.env`
   - Appelle modules dans l'ordre
   - Génère JSON d'analyse + message Slack
   - POST vers Slack Bridge si pas dry-run
   - Update Google Sheet state

### 🟡 P1 — Pour cycle de validation:

6. **`routine/handle_franck_response.py`** — Gère les réponses de Franck
   - Reçoit conversation continue depuis Slack Bridge
   - Met à jour proposed_entries selon corrections
   - Détecte commande APPROVE → trigger ClickUp posting

7. **`routine/post_to_clickup.py`** — Crée les time entries dans ClickUp
   - Input: proposed_entries approuvés
   - Pour chaque entry: `clickup_add_time_entry` (start, end_time, description, billable)
   - Crée les nouvelles tâches si recommandé
   - Output: rapport succès/échec

8. **`routine/scheduler.py`** — Cron job 7h EST (production)

### 🟢 P2 — Polish:

9. Tests unitaires
10. Logs structurés
11. Error recovery (retry si AW .db pas encore uploadé)

---

## 🧪 PREMIER TEST — RECONSTRUIRE LE 27 AVRIL 2026

**Objectif:** Valider que toute la pipeline fonctionne avec une vraie journée passée.

**Données disponibles pour ce test:**
- ActivityWatch .db: `peewee-sqlite.v2_2026-04-28_17-31-43.db` (Drive ID: `1mZe1WRh_lusTQotMJvvjMqMzPHKx3NYq`) — contient les events du 27 avril
- Calendar: `Daily Scrum - Solution Design` 6:30-7:00 AM avec Abdul
- Gmail: emails AuPoint, Upwork payment notification, etc.
- ClickUp: AUCUNE entrée de temps loggée (c'est ce qu'on doit créer)

**Procédure:**
```bash
# Dans Windsurf / Claude Code
cd C:\Users\FMunn\repos\owner-time-reconstruction-agent
python routine/main.py --target-date 2026-04-27 --debug --dry-run
```

**Attendu:** JSON d'analyse imprimé + message Slack généré (pas envoyé en dry-run)

**Voir détails complets dans `tests/test_plan_2026-04-27.md`**

---

## 🔧 CONFIGURATION

### Variables d'environnement nécessaires (`config/.env`):

```bash
# Anthropic API (pour génération message conversationnel)
ANTHROPIC_API_KEY=sk-ant-...

# Slack Bridge (pour POST le message matinal)
SLACK_BRIDGE_URL=https://[ton-app].railway.app
TIME_TRACKING_CHANNEL=C...  # ID du channel Slack

# Google Sheet state store
STATE_SPREADSHEET_ID=1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM

# ActivityWatch
AW_DRIVE_FOLDER_ID=1f4XFaYCrUDzu5MmUcu_ZVAh4x1hqOujS

# ClickUp (workspace ID auto-détecté via MCP)
# Pas besoin de token manuel - utilise MCP connector
```

**IMPORTANT:** `.env` est dans `.gitignore`. Voir `config/.env.example` pour le template.

---

## 📋 RÈGLES DE COMPORTEMENT CRITIQUES

Voir `docs/agent_behavior_spec.md` pour la liste complète. Les **5 règles non-négociables**:

1. **TOUJOURS demander le travail off-computer** avant de finaliser une journée
2. **JAMAIS créer une entrée ClickUp sans validation explicite** de Franck (APPROVE)
3. **TOUJOURS faire la réconciliation** des totaux (AW + off-computer = day total)
4. **Flagger les ambiguïtés** au lieu de deviner (Zoho, ClickUp meta-work, etc.)
5. **Mapper proprement vers ClickUp** — privilégier tâches existantes, recommander nouvelles tâches si pas de match >70%

---

## 🚦 PRÉ-FLIGHT CHECKLIST AVANT DE CODER

Avant ta première ligne de code, valide ceci:

- [ ] `git clone https://github.com/Ozhaawashkwaa/owner-time-reconstruction-agent.git` réussi
- [ ] Les 5 MCP connectors fonctionnent (test minimal: liste 3 fichiers Drive, dernier email Gmail, etc.)
- [ ] Téléchargement test du fichier `peewee-sqlite.v2_2026-04-28_17-31-43.db` (57 MB) réussit dans `/tmp/` ou similaire
- [ ] `sqlite3` accessible (CLI ou module Python)
- [ ] Lecture test du Google Sheet `1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM` réussit
- [ ] Slack Bridge URL Railway accessible (curl `/health` retourne 200)

Si **n'importe quoi échoue**, **STOP** et reporte avant de coder. On ne devine pas.

---

## 📂 STRUCTURE DU REPO

```
owner-time-reconstruction-agent/
├── CONTEXT.md                          ← TU LIS ÇA EN PREMIER
├── README.md                           ← Vue d'ensemble pour humains
├── .gitignore
│
├── docs/
│   ├── architecture.md                 ← 20 spécifications + vision
│   ├── agent_behavior_spec.md          ← Règles de l'agent
│   ├── edge_cases.md                   ← Cas particuliers
│   ├── morning_message_prompt.md       ← Template message matinal
│   ├── clickup_workspace_mapping.md    ← Mapping connu
│   ├── current_state.md                ← Statut actuel détaillé
│   └── slack_bridge_reference.md       ← Doc du Slack Bridge externe
│
├── routine/                            ← LE CODE PRINCIPAL À ÉCRIRE
│   ├── main.py                         ← Entry point CLI
│   ├── download_aw_db.py
│   ├── parse_aw.py
│   ├── correlate_signals.py
│   ├── map_to_clickup.py
│   ├── handle_franck_response.py
│   ├── post_to_clickup.py
│   ├── scheduler.py
│   ├── requirements.txt
│   └── README.md
│
├── tests/
│   ├── test_plan_2026-04-27.md         ← Plan de test détaillé
│   ├── test_aw_parsing.py              ← À écrire
│   └── fixtures/                       ← Sample data pour tests
│
├── config/
│   ├── .env.example                    ← Template variables
│   └── settings.py                     ← Config Python
│
└── scripts/
    ├── setup.md                        ← Guide installation locale
    └── deploy_to_claude_code.md        ← Guide déploiement routine
```

---

## 🔗 LIENS UTILES

- **Repo principal (ce projet):** https://github.com/Ozhaawashkwaa/owner-time-reconstruction-agent
- **Repo Slack Bridge (séparé):** https://github.com/Ozhaavashkwaa/owner-time-slack-bridge
- **Google Sheet state:** https://docs.google.com/spreadsheets/d/1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM/edit
- **Google Drive ActivityWatch folder:** https://drive.google.com/drive/folders/1f4XFaYCrUDzu5MmUcu_ZVAh4x1hqOujS
- **Railway dashboard (Slack Bridge):** [À ajouter par Franck]

---

## 💡 PROCHAINE ACTION IMMÉDIATE

Une fois ce repo cloné dans Windsurf et CONTEXT.md lu:

1. Lis `docs/architecture.md` (10 min) pour comprendre les 20 spécifications
2. Lis `docs/agent_behavior_spec.md` (5 min) pour les règles
3. Lis `tests/test_plan_2026-04-27.md` (5 min) pour le test cible
4. Lance la pré-flight checklist (section ci-dessus)
5. Si tout passe → commence par `routine/download_aw_db.py`
6. Une fois le téléchargement validé → `routine/parse_aw.py` (le plus critique)
7. **Ne va pas plus loin sans tester chaque module isolément**

---

**Dernière mise à jour:** 2026-05-01
**Status:** 🟡 Setup en cours — code routine à écrire
**Owner:** Franck Munn (franck@munnandaffiliates.com)
