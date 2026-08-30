# ADR-0002: Migration Cascade → Devin Local

> Status: Accepted
> Date: 2026-08-02

## Context

Le 2 juin 2026, Cognition a livré Devin Desktop comme mise à jour OTA de Windsurf. Le 1er juillet 2026, **Cascade** — l'agent local de Windsurf — a atteint son **end-of-life** et a été remplacé par **Devin Local** (réécrit from scratch en Rust, 30% plus token-efficient, support des subagents).

Avant cette migration, ce projet (`cascade-self-config`) était dans un état obsolète :

1. **Nom du projet** : `cascade-self-config` — nommé d'après un agent (Cascade) qui n'existe plus.
2. **Description du skill** : "Auto-configuration de **Cascade**" — l'agent cible n'existe plus.
3. **Chemin d'installation global** : `~/.codeium/windsurf/skills/` — chemin legacy de l'ère Windsurf/Cascade. La documentation officielle de Devin CLI (https://docs.devin.ai/cli/extensibility/skills) introduit de nouveaux chemins canoniques :
   - `~/.config/devin/skills/` (XDG-convention, Linux/macOS)
   - `%APPDATA%\devin\skills\` (Windows)
   - `~/.agents/skills/` (standard cross-agent `.agents`)
4. **Documentation du skill (Phase 0)** : référençait uniquement les docs Cascade (`/desktop/cascade/*`), sans mentionner Devin Local ni le format skills de Devin CLI.
5. **Memories** : la doc officielle précise que les Memories s'appliquent **uniquement à Cascade (legacy)** — Devin Local ne persiste pas de memories. Le skill utilisait déjà Rules/Skills/AGENTS.md (pas de Memories), mais la doc Phase 0 ne mentionnait pas cette distinction.

## Decision

**Migrer entièrement le projet de Cascade vers Devin Local.** Trois changements concrets :

### 1. Renommage : `cascade-self-config` → `devin-self-config`

- Skill directory : `.devin/skills/cascade-self-config/` → `.devin/skills/devin-self-config/`
- Skill name (frontmatter) : `cascade-self-config` → `devin-self-config`
- Description : "Auto-configuration de Cascade" → "Auto-configuration de Devin Local"
- Toutes les références "Cascade" dans SKILL.md, references, et documentation → "Devin Local" ou "l'agent"
- Invocation : `@cascade-self-config` → `@devin-self-config`
- Commit message convention : `chore(cascade-config)` → `chore(devin-config)`

> **Note** : Le repo GitHub (`yoikitsune/cascade-self-config`) et le répertoire local (`/home/julien/Sources/cascade-self-config`) gardent leur nom historique pour l'instant. Le renommage du repo GitHub est noté dans `TODO.md` comme tâche différée.
>
> **Amendement (2026-08-03)** : Le repo GitHub a été renommé en `yoikitsune/devin-self-config` et le dossier local en `/home/julien/Sources/devin-self-config`. Voir `TODO.md`.
>
> **Amendement (2026-08-30)** : La migration des **noms d'outils** Cascade (`trajectory_search`, `run_command`, `read_file`, `grep_search`, `multi_edit`, `read_url_content`, `search_web`) vers les noms d'outils Devin Local canoniques (`webfetch`, `web_search`, `exec`, `read`, `write`, `edit`, `grep`, etc.) n'avait pas été couverte par cette décision. Corrigé par **ADR-0004**. Par ailleurs, les URLs Cascade obsolètes référencées ci-dessous (`/desktop/cascade/memories`, `/desktop/cascade/agents-md`) sont remplacées par la page canonique CLI `/cli/extensibility/rules` qui couvre Rules **et** AGENTS.md ensemble.

### 2. Migration du chemin d'installation global

| Plateforme | Avant (legacy) | Après (ADR-0002) |
|---|---|---|
| Linux/macOS | `~/.codeium/windsurf/skills/` | `~/.config/devin/skills/` |
| Windows | `%USERPROFILE%\.codeium\windsurf\skills\` | `%APPDATA%\devin\skills\` |

**Cleanup automatique** : les scripts `install-skills.{sh,ps1}` vérifient et nettoient automatiquement les anciens chemins legacy lors de l'installation (`cleanup_legacy_skill` / `Cleanup-Legacy-Skill`). Ils vérifient à la fois le nouveau nom (`devin-self-config`) et l'ancien nom (`cascade-self-config`) dans l'ancien chemin.

### 3. Mise à jour de la documentation Phase 0

La Phase 0 du SKILL.md lit maintenant **4 pages** au lieu de 3 :

1. **Devin Local** : `https://docs.devin.ai/desktop/devin-local` (NOUVEAU — modes, permissions, subagents, différences avec Cascade)
2. **Skills — overview** : `https://docs.devin.ai/cli/extensibility/skills/overview` (CHANGÉ — était `/desktop/cascade/skills` ; Devin Local utilise le format skills de Devin CLI)
3. **Skills — creating** : `https://docs.devin.ai/cli/extensibility/skills/creating-skills` (NOUVEAU — référence du format SKILL.md)
4. **Rules & AGENTS.md** : `https://docs.devin.ai/cli/extensibility/rules` (CHANGÉ — était `/desktop/cascade/memories` + `/desktop/cascade/agents-md` ; la page CLI canonique couvre Rules **et** AGENTS.md ensemble, avec note explicite que Memories sont Cascade-only)

### Ce qui change par rapport à l'état précédent

| Aspect | Avant (ADR-0001) | Après (ADR-0002) |
|---|---|---|
| Nom du skill | `cascade-self-config` | `devin-self-config` |
| Agent cible | Cascade (EOL 2026-07-01) | Devin Local |
| Chemin global Linux | `~/.codeium/windsurf/skills/` | `~/.config/devin/skills/` |
| Chemin global Windows | `%USERPROFILE%\.codeium\windsurf\skills\` | `%APPDATA%\devin\skills\` |
| Cleanup legacy | Non | Oui (automatique dans les scripts) |
| Doc Phase 0 | 3 URLs (Cascade) | 4 URLs (Devin Local + CLI skills) |
| Memories | Non utilisées (silencieux) | Non utilisées (explicite : Cascade-only) |

## Consequences

- **Positives** :
  - Le projet référence l'agent actuel (Devin Local), pas un agent mort (Cascade)
  - Chemin d'installation canonique XDG (`~/.config/devin/skills/`) — aligné avec la doc officielle
  - Cleanup automatique des anciennes installations legacy — pas de drift entre anciens et nouveaux chemins
  - Documentation Phase 0 inclut Devin Local et le format skills de Devin CLI
  - Distinction explicite Memories (Cascade-only) vs Rules (cross-agent)

- **Nécessite** :
  - Test empirique Windows du nouveau chemin `%APPDATA%\devin\skills\` (voir `TODO.md`)
  - Renommage du repo GitHub `cascade-self-config` → `devin-self-config` (voir `TODO.md`)
  - Les projets utilisateurs qui ont des références projet-spécifiques dans `<projet>/.devin/skills/cascade-self-config/references/` doivent les renommer en `<projet>/.devin/skills/devin-self-config/references/`

## Evidence

- **Devin Desktop launch** : https://devin.ai/blog/windsurf-is-now-devin-desktop (2 juin 2026)
- **Devin Desktop FAQ** : https://docs.devin.ai/desktop/devin-desktop-faq — "Cascade remains available through July 1st"
- **Devin Local docs** : https://docs.devin.ai/desktop/devin-local — "Devin Local is our next-generation agent harness shared with Devin CLI"
- **Devin CLI Skills docs** : https://docs.devin.ai/cli/extensibility/skills/overview — nouveaux chemins canoniques (`~/.config/devin/skills/`, `~/.agents/skills/`)
- **Rules & AGENTS.md docs** : https://docs.devin.ai/cli/extensibility/rules — page canonique CLI couvrant Rules **et** AGENTS.md ensemble (remplace les pages Cascade obsolètes `/desktop/cascade/memories` et `/desktop/cascade/agents-md`). Les Memories sont Cascade-only et ne s'appliquent pas à Devin Local.

## Relations

- **Supersede** : le nom `cascade-self-config` et le chemin `~/.codeium/windsurf/skills/` (issus d'ADR-0001)
- **Conserve** : ADR-0001 (principe de distribution par symlinks — seul le chemin change, pas le principe)
- **Adopte** : les chemins canoniques de la doc Devin CLI (XDG-convention)
- **Amended by** : ADR-0004 (migration des noms d'outils Cascade → Devin Local canoniques, non couverte par cette décision originale)
