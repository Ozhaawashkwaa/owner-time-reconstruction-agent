# Premier prompt à donner à Claude Code dans Windsurf

> Une fois le repo cloné et ouvert dans Windsurf, copie-colle ce prompt à Claude Code.

---

## ⚠️ Avant de copier le prompt

Vérifier que dans Windsurf:
- [ ] Le folder `owner-time-reconstruction-agent` est ouvert
- [ ] Tu vois bien `CONTEXT.md` à la racine du folder
- [ ] Claude Code a accès aux MCP connectors (Drive, Gmail, Calendar, ClickUp, Slack)

---

## 📋 Prompt à copier-coller (UNE FOIS)

```
Bonjour Claude Code. Je reprends le développement du projet Owner Time 
Reconstruction Agent. 

PREMIÈRE ACTION: lis attentivement CONTEXT.md à la racine de ce repo. 
C'est le document maître qui te donne tout le contexte nécessaire.

Une fois lu:
1. Confirme-moi en 3-5 phrases ta compréhension du projet
2. Identifie clairement:
   - Ce qui existe déjà et ne doit pas être recréé
   - Ce que tu vas devoir construire
   - Les risques principaux
3. Lance la pré-flight checklist (section "PRÉ-FLIGHT CHECKLIST" du CONTEXT.md):
   - Test des 5 MCP connectors avec un appel minimal pour chacun
   - Test de téléchargement du fichier ActivityWatch DB
   - Test de lecture du Google Sheet
   - Test du Slack Bridge /health endpoint

Pour chaque test, donne le résultat: ✅ OK, ❌ Échec (avec raison), 
ou ⚠️ Partiel (avec détails).

Ne commence PAS à coder avant qu'on ait validé ensemble que tout 
l'environnement fonctionne. Si quelque chose échoue, on debug avant 
d'avancer.

Une fois la pré-flight passée, on commencera par le module 
download_aw_db.py — la fonctionnalité critique de téléchargement du 
fichier .db SQLite ActivityWatch depuis Google Drive.
```

---

## 🎯 Réponse attendue de Claude Code

Tu devrais voir une réponse structurée comme:

```
J'ai lu CONTEXT.md. Voici ma compréhension:

[3-5 phrases résumant le projet]

CE QUI EXISTE DÉJÀ:
- Slack Bridge sur Railway (ne pas toucher)
- Google Sheet state (3 onglets)
- ActivityWatch backups dans Drive
- Repo Slack Bridge séparé

CE QUE JE DOIS CONSTRUIRE:
- 8-10 modules Python dans routine/
- Tests dans tests/
- Le tout orchestré par main.py avec CLI

RISQUES PRINCIPAUX:
- Casser le Slack Bridge déployé
- Mal mapper du temps facturable vers ClickUp
- Dépendances MCP qui échouent

PRÉ-FLIGHT CHECKLIST:

Test 1: Google Drive
[Claude Code fait l'appel]
✅ OK — Listé 3 fichiers récents

Test 2: Gmail
✅ OK — Récupéré dernier email

Test 3: Calendar
✅ OK — Listé événements semaine

Test 4: ClickUp
✅ OK — Workspace name: "Munn & Affiliates"

Test 5: Slack
⚠️ Partiel — Connecteur OK mais pas testé envoi

Test 6: Téléchargement .db ActivityWatch
[Claude Code essaie de télécharger 1mZe1WRh_lusTQotMJvvjMqMzPHKx3NYq]
✅ OK — Fichier de 57 MB dans /tmp/aw_test.db
   ou
❌ Échec — [raison à debug]

Test 7: Google Sheet
✅ OK — Lu les 3 onglets

Test 8: Slack Bridge /health
✅ OK — Returns 200 healthy

[...]

Si tout passe → propose-moi de commencer download_aw_db.py
Si quelque chose échoue → me demande comment debug
```

---

## ⚠️ Si Claude Code répond mal

### Symptôme: Il commence direct à coder sans pré-flight

→ Réponds: "Stop. Tu n'as pas fait la pré-flight checklist. Lis CONTEXT.md 
section 'PRÉ-FLIGHT CHECKLIST' et exécute-la avant de coder."

### Symptôme: Il dit ne pas avoir accès aux MCP

→ Vérifier dans Windsurf settings que les MCP sont bien configurés. 
Voir documentation Windsurf pour ajouter MCP servers.

### Symptôme: Il invente des choses (URL, IDs)

→ Réponds: "Tu hallucines. Toutes les valeurs sont dans config/.env 
ou dans CONTEXT.md. N'invente jamais — demande si tu manques d'info."

### Symptôme: Le téléchargement du .db échoue

→ Le plus probable est un problème d'auth Drive. Demander à Claude Code:
"Quelle est l'erreur exacte? Est-ce un problème d'auth ou de taille de fichier? 
Vérifie le scope OAuth du connecteur Drive."

---

## 🚀 Une fois la pré-flight validée

Prochain prompt:

```
Parfait, l'environnement est prêt. Commençons par routine/download_aw_db.py.

Spécifications:
- Input: target_date (str, format YYYY-MM-DD), output_path (str)
- Logique:
  1. Lister les .db dans le folder Drive AW_DRIVE_FOLDER_ID
  2. Trouver celui dont la date de modification est >= target_date+1 jour
     (parce que les .db sont cumulatifs et nommés par date d'upload)
  3. Si plusieurs candidats, prendre le plus récent
  4. Télécharger via MCP Google Drive vers output_path
  5. Vérifier que le fichier est un valid SQLite avec sqlite3.connect()
- Output: chemin du fichier téléchargé
- Erreurs custom: AWDbNotFoundError, AWDbCorruptedError

Code-le, ajoute le bloc CLI à la fin pour test standalone, 
et lance un test avec --target-date 2026-04-27 --output /tmp/aw_test.db

Quand ça marche, on passe à parse_aw.py.
```

---

**Important:** Avance UN module à la fois. Teste chaque module isolément 
AVANT de passer au suivant. Si un test échoue → debug avant d'avancer.
