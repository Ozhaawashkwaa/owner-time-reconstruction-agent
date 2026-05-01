# ClickUp Workspace Mapping

> Référence des spaces, folders, lists et tâches connus du workspace ClickUp de Franck.
> 
> Ce document est un **point de départ**. Le tab `clickup_mapping` du Google Sheet 
> (ID `1XHrhhoQIbPNVvsH6-kQMcNTa52rtFdNAFtqng4JnxLM`) est la source de vérité 
> évolutive — il apprend des validations de Franck.

---

## Workspace structure

### Spaces principaux

| Space | ID | Type |
|---|---|---|
| AUpoint | `90170935817` | Client |
| Imaginau | `90171476126` | Client |
| MAIA | (à confirmer) | Produit interne / clients |
| AI LAB AGENCY | (à confirmer) | Internal admin |
| MUNNAFF | `4246019` | Holding personal |

---

## AUpoint (Client majeur)

### Structure connue

```
AUpoint (Space 90170935817)
├── Marketing Operations (subcategory 901704917834)
│   ├── Brand Guideline - AUpoint (task 86dx2tjj5)
│   ├── Branding - Site web AUpoint (task 86dx8ppab)
│   ├── Portfolio AUpoint (task 86dx2tq19)
│   └── Website (task 86dx2uarj)
│
├── Analyse-Strategy-Design (subcategory 901703400998)
│   ├── AUpoint Ongoing Strategy (task 86dxv48c9) ← assigné à Franck
│   ├── SEO strategy AUpoint onpage (task 86dyc0fxh)
│   └── [multiples sous-tâches]
│
├── EspoCRM - Planned Deliverables (subcategory 901704790962)
│   ├── AUpoint - CRM (EspoCRM) (task 86dx16t91)
│   └── Update AUpoint DATABASE integration schema (task 86dww69ge)
│
├── SEO (subcategory 901704889601)
│   └── Create meta description (task 86dxtt89w)
│
├── Social Media Assets (subcategory 901705622147)
│   └── Setup AUpoint FB and IG on planable (task 86dxw6q54)
│
└── Projets-Portefeuille (subcategory 901703940566)
```

### Mapping heuristique pour signaux AW

| AW Signal | Mapping suggéré |
|---|---|
| Email avec @aupoint.ca + content "marketing" | Marketing Operations > Website |
| Email avec @aupoint.ca + content "CRM" / "EspoCRM" | EspoCRM > AUpoint - CRM |
| Email avec @aupoint.ca + content "stratégie" | Analyse-Strategy > AUpoint Ongoing Strategy |
| Drive folder "AUpoint" | Selon contenu document |
| ClickUp task page AUpoint | Direct mapping à la task vue |
| Microsoft Teams (meeting AUpoint) | Selon agenda meeting |

---

## Imaginau (Client)

### Structure connue

```
Imaginau (Space 90171476126)
├── SEO (subcategory 901705594487)
│   ├── Aupoint SEO documentation (task 86dy3c3ef)
│   ├── Aupoint - List of competidors (task 86dy2p9xj) ← assigné à Franck
│   └── Review initial SEO Strategy (task 86dxupcjj)
│
├── Wordpress (subcategory 901705048100)
│   └── Landing page layout and Outline (task 86dwk8bq0)
│
├── Wordpress - Phase 2 (subcategory 901705070864)
│   ├── Install wordpress on Bluehost (task 86dxaywve)
│   ├── Email Copy (task 86dxj2ybd)
│   └── Email template for imaginau campaign (task 86dxj2331)
│
└── Email asset & Campaign (subcategory 901705048106)
    └── Create Sender Email (task 86dw5qwtc)
```

---

## MAIA / AI Lab Agency (Produits internes)

### Structure inférée (à valider)

