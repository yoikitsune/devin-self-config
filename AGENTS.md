# AGENTS.md — devin-self-config

> Skill global d'auto-configuration de Devin Local. Permet à Devin Local d'analyser et d'améliorer son propre comportement en modifiant sa configuration `.devin/` (rules, skills, AGENTS.md) à partir du diagnostic de conversations passées (mode conversation) ou de vérifier la conformité des artifacts existants (mode alignment, ADR-0005).

## Quick Start

```bash
# Installer le skill globalement (Linux/macOS) — idempotent
./scripts/install-skills.sh

# Vérifier l'installation
./scripts/install-skills.sh --list

# Désinstaller
./scripts/install-skills.sh --remove
```

Sur Windows : `.\scripts\install-skills.ps1` (junctions, pas besoin d'admin).

## Tech Stack

- **Type** : Skill pur (pas de code, pas de CLI) — procédure markdown + références
- **Dépendance** : `dcr` (Devin Conversations Retriever) requis pour le mode conversation uniquement (Phase 1) — le mode alignment (ADR-0005) n'en a pas besoin
- **Distribution** : symlink global via `scripts/install-skills.{sh,ps1}` (per ADR-0001, adoptant ADR-0007 du projet dcr)
- **Chemin global** : `~/.config/devin/skills/` (XDG-convention, per ADR-0002)

## Project Structure

```
.
├── AGENTS.md              # Vous êtes ici — point d'entrée pour les agents
├── docs/
│   ├── index.md           # Routeur de documentation
│   └── decisions/         # Architecture Decision Records (ADR)
│       ├── 0001-adopt-adr-0007-symlink-distribution.md
│       ├── 0002-migrate-cascade-to-devin-local.md
│       ├── 0003-memory-in-devin-memory.md
│       ├── 0004-tool-names-devin-local.md
│       └── 0005-mode-alignment-sans-conversation.md
├── progress.md            # Tableau de bord vivant
├── TODO.md                # Tâches différées (hors scope courant)
├── scripts/
│   ├── install-skills.sh   # Installation globale (Linux/macOS, symlinks)
│   └── install-skills.ps1  # Installation globale (Windows, junctions)
└── .devin/
    └── skills/
        └── devin-self-config/
            ├── SKILL.md    # Procédure du skill (phases 0→5)
            └── references/ # Guides génériques + patterns dcr + templates
                ├── agents-md-guide.md
                ├── dcr-diagnostic-patterns.md
                ├── diagnostic-catalog-template.md
                ├── project-tooling-template.md
                ├── rules-guide.md
                └── skills-guide.md
```

## Conventions

- Le skill vit dans `.devin/skills/devin-self-config/` (source canonique versionnée)
- L'installation globale crée un symlink depuis `~/.config/devin/skills/devin-self-config` vers le repo
- Les `references/` génériques (guides rules/skills/agents-md) sont partagées entre tous les projets
- Les fichiers de mémoire projet-spécifiques (`diagnostic-catalog.md`, `project-tooling.md`) vivent dans `<projet>/.devin/memory/devin-self-config/` (per ADR-0003) — pas dans ce repo
- **Pas de global rule** — l'awareness est porté par la description tier-1 du skill (per ADR-0001)
- Les scripts d'installation nettoient automatiquement les anciens chemins legacy (`~/.codeium/windsurf/skills/`) per ADR-0002

## What NOT to Do

- Ne pas créer de `global_rules.md` — obsolète per ADR-0001 (l'awareness vient du skill)
- Ne pas copier le skill manuellement dans `~/.config/devin/skills/` — utiliser `scripts/install-skills.sh` (sinon drift)
- Ne pas committer de fichiers de mémoire projet-spécifiques dans ce repo — ils appartiennent aux projets utilisateurs
- Ne pas introduire de code applicatif — ce repo ne contient que des artifacts de configuration
- Ne pas installer dans l'ancien chemin `~/.codeium/windsurf/skills/` — obsolète per ADR-0002 (les scripts nettoient ce chemin automatiquement)

## Maintenance — Mise à jour des références de documentation

> Procédure à suivre quand l'utilisateur demande de « mettre à jour ce projet » ou de vérifier que les références de doc sont à jour. C'est une procédure de **recherche et rapport** : on ne modifie rien tant que l'utilisateur n'a pas validé.

### Source de vérité

- **Doc internet** (`https://docs.devin.ai`) = source de vérité canonique. C'est elle qu'on vérifie.
- **Doc interne** (bundled avec l'app, dans `…\devin\share\devin\docs`) = sous-ensemble **CLI uniquement**. Elle n'a **aucune section Desktop** (pas de page `devin-local`, pas de `cascade/*`). Utile pour le format des artifacts (skills, rules) mais **insuffisante pour Devin Local**. Ne pas s'y fier pour vérifier la validité des liens ou découvrir les nouveautés.
- **Index complet** : `https://docs.devin.ai/llms.txt` — liste toutes les pages disponibles (Cloud, CLI, Desktop, Enterprise, Federal). À fetcher à chaque mise à jour pour découvrir les nouvelles pages.

### Scope actuel

On se concentre sur **3 sujets** pour Devin Local : **skills**, **AGENTS.md**, **rules**. Les autres fonctionnalités (subagents, plugins, hooks, permissions, sandbox, etc.) sont notées mais hors scope pour l'instant.

### Procédure

1. **Inventorier les liens existants** : grep tous les `https?://` dans les `.md` du repo (`SKILL.md`, `AGENTS.md`, `docs/**`, `references/**`).
2. **Vérifier chaque lien en live** : `webfetch` sur chaque URL référencée. Noter : valide / redirect / 404 / rate-limit (retry via `web_search` si 429).
3. **Fetcher l'index de la doc** : `webfetch https://docs.devin.ai/llms.txt` pour découvrir toutes les pages disponibles et identifier les nouvelles pages canoniques.
4. **Lire les pages clés du scope** : fetcher en live les pages canoniques pour skills, AGENTS.md, rules (voir section « Pages canoniques » ci-dessous) et comparer avec ce que le projet référence actuellement.
5. **Comparer doc interne vs doc internet** : confirmer que la doc interne ne couvre pas le scope Devin Local (pas de section Desktop).
6. **Rapporter à l'utilisateur** (sans modifier) :
   - Tableau des liens : URL → statut (valide / cassé / obsolète)
   - Pages canoniques manquantes ou plus pertinentes que celles référencées
   - Nouveautés Devin Local/CLI pertinentes pour le scope
   - Recommandations concrètes (quels liens ajouter/remplacer, dans quels fichiers)
7. **Attendre validation** avant toute modification.

### Pages canoniques à vérifier (scope skills / AGENTS.md / rules)

| Sujet | URL canonique Devin Local/CLI |
|---|---|
| Devin Local (vue d'ensemble) | `https://docs.devin.ai/desktop/devin-local` |
| Skills — overview | `https://docs.devin.ai/cli/extensibility/skills/overview` |
| Skills — creating (format SKILL.md) | `https://docs.devin.ai/cli/extensibility/skills/creating-skills` |
| Rules & AGENTS.md (CLI, canonique pour Devin Local) | `https://docs.devin.ai/cli/extensibility/rules` |

> La page `/cli/extensibility/rules` est la référence canonique pour Devin Local : elle couvre Rules **et** AGENTS.md ensemble (`AGENTS.md`, `AGENTS.local.md`, `AGENT.md`, `.devin/rules/*.md`, `.devin/global_rules.md`, règles globales, imports depuis autres outils). Les pages `/desktop/cascade/*` (Cascade legacy, EOL) ne sont plus référencées — Cascade a disparu de Devin Desktop.

## Current State

Voir `progress.md` pour le statut live et `docs/index.md` pour le routeur de documentation complet.
