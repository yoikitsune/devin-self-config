# Progress — devin-self-config

> Dernière mise à jour : 2026-09-05 (ADR-0005 — mode alignment + auto-review Phase 4b)

## Current Phase: Stabilisation post-migration

Le projet a migré de Cascade vers Devin Local (ADR-0002), a déplacé la mémoire projet vers `.devin/memory/` (ADR-0003), a migré les noms d'outils Cascade vers les noms d'outils Devin Local canoniques (ADR-0004), puis a ajouté le mode alignment et l'auto-review Phase 4b (ADR-0005). Toutes les phases actives sont terminées.

## Ce qui est fait

- [x] **ADR-0005 — Mode alignment + auto-review Phase 4b** (2026-09-04) : ajout d'un mode alignment au skill (diagnostic de conformité des artifacts `.devin/` sans conversation source). Auto-review Phase 4b : le skill applique ses propres critères de diagnostic aux artifacts qu'il vient de créer. Anti-redundance : SKILL.md pointe vers les guides `references/` au lieu de répéter les règles de formatage inline. Nouvelles catégories de diagnostic : `format`, `size`, `best-practice`, `deprecated`, `unused-capability`. Template `diagnostic-catalog-template.md` mis à jour avec les catégories par mode. Né d'un besoin concret sur OUTILS_16 (vérifier la conformité d'artifacts sans conversation).
- [x] **Corriger les gaps des guides vs doc officielle** (2026-09-05) : trigger `agent` ajouté à `rules-guide.md`, recommandation "Skills > Rules" ajoutée, section Plugins ajoutée aux 3 guides, clarification `glob` vs `find_file_by_name` dans `skills-guide.md`, chemin `~/.codeium/<channel>/skills/` ajouté.
- [x] **ADR-0004 — Noms d'outils Devin Local canoniques** (2026-08-30) : migration des noms d'outils Cascade (`trajectory_search`, `run_command`, `read_file`, `grep_search`, `multi_edit`, `read_url_content`, `search_web`) vers les noms d'outils Devin Local canoniques (`webfetch`, `web_search`, `exec`, `read`, `write`, `edit`, `grep`, `find_file_by_name`, `code_search`, `browser_preview`) dans le SKILL.md, `project-tooling-template.md` et `dcr-diagnostic-patterns.md`. Documente l'absence d'équivalent à `trajectory_search` en Devin Local (fallback manuel). ADR-0002 amendé sur ce gap. Référence résiduelle "workflows" retirée du SKILL.md (concept Cascade EOL).
- [x] **Mise à jour des références de documentation** (2026-08-30) : section "Maintenance — Mise à jour des références de documentation" ajoutée à `AGENTS.md` (procédure de vérification des liens de doc). Guides `skills-guide.md`, `rules-guide.md`, `agents-md-guide.md` refondus pour aligner avec la doc officielle actuelle (frontmatter `model`/`subagent`/`agent`/`permissions`/`triggers`). URLs canoniques Devin Local/CLI adoptées (page `/cli/extensibility/rules` remplace les pages Cascade obsolètes).
- [x] **ADR-0003 — Mémoire projet dans `.devin/memory/`** (2026-08-03) : SKILL.md mis à jour (Phase 0b avec migration automatique, tous les chemins mis à jour), ADR créé. La migration des fichiers existants dans les projets utilisateurs se fera automatiquement à la prochaine invocation du skill (Phase 0b).
- [x] **ADR-0002 — Migration Cascade → Devin Local** (2026-08-02) : renommage du skill `cascade-self-config` → `devin-self-config`, migration du chemin global `~/.codeium/windsurf/skills/` → `~/.config/devin/skills/` (XDG), scripts mis à jour avec cleanup automatique des anciens chemins, SKILL.md réécrit (Cascade → Devin Local, 4 URLs de doc au lieu de 3, doc Devin Local ajoutée), toutes les références et la doc mises à jour. **Amendé 2026-08-30** : la migration des noms d'outils (non couverte à l'origine) est désormais traitée par ADR-0004 ; les URLs Cascade obsolètes (`/desktop/cascade/memories`, `/desktop/cascade/agents-md`) sont remplacées par la page canonique `/cli/extensibility/rules`.
- [x] **Phase 2 — Documentation projet** : `AGENTS.md`, `docs/index.md`, `docs/decisions/0001-adopt-adr-0007-symlink-distribution.md`, `progress.md` créés, `TODO.md` nettoyé
- [x] **Phase 1 — Restructuration selon ADR-0007** : skill déplacé dans `.devin/skills/devin-self-config/`, `scripts/install-skills.{sh,ps1}` créés, `global_rules.md` supprimé, installation globale migrée (copie stale → symlink validé)
- [x] **Phase 3 — Mise à jour du SKILL.md** : section "Architecture hybride" réécrite pour le modèle symlink (ADR-0001), prérequis `dcr` explicite avec procédure d'installation, référence à `cascade-self-automation` supprimée (skill inexistant), `references/dcr-diagnostic-patterns.md` mis à jour (`dcr` sur PATH via wrapper, `dcr sync` manuel retiré — auto-sync)
- [x] **Phase 4 — Test et validation** : `--list` validé, résolution symlink validée, mise à jour live validée (marker temporaire propagé instantanément), cycle remove/install validé
- [x] Intégration `dcr` dans le skill (commit `232cdb6`) — Phase 1 du SKILL.md utilise `dcr` pour l'analyse de conversations
- [x] Support Windows dans le SKILL.md (commit `6084446`) — chemins et commandes PowerShell

## Ce qui est en cours

Rien — toutes les phases sont terminées.

## Ce qui est prévu

Rien — toutes les phases sont terminées.

## Ce qui est bloqué

Rien actuellement bloqué.

## Architecture

- **Type** : Skill pur (markdown + références, pas de code)
- **Distribution** : symlink global via `scripts/install-skills.{sh,ps1}` (per ADR-0001)
- **Chemin global** : `~/.config/devin/skills/` (XDG-convention, per ADR-0002) — les scripts nettoient automatiquement l'ancien chemin `~/.codeium/windsurf/skills/`
- **Dépendance** : `dcr` (Devin Conversations Retriever) requis pour le mode conversation uniquement (Phase 1) — le mode alignment (ADR-0005) n'en a pas besoin. Installation via `devin-conversations-retriever/scripts/install-skills.sh`
- **Awareness** : porté par la description tier-1 du skill (~100 tokens), pas par une global rule
- **Agent cible** : Devin Local (successeur de Cascade depuis le 1er juillet 2026)

## ADRs

| ADR | Titre | Statut |
|---|---|---|
| [ADR-0001](docs/decisions/0001-adopt-adr-0007-symlink-distribution.md) | Adoption d'ADR-0007 (distribution par symlinks) | Accepted |
| [ADR-0002](docs/decisions/0002-migrate-cascade-to-devin-local.md) | Migration Cascade → Devin Local (renommage + chemins) | Accepted (amended 2026-08-30) |
| [ADR-0003](docs/decisions/0003-memory-in-devin-memory.md) | Mémoire projet dans `.devin/memory/` au lieu de `.devin/skills/` | Accepted |
| [ADR-0004](docs/decisions/0004-tool-names-devin-local.md) | Noms d'outils Devin Local canoniques + absence d'équivalent `trajectory_search` | Accepted |
| [ADR-0005](docs/decisions/0005-mode-alignment-sans-conversation.md) | Mode alignment (diagnostic de conformité sans conversation) + auto-review Phase 4b | Accepted |
