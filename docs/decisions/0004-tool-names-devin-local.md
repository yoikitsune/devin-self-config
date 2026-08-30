# ADR-0004: Adoption des noms d'outils Devin Local canoniques

> Status: Accepted
> Date: 2026-08-30

## Context

ADR-0002 a migré le projet de Cascade vers Devin Local (renommage du skill, chemins d'installation, documentation Phase 0). La décision originale (ligne 30) stipulait : *"Toutes les références 'Cascade' dans SKILL.md, references, et documentation → 'Devin Local' ou 'l'agent'"*.

Cependant, cette migration **n'a pas couvert les noms d'outils**. Le SKILL.md et les templates référençaient encore des noms d'outils **Cascade** qui n'existent pas dans Devin Local :

| Outil Cascade référencé | Outil réel Devin Local | Statut |
|---|---|---|
| `read_url_content` | `webfetch` | ❌ inexistant |
| `search_web` | `web_search` | ❌ inexistant |
| `trajectory_search` | **aucun équivalent** | ❌ Cascade-only |
| `run_command` | `exec` | ❌ inexistant |
| `read_file` | `read` | ❌ inexistant |
| `write_to_file` | `write` | ❌ inexistant |
| `grep_search` | `grep` | ❌ inexistant |
| `multi_edit` | `edit` | ❌ inexistant |

Deux problèmes concrets découlaient de ce gap :

1. **Instructions non exécutables** : quand le skill s'exécute dans Devin Local, il instruit l'agent d'utiliser des outils inexistants (ex: `read_url_content` en Phase 0). L'agent doit deviner le mapping lui-même, ce qui mine l'efficacité de l'auto-configuration.

2. **Template propagé incorrect** : `project-tooling-template.md` listait intégralement des outils Cascade. Ce template est copié dans chaque nouveau projet pour servir de référence d'outils — il propageait donc l'erreur à chaque nouvelle invocation.

## Decision

### 1. Adopter les noms d'outils Devin Local canoniques

Le SKILL.md, les références et les templates utilisent désormais les noms d'outils réels de Devin Local :

| Outil Devin Local | Rôle |
|---|---|
| `exec` | Exécuter une commande CLI (shell persistent) |
| `read` | Lire un fichier (texte ou image) |
| `write` | Créer/écraser un fichier |
| `edit` | Modifier un fichier existant (remplacement exact) |
| `grep` | Rechercher dans le code (ripgrep) |
| `find_file_by_name` | Recherche de fichiers par glob pattern |
| `code_search` | Recherche sémantique dans le code (subagent) |
| `web_search` | Recherche web |
| `webfetch` | Lire le contenu d'une URL |
| `browser_preview` | Lancer un aperçu navigateur d'un serveur web |

### 2. Documenter l'absence d'équivalent à `trajectory_search`

Devin Local **n'a pas d'outil natif de recherche dans l'historique des conversations**. `trajectory_search` était un outil Cascade, désormais EOL.

**Conséquence pour le skill** : le fallback de la Phase 1 (quand `dcr` n'est pas disponible) passe de "utiliser `trajectory_search`" à "demander à l'utilisateur de coller le contenu de la conversation à analyser". C'est un fallback manuel, moins puissant mais honnête — il ne prétend pas utiliser un outil qui n'existe pas.

### 3. Nettoyer les références `workflows` résiduelles

Les "workflows" sont un concept Cascade (EOL). Devin Local couvre ce cas d'usage via les skills (slash commands). La référence résiduelle "rules/skills/workflows" dans le SKILL.md (Phase 0b) est retirée au profit de "rules/skills".

## Consequences

- **Positives** :
  - Le skill instruit l'agent avec des noms d'outils qui existent réellement — fin des instructions non exécutables
  - Le `project-tooling-template.md` propagé dans chaque nouveau projet est désormais correct
  - Le fallback de Phase 1 est honnête (manuel) plutôt que trompeur (outil inexistant)
  - Cohérence avec la doc officielle Devin Local/CLI

- **Négatives** :
  - Le fallback manuel est moins pratique que `trajectory_search` l'était en Cascade — mais c'est la réalité de Devin Local, et `dcr` reste l'outil principal (le fallback n'est qu'un cas edge)

- **Compatibilité** : aucune — les noms d'outils Cascade n'étaient de toute façon pas fonctionnels en Devin Local. Ce commit corrige une incohérence, il ne casse rien.

## Evidence

- **Devin Local docs** : `https://docs.devin.ai/desktop/devin-local` — présente l'agent, ses modes, ses permissions, ses subagents. Aucun outil `trajectory_search` mentionné.
- **Skills — creating** : `https://docs.devin.ai/cli/extensibility/skills/creating-skills` — liste les outils disponibles pour `allowed-tools` : `read`, `edit`, `grep`, `glob`, `exec` (+ MCP tools). Aucun `run_command`, `read_file`, `grep_search`, etc.
- **Rules & AGENTS.md** : `https://docs.devin.ai/cli/extensibility/rules` — page canonique Devin Local/CLI couvrant Rules **et** AGENTS.md ensemble. Remplace les pages Cascade obsolètes (`/desktop/cascade/memories`, `/desktop/cascade/agents-md`).

## Relations

- **Amends** : ADR-0002 (la migration des noms d'outils était un gap non couvert par la décision originale)
- **Supersede** : les références aux noms d'outils Cascade dans le SKILL.md et les templates
- **Conserve** : ADR-0001 (distribution par symlinks), ADR-0002 (migration Cascade → Devin Local sur les autres aspects), ADR-0003 (mémoire projet dans `.devin/memory/`)
