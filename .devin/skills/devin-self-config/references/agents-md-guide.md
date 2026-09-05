# Guide AGENTS.md

> Source canonique : `https://docs.devin.ai/cli/extensibility/rules` (page « Rules & AGENTS.md » de la doc Devin CLI). Devin Local lit `AGENTS.md` via le système rules de Devin CLI.

## Qu'est-ce que AGENTS.md ?

`AGENTS.md` est un fichier markdown (sans frontmatter) qui fournit un contexte permanent à Devin Local pour un répertoire spécifique. Il est chargé automatiquement quand Devin Local travaille dans ce répertoire.

## Fichiers supportés

Devin Local lit les rules depuis tous ces fichiers (traités de manière identique) :

| Fichier | Notes |
|---|---|
| `AGENTS.md` | Recommandé |
| `AGENTS.local.md` | Rules personnelles (à gitignorer) — même comportement always-on |
| `AGENT.md` | Alternative singulier |
| `.windsurfrules` | Legacy Windsurf workspace rules |

> **`AGENTS.local.md`** : pour les instructions personnelles non partageables (préférences de travail, habitudes de test). À placer à côté de `AGENTS.md` et à ajouter au `.gitignore`. Suit la même convention que `.devin/config.local.json` — le suffixe `.local.` signale un override personnel.

## Discovery et Scoping

### Discovery
- **Workspace scanning** : Tous les fichiers `AGENTS.md` dans le workspace et ses sous-répertoires sont découverts
- **Git repository support** : Pour les dépôts git, recherche aussi dans les répertoires parents jusqu'au git root
- **Case insensitive** : `AGENTS.md` et `agents.md` sont tous deux reconnus
- **Sous-répertoires** : découverte paresseuse quand l'agent accède à des fichiers dans ce répertoire (garde le contexte focalisé)

### Scoping automatique
- **Racine du projet** : `AGENTS.md` à la racine = traité comme une rule `always_on` — le contenu complet est inclus dans le system prompt de Devin Local à chaque message
- **Sous-répertoires** : `AGENTS.md` dans un sous-répertoire = traité comme une rule `glob` avec pattern auto-généré `<directory>/**` — le contenu s'applique uniquement quand Devin Local lit ou modifie des fichiers dans ce répertoire
- Les AGENTS.md locaux **s'additionnent** au global (pas de remplacement)

### Emplacement global
Les `AGENTS.md` peuvent aussi être placés dans le répertoire de config global pour s'appliquer à tous les projets :
- Linux/macOS : `~/.config/devin/AGENTS.md`
- Windows : `%APPDATA%\devin\AGENTS.md`

> **Compatibilité Claude Code** : Devin Local lit aussi `~/.claude/CLAUDE.md` comme global rule.

### Plugins
Les [plugins](https://docs.devin.ai/cli/extensibility/plugins/overview) installés peuvent aussi fournir des rules : un `AGENTS.md` always-on à la racine du plugin plus des fichiers `rules/*.md` avec frontmatter `trigger`.

## Format

Markdown simple, **pas de frontmatter YAML**. Structure typique :

```markdown
# Contexte Permanent pour [répertoire]

> Ce fichier est chargé en permanence dans le contexte de Devin Local pour ce répertoire.

## Règles Absolues
1. Règle 1
2. Règle 2

## Documentation de Référence
| Document | Contenu | Quand le consulter |
|---|---|---|
| doc1.md | Description | Quand |

## Stack Technique
- Framework X
- Langage Y

## Processus Standard
1. Étape 1
2. Étape 2
```

## Best practices (documentation officielle)

- **Instructions focalisées** : Chaque AGENTS.md doit contenir des instructions pertinentes pour le purpose de son répertoire
- **Formatage clair** : Bullet points, headers, et code blocks rendent les instructions plus faciles à suivre pour Devin Local
- **Être spécifique** : Des exemples concrets et des conventions explicites fonctionnent mieux que des guidelines vagues
- **Éviter la redondance** : Ne pas répéter les instructions globales dans les fichiers de sous-répertoire — ils héritent des parents

## Comparaison avec Rules

| Aspect | AGENTS.md | Rule |
|---|---|---|
| **Scope** | Répertoire spécifique (auto-généré) | Global ou glob pattern (manuel) |
| **Format** | Markdown simple (pas de frontmatter) | Frontmatter YAML + contenu |
| **Activation** | Automatique par répertoire | always_on, model_decision, glob, manual |
| **Racine** | always_on (inclus dans chaque message) | N/A |
| **Sous-répertoire** | glob `<directory>/**` (auto) | N/A |
| **Usage typique** | Contexte structurel d'un module | Contraintes comportementales |
| **Taille** | Peut être plus long | < 12 000 caractères |

## Quand utiliser AGENTS.md vs Rule vs Skill

- **AGENTS.md** : Contexte permanent pour un répertoire (structure, conventions, stack technique)
- **Rule** : Contrainte comportementale courte qui s'applique dans un contexte donné
- **Skill** : Procédure complexe multi-étapes invoquée manuellement ou automatiquement
