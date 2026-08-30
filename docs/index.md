# Documentation Index — devin-self-config

> Routeur source-of-truth. Dernière mise à jour : voir `progress.md`.

## Documentation principale

| Document | Rôle | Quand le lire |
|---|---|---|
| [`/AGENTS.md`](../AGENTS.md) | Point d'entrée pour les agents dans ce repo | Toujours — en premier |
| [`/progress.md`](../progress.md) | Tableau de bord vivant (phase, statut, blocages) | Pour savoir où en est le projet |
| [`/TODO.md`](../TODO.md) | Tâches différées hors scope courant | Pour ne pas oublier les sujets ouverts |
| [`/.devin/skills/devin-self-config/SKILL.md`](../.devin/skills/devin-self-config/SKILL.md) | Procédure du skill (phases 0→5) | Pour comprendre comment le skill fonctionne |
| [`/.devin/skills/devin-self-config/references/`](../.devin/skills/devin-self-config/references/) | Guides génériques + patterns dcr + templates | Quand on modifie le skill ou ses références |

## Architecture Decision Records

| ADR | Titre | Statut |
|---|---|---|
| [ADR-0001](decisions/0001-adopt-adr-0007-symlink-distribution.md) | Adoption d'ADR-0007 (distribution par symlinks) | Accepted |
| [ADR-0002](decisions/0002-migrate-cascade-to-devin-local.md) | Migration Cascade → Devin Local (renommage + chemins) | Accepted (amended 2026-08-30) |
| [ADR-0003](decisions/0003-memory-in-devin-memory.md) | Mémoire projet dans `.devin/memory/` au lieu de `.devin/skills/` | Accepted |
| [ADR-0004](decisions/0004-tool-names-devin-local.md) | Noms d'outils Devin Local canoniques + absence d'équivalent `trajectory_search` | Accepted |

## Scripts

| Script | Rôle |
|---|---|
| [`/scripts/install-skills.sh`](../scripts/install-skills.sh) | Installation globale du skill (Linux/macOS, symlinks vers `~/.config/devin/skills/`) |
| [`/scripts/install-skills.ps1`](../scripts/install-skills.ps1) | Installation globale du skill (Windows, junctions vers `%APPDATA%\devin\skills\`) |

## Référence externe

- **ADR-0007 du projet `devin-conversations-retriever`** : décision originale sur la distribution des skills par symlinks et la suppression des global rules. Ce repo adopte cette décision via ADR-0001 local.
  - Chemin : `/home/julien/Sources/devin-conversations-retriever/docs/decisions/0007-skill-distribution-symlinks.md`
- **Devin Local documentation** : `https://docs.devin.ai/desktop/devin-local` — l'agent local de Devin Desktop (depuis le 1er juillet 2026)
