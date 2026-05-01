# Setup Guide — Installation locale dans Windsurf

> Étapes pour cloner le repo et démarrer le développement avec Claude Code.

---

## Prérequis

- Windows 10/11 (Franck est sur Windows)
- Windsurf installé (avec Claude Code)
- Python 3.10+ installé
- Git installé
- Compte GitHub avec accès au repo `Ozhaawashkwaa/owner-time-reconstruction-agent`

---

## Étape 1 — Cloner le repo

Ouvrir PowerShell ou Terminal:

```powershell
cd C:\Users\FMunn\repos
git clone https://github.com/Ozhaawashkwaa/owner-time-reconstruction-agent.git
cd owner-time-reconstruction-agent
```

---

## Étape 2 — Importer le project kit (UNE FOIS)

Si le repo est vide ou contient juste un README, copier les fichiers du **project kit** que Claude (claude.ai) a généré:

1. Télécharger le project_kit.zip (ou les fichiers individuellement)
2. Décompresser dans `C:\Users\FMunn\repos\owner-time-reconstruction-agent\`
3. Vérifier que la structure est:

```
owner-time-reconstruction-agent/
├── CONTEXT.md
├── README.md
├── .gitignore
├── docs/
├── routine/
├── tests/
├── config/
└── scripts/
```

4. Premier commit:

```powershell
git add .
git commit -m "Initial project kit — handoff from claude.ai to Claude Code"
git push origin main
```

---

## Étape 3 — Setup Python environment

```powershell
# Optionnel mais recommandé: virtualenv
python -m venv venv
.\venv\Scripts\Activate.ps1

# Installer les dépendances
pip install -r routine\requirements.txt
```

---

## Étape 4 — Configuration

```powershell
# Copier le template .env
copy config\.env.example config\.env

# Ouvrir .env dans Notepad ou VS Code/Windsurf
notepad config\.env
```

**Valeurs à remplir:**
- `ANTHROPIC_API_KEY` — Récupérer depuis console.anthropic.com
- `SLACK_BRIDGE_URL` — Aller dans Railway dashboard, copier l'URL publique de ton app
- `TIME_TRACKING_CHANNEL` — Dans Slack, clic droit sur le channel #time-tracking → "Copy link" → l'ID est dans l'URL (format `C...`)
- `SLACK_BOT_TOKEN` — Dans Slack App admin → OAuth & Permissions → Bot User OAuth Token
- `CLICKUP_WORKSPACE_ID` — Sera récupéré dynamiquement via Claude Code, mais peut être hardcodé pour simplicité

---

## Étape 5 — Ouvrir dans Windsurf

```powershell
# Si Windsurf a un CLI:
windsurf .

# Sinon: ouvrir Windsurf, File > Open Folder, sélectionner le dossier
```

---

## Étape 6 — Lancer Claude Code

Dans Windsurf, ouvrir Claude Code (selon ton workflow habituel).

**Premier prompt à lui donner:** voir `scripts/first_prompt.md`

---

## Vérifier les MCP connectors

Avant de coder, valider que Claude Code voit bien:
- ✅ Google Drive
- ✅ Gmail
- ✅ Google Calendar
- ✅ ClickUp
- ✅ Slack

Si certains manquent → les configurer dans Claude Code (procédure dépend de ta config Windsurf).

---

## Troubleshooting

### Erreur d'authentification Drive

→ MCP connector Drive expiré. Reconnect dans Claude Code settings.

### Python pas trouvé

```powershell
# Vérifier installation
python --version

# Si pas installé: télécharger depuis python.org
```

### Erreur git push

→ Vérifier credentials GitHub (token PAT ou SSH key)

```powershell
git config --global user.name "Franck Munn"
git config --global user.email "franck@munnandaffiliates.com"
```

### sqlite3 ne marche pas

→ Devrait être inclus avec Python. Vérifier:
```python
python -c "import sqlite3; print(sqlite3.sqlite_version)"
```
