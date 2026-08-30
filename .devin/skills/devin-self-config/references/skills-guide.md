# Guide des Skills Devin Local

## Structure

Un skill est un dossier contenant un fichier `SKILL.md` obligatoire et des fichiers de référence optionnels :

```
skill-name/
├── SKILL.md (obligatoire)
└── references/ (optionnel)
    ├── guide1.md
    └── guide2.md
```

## Syntaxe SKILL.md

Le fichier `SKILL.md` a un frontmatter YAML optionnel suivi du prompt. Tous les champs sont optionnels (le nom par défaut au nom du dossier) :

```yaml
---
name: skill-name
description: Brief explanation shown to the model to help it decide when to invoke the skill
argument-hint: "[file] [options]"
model: sonnet
subagent: true
allowed-tools:
  - read
  - grep
  - glob
  - exec
permissions:
  allow:
    - Read(src/**)
  deny:
    - exec
  ask:
    - Write(**)
triggers:
  - user
  - model
---

Your prompt content goes here...
```

### Référence des champs frontmatter

| Champ | Type | Défaut | Description |
|-------|------|--------|-------------|
| `name` | string | nom du dossier | Nom affiché du skill (utilisé pour `/skill-name`) |
| `description` | string | none | Affiché dans les complétions slash command et au modèle pour décider de l'invocation |
| `argument-hint` | string | none | Indice affiché après le nom de commande (ex: `[filename]`) |
| `model` | string | modèle courant | Override du modèle pour ce skill (ex: `opus`, `sonnet`, `swe`, `codex`) |
| `subagent` | boolean | `false` | Exécuter le skill comme subagent indépendant (propre context window) |
| `agent` | string | none | Exécuter comme subagent avec un profil custom spécifique |
| `allowed-tools` | list | tous les outils | Restreindre les outils disponibles pour le skill |
| `permissions` | object | hérite | Overrides de permissions (`allow`/`deny`/`ask`) — additifs au permissions de session |
| `triggers` | list | `[user, model]` | Comment le skill peut être invoqué |

### Triggers

| Trigger | Description | Défaut |
|----------|-------------|--------|
| `user` | L'utilisateur invoque via `/skill-name` | Activé |
| `model` | L'agent invoque autonomement quand pertinent | Activé |

Mettre `triggers: [user]` pour empêcher l'agent d'invoquer le skill de lui-même.

### Outils autorisés (`allowed-tools`)

Outils disponibles : `read`, `edit`, `grep`, `glob`, `exec`. Outils MCP supportés via `mcp__<server>__<tool>` (ex: `mcp__github__list_issues`).

> **Sécurité** : si `allowed-tools` n'est pas spécifié, le skill a accès à tous les outils. Pour les skills sensibles, toujours restreindre au minimum nécessaire.

### Permissions

```yaml
permissions:
  allow:
    - Read(src/**)
    - Exec(npm run test)
  deny:
    - Write(/etc/**)
    - exec
  ask:
    - Write(src/**)
```

- `allow` — auto-approuvé pendant l'exécution du skill
- `deny` — bloqué pendant l'exécution
- `ask` — toujours demande à l'utilisateur

> Les permissions de skill sont **additives** au permissions de session de base. Un skill ne peut pas accorder des permissions refusées à un niveau supérieur (projet ou organisation).

### Exécution en subagent

`subagent: true` fait tourner le skill comme un subagent indépendant avec son propre context window — utile pour les tâches focalisées qui ne doivent pas encombrer la conversation principale. Voir `/cli/subagents` pour les profils custom via `agent: <profile>`.

### Règles de formatage critiques

- Frontmatter avec **exactement 3 tirets** `---` (pas 4)
- Le frontmatter doit être le **tout premier contenu** du fichier — aucun titre ni commentaire avant
- La description ne doit pas contenir `: ` (colon+espace) — cela casse le parsing YAML
- Vérifier après création : `head -1 SKILL.md` doit afficher `---`
- Contenu < 500 lignes

## Skill Scopes

### Skills projet (committés dans git)
- `.devin/skills/` (emplacement recommandé)
- `.agents/skills/` (standard cross-agent)
- `.windsurf/skills/` (legacy, déprécié — même format que `.devin/skills/`)

### Skills globaux (utilisateur, non committés)
- Linux/macOS : `~/.config/devin/skills/` (emplacement canonique XDG, per ADR-0002)
- Windows : `%APPDATA%\devin\skills\` (typiquement `C:\Users\<user>\AppData\Roaming\devin\skills\`)
- `~/.agents/skills/` (standard cross-agent `.agents`)

> **Note** : Les skills tiers installables via des outils compatibles `.agents` fonctionnent avec Devin Local (support du standard `.agents`).

### Skills système (Enterprise)
- macOS : `/Library/Application Support/Windsurf/skills/`
- Linux : `/etc/windsurf/skills/`
- Windows : `C:\ProgramData\Windsurf\skills\`

## Invocation des skills

### Invocation automatique
Devin Local décide d'invoquer un skill en fonction de sa `description`. Une description claire et spécifique est essentielle.

### Invocation manuelle
L'utilisateur peut invoquer un skill avec `@skill-name`.

## Progressive Disclosure

Les skills doivent suivre le principe de "progressive disclosure" :

1. **Description claire** dans le frontmatter pour que Devin Local sache quand invoquer le skill
2. **Section "Quand utiliser ce skill"** explicite dans le contenu
3. **Procédure structurée** avec des étapes claires
4. **Références optionnelles** dans `references/` pour les détails complexes

## Exemple de structure

```markdown
---
name: deploy-to-production
description: Guides the deployment process to production with safety checks
---

## Pre-deployment Checklist
1. Run all tests
2. Check for uncommitted changes
3. Verify environment variables

## Deployment Steps
Follow these steps to deploy safely...

[Reference supporting files in this directory as needed]
```

## Best practices (documentation officielle)

1. **Descriptions claires** : La description aide Devin Local à décider quand invoquer le skill. Être spécifique sur ce que le skill fait et quand l'utiliser.
2. **Inclure des ressources pertinentes** : Templates, checklists, et exemples rendent les skills plus utiles.
3. **Noms descriptifs** : `deploy-to-staging` est meilleur que `deploy1`. Les noms doivent indiquer clairement ce que le skill fait.

## Skills vs Rules vs AGENTS.md

| Type | Format | Invocation | Usage |
|---|---|---|---|
| **Skill** | `SKILL.md` + références | `/skill-name` (slash command) ou automatique (trigger `model`) | Procédures complexes multi-étapes, tâches focalisées |
| **Rule** | `.md` avec frontmatter | `always_on`, `glob`, `model_decision`, `manual` | Contraintes comportementales courtes |
| **AGENTS.md** | markdown simple (pas de frontmatter) | Automatique par répertoire (racine = always_on, sous-rép = glob) | Contexte structurel d'un module |

> **Note** : les anciens « workflows » (Cascade) n'existent pas dans Devin Local. Les skills couvrent ce cas d'usage via l'invocation `/skill-name` (slash command, trigger `user`).

## Limites

- SKILL.md : < 500 lignes
- Éviter les skills trop génériques qui se chevauchent
- Mettre à jour `.devin/AGENTS.md` (inventory) lors de la création d'un nouveau skill
