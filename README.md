# Owner Time Reconstruction Agent

> Agent automatisé qui reconstitue le temps de travail facturable de Franck à partir de signaux ActivityWatch + connecteurs (Gmail, Calendar, Drive, ClickUp), génère des propositions de time entries dans ClickUp, et discute avec Franck dans Slack pour validation.

## 🎯 Le Problème

Franck (owner d'AI Lab Agency / Munn & Affiliates) a chroniquement 3 jours de retard sur son time tracking ClickUp. Reconstruire manuellement = 1-2h par jour. Inacceptable.

## 💡 La Solution

Un agent qui:
1. Capture passivement l'activité ordinateur (ActivityWatch déjà installé)
2. Chaque matin à 7h, analyse la journée précédente avec tous les signaux disponibles
3. Propose des time entries dans Slack avec questions ciblées (off-computer? ambiguïtés?)
4. Sur APPROVE → crée les entrées dans ClickUp directement

## 🏗️ Architecture

Voir [`CONTEXT.md`](./CONTEXT.md) pour les détails complets.

```
ActivityWatch → Drive (cumulative .db) 
                    │
                    ▼
            Claude Code Routine (7h EST cron)
                    │
                    ▼
         Slack Bridge (Railway) ─→ Slack channel
                    │                    │
                    ▼                    ▼
            ClickUp (time entries)   Franck (validation)
```

## 📦 Composants

| Composant | Repo | Statut |
|---|---|---|
| **Routine principale** | (ici) `Ozhaawashkwaa/owner-time-reconstruction-agent` | 🔧 En développement |
| **Slack Bridge** | `Ozhaavashkwaa/owner-time-slack-bridge` | ✅ Déployé sur Railway |
| **Google Sheet state** | (lien dans CONTEXT.md) | ✅ Structure en place |

## 🚀 Quick Start (pour Claude Code dans Windsurf)

```bash
# 1. Cloner
git clone https://github.com/Ozhaawashkwaa/owner-time-reconstruction-agent.git
cd owner-time-reconstruction-agent

# 2. Lire le contexte
cat CONTEXT.md

# 3. Setup environnement
cp config/.env.example config/.env
# Remplir les valeurs dans .env

# 4. Installer dépendances
pip install -r routine/requirements.txt

# 5. Premier test (dry-run sur le 27 avril)
python routine/main.py --target-date 2026-04-27 --debug --dry-run
```

## 📚 Documentation

- [`CONTEXT.md`](./CONTEXT.md) — Document maître pour reprise du projet
- [`docs/architecture.md`](./docs/architecture.md) — 20 spécifications + vision technique
- [`docs/agent_behavior_spec.md`](./docs/agent_behavior_spec.md) — Règles de comportement
- [`docs/edge_cases.md`](./docs/edge_cases.md) — Cas particuliers à gérer
- [`docs/clickup_workspace_mapping.md`](./docs/clickup_workspace_mapping.md) — Mapping projets connus
- [`tests/test_plan_2026-04-27.md`](./tests/test_plan_2026-04-27.md) — Premier test de bout en bout

## 🔒 Secrets & Sécurité

- `config/.env` est dans `.gitignore`
- Tokens (Slack, Anthropic) configurés dans Railway pour le Slack Bridge
- MCP connectors gèrent l'auth pour Drive/Gmail/Calendar/ClickUp/Slack

## 👤 Owner

Franck Munn — franck@munnandaffiliates.com