```
MAIA (Space ID à confirmer)
├── AI Studio
│   ├── UX design (task 86e144f6v)
│   └── UI prototype review (task 86e148ycq)
│
├── Solution Design
│   ├── Daily Scrum task (à identifier)
│   └── [tâches développement]
│
├── Project task (list 901703121942) — diverses tâches dev Franck
│   ├── Draft Intent (86dupbcp5)
│   ├── Draft variables (86dupbcrw)
│   ├── Draft workflow home (86dupbcv1)
│   └── [...]
│
└── Product Development (list 901703278308)
    ├── Add 4 most popular INTENTS (86dutfhkf)
    ├── Test with URL within KB (86duun7pe)
    └── [...]
```

### Mapping heuristique

| AW Signal | Mapping suggéré |
|---|---|
| VS Code + repo "lacasse-agent" | MAIA > AI Studio > UX design |
| Daily Scrum 6h30-7h00 + Abdul | MAIA > Solution Design > Daily Scrum |
| Notion + page MAIA | Product Development > [page liée] |
| Cursor + repo "maia-*" | Product Development selon repo |

---

## AI LAB AGENCY (Admin interne)

### Structure inférée

```
AI LAB AGENCY (Space)
└── ADMIN
    ├── 1_Planning & PM (list)
    │   └── Open AI API management (86e144aq8)
    ├── 2.0 Recrutement (list)
    │   └── Post review (86e1445q1)
    └── [autres listes admin]
```

### Mapping heuristique

| AW Signal | Mapping suggéré |
|---|---|
| ClickUp Home / Inbox | ADMIN > 1_Planning & PM |
| Upwork (job posts, messages, interviews) | ADMIN > 2.0 Recrutement |
| Email tri sans contexte client | ADMIN > 1_Planning & PM |
| Gestion équipe (Slack admin, Zoho personnel) | ADMIN selon contexte |

---

## Tasks récurrentes assignées à Franck

Dernière liste vue (à régénérer dynamiquement via `clickup_search` user=`4205656`):

| Task ID | Nom | List/Project |
|---|---|---|
| 86duqmkmb | Develop and refine interaction block and AI conversational prompt | Product Development |
| 86dupbcp5 | Draft Intent | Project task |
| 86dupbcrw | Draft variables | Project task |
| 86duun7pe | Test with URL within KB | Product Development |
| 86dutfhkf | Add 4 most popular INTENTS | Product Development |
| 86dxv48c9 | AUpoint Ongoing Strategy | Analyse-Strategy-Design |
| 86dx3xfwm | Validate chart account selection | 2025 first semester accounting |
| 86dx65nhq | send rational to natalia and LF | Marketing Operations |
| ... | (et plus) | |

---

## Personnes & emails

| Email | Personne | Contexte |
|---|---|---|
| franck@munnandaffiliates.com | Franck Munn | Owner |
| abdulmunimjemal@gmail.com | Abdul | MAIA / Solution Design dev |
| lfrodrigue@aupoint.ca | Louis-François Rodrigue | AUpoint client |
| sbatres@aupoint.ca | Sonia Batres | AUpoint client |
| nfranco@aupoint.ca | Natalia Franco | AUpoint client |
| clavoie@aupoint.ca | Coraly | AUpoint client |
| ivan@team.munn.ai | Ivan Aranaga | Team interne (SEO) |
| gabriel@munnandaffiliates.com | Gabriel Lobo | Team interne (dev) |

---

## Comment ce mapping évolue

À chaque approbation Franck d'un mapping, l'agent doit:

1. INSERT dans Google Sheet tab `clickup_mapping`:
```
app_signal | client_context | work_type | clickup_space | clickup_folder | clickup_list | task_id | task_name | confidence_modifier | last_used | franck_confirmed
```

2. Si même `app_signal` déjà présent + `franck_confirmed=true` → augmenter `confidence_modifier`

3. Au prochain run, l'agent **prioritise** les mappings avec haute confidence et `franck_confirmed=true`.

C'est ce qui permet à l'agent de devenir **plus rapide et plus précis** semaine après semaine.

---

## Workspace ID ClickUp

À identifier dans Claude Code via:
```python
ClickUp:clickup_get_workspace_hierarchy()
```

Résultat à mettre dans `config/.env`:
```bash
CLICKUP_WORKSPACE_ID=<numeric_id>
```
