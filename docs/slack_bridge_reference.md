# Slack Bridge Reference

> Le Slack Bridge est dans un **repo séparé** déjà déployé. Ce document explique 
> ce qu'il fait et comment la routine principale doit communiquer avec lui.
> 
> **NE PAS modifier le repo Slack Bridge depuis ce projet sans plan rollback.**

---

## Repo & déploiement

- **Repo:** https://github.com/Ozhaavashkwaa/owner-time-slack-bridge
- **Plateforme:** Railway.app
- **URL publique:** [À ajouter par Franck dans config/.env]
- **Statut:** 🟢 LIVE et fonctionnel

## Endpoints exposés

### `GET /health`

Health check. Retourne 200 si l'app tourne.

```bash
curl https://[ton-app].railway.app/health
# Response: {"status": "healthy", ...}
```

### `POST /slack/events`

Webhook configuré dans Slack App pour recevoir les events:
- URL verification challenge
- Messages dans le channel `TIME_TRACKING_CHANNEL`

**Tu n'appelles pas ça depuis la routine.** Slack l'appelle quand Franck poste.

### `POST /morning-message`

**C'est là que la routine envoie le message matinal.**

Payload attendu (à confirmer dans le code Slack Bridge actuel):
```json
{
  "date": "2026-04-27",
  "message": "Bonjour Franck...\n\n[message complet formaté markdown]",
  "analysis": {
    "aw_total_minutes": 408,
    "off_computer_minutes": null,
    "proposed_total_minutes": 0,
    "time_blocks": [...],
    "proposed_entries": [...],
    "questions": [...]
  }
}
```

Le Slack Bridge:
1. Stocke `analysis` dans `conversation_state[date]` 
2. Post `message` dans le channel Slack
3. Enregistre que la conversation est active

## Variables d'env du Slack Bridge (Railway)

Configurées dans Railway dashboard:
- `SLACK_BOT_TOKEN` — `xoxb-...`
- `SLACK_SIGNING_SECRET`
- `ANTHROPIC_API_KEY` — pour appels Claude depuis le bridge
- `TIME_TRACKING_CHANNEL` — ID du channel Slack (ex: `C091...`)

## Communication routine → Slack Bridge

Dans `routine/slack_bridge_client.py`:

```python
import os
import requests

SLACK_BRIDGE_URL = os.environ['SLACK_BRIDGE_URL']

def post_morning_message(date: str, message: str, analysis: dict) -> dict:
    """Envoie le message matinal vers le Slack Bridge.
    
    Le Slack Bridge va le poster dans Slack et stocker l'état de conversation.
    """
    response = requests.post(
        f"{SLACK_BRIDGE_URL}/morning-message",
        json={
            "date": date,
            "message": message,
            "analysis": analysis
        },
        timeout=30
    )
    response.raise_for_status()
    return response.json()
```

## Communication Slack Bridge → routine

**À CLARIFIER pendant le développement.** Plusieurs options:

### Option A: Polling (simple)

La routine ne tourne pas en continu. Quand on a besoin de gérer une réponse Franck:
- Le Slack Bridge stocke les messages en attente
- La routine peut être déclenchée manuellement ou par cron pour vérifier
- Endpoint à ajouter dans le bridge: `GET /pending-messages`

### Option B: Webhook depuis bridge → routine (avancé)

Si la routine est hostée quelque part avec endpoint public:
- Le Slack Bridge fait un POST vers la routine quand un message arrive
- Plus reactif mais nécessite hosting de la routine

### Option C: Slack Bridge fait tout le cycle conversationnel (recommandé)

**Le plus simple pour MVP:**
- La routine envoie le message matinal initial
- Le Slack Bridge gère TOUTE la conversation suivante (Franck répond, bridge appelle Anthropic API directement, met à jour state)
- Quand Franck dit `APPROVE`, le bridge fait un appel à un endpoint de la routine pour créer les entries ClickUp
- Endpoint à ajouter dans le bridge → routine: `POST /create-clickup-entries`

**Décision à prendre dans Phase 2 du dev.** Pour Phase 1 (MVP rétroactif), 
on fait juste:
- Routine génère message
- Routine POST vers `/morning-message`
- Franck lit dans Slack
- Pour la conversation, on traitera dans Phase 2

---

## Modifications futures du Slack Bridge

Si on a besoin de modifier le bridge (ajouter endpoints, changer logique):

### Procédure safe

1. **Branche feature:** `git checkout -b feature/new-endpoint`
2. **Test local:** `python app.py` (pas de Slack signature → modifier temporairement)
3. **Deploy preview Railway** (si possible) ou dans un service séparé pour test
4. **Test exhaustif** sur l'env de preview
5. **Merge vers main** seulement quand tout marche
6. **Monitor Railway logs** après deploy

### Plan rollback

Le Railway garde l'historique des déploiements. En cas de problème:
1. Aller dans Railway dashboard → Deployments
2. Trouver le dernier déploiement qui marchait
3. Click "Redeploy" sur celui-là

---

## Variables à ajouter dans config/.env de la routine

```bash
# Pour communiquer avec le Slack Bridge
SLACK_BRIDGE_URL=https://[ton-app].railway.app

# Pour appel direct Slack si besoin (rare — préférer passer par le bridge)
SLACK_BOT_TOKEN=xoxb-...
TIME_TRACKING_CHANNEL=C091...
```
