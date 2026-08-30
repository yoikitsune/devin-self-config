---
name: devin-self-config
description: Auto-configuration de Devin Local pour améliorer son comportement. Analyser et corriger des erreurs ou optimiser des méthodes, diagnostiquer les causes, et créer/modifier les artifacts .devin (rules, skills, AGENTS.md) pour éviter la récurrence ou adopter de meilleures pratiques. Utiliser quand l'utilisateur dit "auto-configure-toi", "améliore ta config", "tu as fait une erreur", "tu pourrais faire mieux", ou après une session où Devin Local a été imprécis ou peu efficace.
---

# Devin Self-Config

## Objectif

Permettre à Devin Local d'**analyser et d'améliorer son propre comportement** en modifiant sa configuration `.devin/` — que ce soit pour corriger des erreurs passées ou pour adopter de meilleures méthodes de travail.

## Architecture hybride de ce skill

Ce skill est distribué globalement via **symlink** (per ADR-0001, adoptant ADR-0007 du projet `devin-conversations-retriever`). La source canonique est dans ce repo à `.devin/skills/devin-self-config/`, et l'installation globale crée un lien symbolique :

- **Linux/macOS** : `~/.config/devin/skills/devin-self-config` → `<repo>/.devin/skills/devin-self-config/` (via `scripts/install-skills.sh`)
- **Windows** : `%APPDATA%\devin\skills\devin-self-config` → `<repo>\.devin\skills\devin-self-config\` (junction via `scripts/install-skills.ps1`)

**Avantages du modèle symlink** :
- **Updates live** — éditer `SKILL.md` dans le repo se propage immédiatement à toutes les sessions. `git pull` est le mécanisme de mise à jour.
- **Pas de drift** — une seule source de vérité (le repo), pas de copie à synchroniser manuellement.
- **Désinstallation propre** — `./scripts/install-skills.sh --remove` supprime uniquement le symlink, le repo est intact.

Le skill contient :
- Le `SKILL.md` (procédure, identique pour tous les projets)
- Des **références génériques** (guides rules/skills/agents-md, patterns dcr) dans `references/`

À chaque invocation dans un projet, il utilise/crée des **fichiers de mémoire projet-spécifiques** dans `<projet>/.devin/memory/devin-self-config/` (per ADR-0003) :
- `diagnostic-catalog.md` — catalogue des erreurs diagnostiquées pour ce projet
- `project-tooling.md` — outils CLI, MCP, navigateur spécifiques à ce projet

Ces fichiers projet permettent d'accumuler de la mémoire par projet sans polluer les autres. Ils vivent dans `.devin/memory/` (état accumulé) et non dans `.devin/skills/` (réservé aux définitions de skills).

## Prérequis

- **`dcr` (Devin Conversations Retriever)** doit être installé globalement. Ce skill dépend fortement de l'analyse des conversations pour fonctionner correctement — la Phase 1 entière est construite autour de `dcr`.
  - Installation : `devin-conversations-retriever/scripts/install-skills.sh` (crée le wrapper `~/.local/bin/dcr` + le skill `dcr-conversation` global)
  - Vérifier : `dcr status` (depuis n'importe quel répertoire)
  - Si `dcr` n'est pas disponible, le skill tombe sur le fallback manuel : demander à l'utilisateur de coller le contenu de la conversation à analyser (Devin Local n'a pas d'outil natif de recherche dans l'historique des conversations — `trajectory_search` était un outil Cascade, désormais EOL)

## Quand utiliser ce skill

- L'utilisateur signale une **erreur** de fonctionnement de Devin Local
- L'utilisateur suggère une **amélioration** de méthode ("tu pourrais faire X plus efficacement", "utilise plutôt tel outil dans ce cas")
- L'utilisateur dit "auto-configure-toi", "améliore ta config", "tu as fait une erreur", "tu pourrais faire mieux"
- Après une session de débogage où Devin Local a gaspillé des étapes (mauvaise syntaxe CLI, mauvais outil, manque de prérequis)
- L'utilisateur fournit une conversation passée à analyser
- L'utilisateur veut que Devin Local adopte une nouvelle pratique de travail

## Ce que ce skill N'EST PAS

- ❌ Ce skill **n'est pas** un outil de résumé de conversation
- ❌ Ce skill **n'est pas** un outil de recommandations projet (logging, code, architecture)
- ❌ Ce skill **ne demande pas** à l'utilisateur quoi faire — il diagnostique, propose, puis agit après validation
- ✅ Ce skill analyse **le comportement de Devin Local lui-même** (erreurs ou améliorations) et crée des corrections dans `.devin/`

> Si tu te surprends à résumer la conversation ou à donner des recommandations sur le projet, **tu es hors-sujet**. Reviens au diagnostic du comportement de Devin Local.

## Procédure d'invocation attendue

L'utilisateur ouvre une **nouvelle conversation dédiée** (pas dans la conversation à analyser) et fournit :

1. `@devin-self-config` — invocation du skill
2. `@[conversation:...]` — référence de la conversation à analyser
3. Optionnellement : une description explicite du problème observé

Deux modes d'analyse :
- **Mode explicite** : l'utilisateur a décrit le problème → se concentrer sur ce point précis
- **Mode libre** : l'utilisateur n'a rien décrit → parcours systématique de la conversation pour identifier les erreurs de Devin Local

Le déroulement attendu est :

```
Phase 0  → lire la doc officielle (PREMIÈRE ACTION, avant toute analyse)
Phase 0b → vérifier/migrer la mémoire projet-spécifique
Phase 1  → dcr (conversation retriever) sur la conversation + diagnostic des erreurs
Phase 2  → choix d'artifact pour chaque correction
Phase 3  → création/modification des artifacts
Phase 4a → rapport de diagnostic à l'utilisateur (avant action)
    → l'utilisateur valide ou rediscute
Phase 4b → après validation : rapport de ce qui a été créé/modifié
Phase 5  → commit après validation explicite de l'utilisateur
```

## Phase 0 — Chargement de la documentation officielle (OBLIGATOIRE — PREMIÈRE ACTION)

> ⚠️ **STOP** : Cette phase doit être exécutée **avant toute autre action**, y compris avant toute recherche avec `dcr` sur la conversation à analyser. Ne pas passer à la Phase 0b tant que les URLs n'ont pas été lues.

**À chaque invocation de ce skill**, lire les 4 pages de documentation officielle pour avoir une vue d'ensemble à jour du fonctionnement des outils de configuration de Devin Local :

1. **Devin Local** : `https://docs.devin.ai/desktop/devin-local` — présentation de l'agent Devin Local, ses modes (Normal/Plan/Ask), ses permissions (Deny/Ask/Allow), ses subagents, le sandboxing
2. **Skills — overview** : `https://docs.devin.ai/cli/extensibility/skills/overview` — format, découverte, scopes (projet/global), triggers (`user`/`model`)
3. **Skills — creating** : `https://docs.devin.ai/cli/extensibility/skills/creating-skills` — référence complète du format `SKILL.md` : frontmatter (`name`, `description`, `allowed-tools`, `triggers`, `model`, `subagent`, `permissions`), contenu du prompt, exemples
4. **Rules & AGENTS.md** : `https://docs.devin.ai/cli/extensibility/rules` — page CLI canonique couvrant Rules **et** AGENTS.md ensemble : `AGENTS.md`, `AGENTS.local.md`, `AGENT.md`, `.devin/rules/*.md`, `.devin/global_rules.md`, règles globales, imports depuis autres outils (Cursor/Windsurf/Claude), `read_config_from`

> **Note sur les Memories** : Devin Local ne persiste pas de memories (c'était un mécanisme Cascade, désormais EOL). Pour le savoir durable, on utilise Rules, Skills ou AGENTS.md — jamais de memories.

Utiliser `webfetch` pour chaque lien. Si une page n'est pas accessible ou si des informations semblent manquantes, faire une `web_search` pour parfaire les connaissances (ex: "Devin Local skills format 2026", "Devin CLI skills progressive disclosure", "Devin AGENTS.md scoping").

> **Pourquoi cette étape** : La documentation officielle peut évoluer. Les références locales dans `references/` sont un résumé, mais la source de vérité est la doc en ligne. Cette lecture garantit que les artifacts créés respectent les conventions actuelles.

## Phase 0b — Vérification de la mémoire projet-spécifique

Après la Phase 0, vérifier et migrer si nécessaire les fichiers de mémoire projet :

### Étape 1 — Migration des anciens emplacements (per ADR-0003)

Avant toute chose, détecter et migrer les fichiers stockés à d'anciens emplacements :

1. **Chercher les anciens chemins** (dans l'ordre) :
   - `<projet>/.devin/skills/cascade-self-config/references/diagnostic-catalog.md`
   - `<projet>/.devin/skills/cascade-self-config/references/project-tooling.md`
   - `<projet>/.devin/skills/devin-self-config/references/diagnostic-catalog.md`
   - `<projet>/.devin/skills/devin-self-config/references/project-tooling.md`

2. **Si des fichiers sont trouvés à un ancien chemin** :
   - Créer `<projet>/.devin/memory/devin-self-config/` si nécessaire
   - Pour chaque fichier trouvé à un ancien chemin :
     - Si le fichier n'existe pas encore au nouvel emplacement → le déplacer
     - Si le fichier existe déjà au nouvel emplacement → comparer les dates de modification. Garder le plus récent, supprimer l'ancien. Si l'ancien est plus récent (cas rare), avertir l'utilisateur avant d'écraser
   - Supprimer les dossiers `references/` vides après migration
   - Supprimer les dossiers `cascade-self-config/` ou `devin-self-config/` vides dans `.devin/skills/` après migration
   - Signaler la migration à l'utilisateur dans le rapport de diagnostic (Phase 4a)

### Étape 1b — Suppression des copies locales obsolètes du skill

Après la migration des fichiers de mémoire, vérifier si une copie locale obsolète du skill existe dans le projet :

1. **Chercher** `<projet>/.devin/skills/cascade-self-config/SKILL.md`
   - Si trouvé → c'est une copie locale obsolète (le skill est maintenant distribué globalement via symlink, per ADR-0001). La supprimer après s'être assuré que les fichiers de mémoire ont été migrés (Étape 1)
   - Supprimer tout le dossier `<projet>/.devin/skills/cascade-self-config/` s'il est vide après migration

2. **Chercher** `<projet>/.devin/skills/devin-self-config/SKILL.md`
   - Si trouvé ET que ce n'est pas un symlink → c'est aussi une copie locale obsolète. Procéder comme ci-dessus
   - Si c'est un symlink → c'est l'installation globale, ne pas toucher

> **Attention** : Ne jamais supprimer un dossier sans avoir d'abord vérifié que les fichiers de mémoire (`diagnostic-catalog.md`, `project-tooling.md`) ont été migrés. Si des fichiers non reconnus sont présents, les signaler à l'utilisateur plutôt que de les supprimer.

### Étape 2 — Vérification/création des fichiers de mémoire

1. **Chercher** `<projet>/.devin/memory/devin-self-config/diagnostic-catalog.md`
   - Si absent → le créer depuis le template global `references/diagnostic-catalog-template.md`
   - Si présent → le lire pour connaître les erreurs déjà diagnostiquées dans ce projet

2. **Chercher** `<projet>/.devin/memory/devin-self-config/project-tooling.md`
   - Si absent → le créer depuis le template global `references/project-tooling-template.md`
   - Si présent → le lire pour connaître les outils et limites de ce projet

3. **Chercher** `<projet>/.devin/AGENTS.md`
   - Si présent → le lire pour connaître l'inventaire des rules/skills existants

> **Note** : Les références génériques (`rules-guide.md`, `skills-guide.md`, `agents-md-guide.md`) sont dans le répertoire global du skill et toujours disponibles. Seuls les fichiers de mémoire projet-spécifiques nécessitent une vérification.

## Procédure

### Phase 1 — Diagnostic

1. **Analyser la conversation source** : Utiliser `dcr` (Devin Conversations Retriever) pour récupérer et analyser la conversation. Consulter `references/dcr-diagnostic-patterns.md` pour les procédures détaillées. Adapter selon le mode :
   - **Mode explicite** : l'utilisateur a décrit le problème ou l'amélioration souhaitée → `dcr search "<mot-clé>"` pour recherche ciblée, puis `dcr show <id>` pour la conversation complète
   - **Mode libre** : l'utilisateur n'a rien décrit → `dcr list -l 10` pour identifier la conversation (auto-sync avant la commande), puis `dcr show <id>` ou `dcr export <id>` pour parcours systématique et identification de toutes les anomalies (commandes échouées, mauvais outils utilisés, prérequis manquants, étapes gaspillées, méthodes sous-optimales)
   - **Fallback** : si `dcr` n'est pas disponible ou la conversation n'est pas dans la DB, demander à l'utilisateur de coller le contenu de la conversation à analyser (pas d'outil natif de recherche d'historique dans Devin Local)

2. **Consulter le catalogue projet** : Lire `diagnostic-catalog.md` (projet) pour vérifier si des erreurs similaires ont déjà été diagnostiquées et corrigées. Éviter de recréer une correction existante.

3. **Identifier le type de diagnostic** :
   - **Diagnostic d'erreur** : quelque chose n'a pas fonctionné (commande échouée, mauvais outil, prérequis manquant, bug)
   - **Diagnostic d'amélioration** : quelque chose a fonctionné mais pourrait être fait mieux (trop d'étapes, outil suboptimal, méthode alternative plus efficace, pratique à adopter)

4. **Analyser le comportement** : Identifier précisément ce qui n'a pas fonctionné ou pourrait être amélioré dans le comportement de Devin Local :
   - Mauvaise syntaxe de commande CLI ?
   - Mauvais outil utilisé (MCP limité, script local au lieu de navigateur) ?
   - Manque de prérequis (credentials, index, configuration) ?
   - Mauvaise compréhension d'une API ou d'un service ?
   - Code de programmation erroné (null-check manquant, mauvais type) ?
   - Trop d'étapes pour une tâche simple ?
   - Outil suboptimal alors qu'une meilleure option existe ?
   - Pratique non documentée qui devrait devenir systématique ?

5. **Catégoriser** :
   - `cli-syntax` : Erreur de syntaxe dans une commande CLI
   - `tool-selection` : Mauvais choix d'outil pour la tâche
   - `missing-prerequisite` : Prérequis d'environnement manquant
   - `api-misuse` : Mauvaise utilisation d'une API
   - `code-error` : Bug dans le code produit
   - `process-gap` : Mauvaise anticipation d'un flux ou d'une intégration
   - `efficiency` : Méthode fonctionnelle mais sous-optimale (trop d'étapes, outil suboptimal)
   - `practice-adoption` : Nouvelle pratique à systématiser
   - `security` : Problème de sécurité (secret committé, commande insecure, permission excessive)

6. **Déterminer la correction** : Quelle connaissance aurait évité cette erreur ou rendu cette amélioration automatique ?

### Phase 2 — Choix de l'artifact

Arbre de décision :

```
La correction est-elle une contrainte comportementale courte ?
├─ OUI → Rule
│  ├─ S'applique toujours ? → trigger: always_on
│  ├─ S'applique selon contexte ? → trigger: model_decision
│  ├─ S'applique à des fichiers spécifiques ? → trigger: glob (avec globs: pattern)
│  └─ Activation manuelle uniquement ? → trigger: manual
│
├─ La correction nécessite-t-elle une procédure multi-étapes ?
│  └─ OUI → Skill (avec fichiers supports si besoin)
│     Note : un skill est invoquable via /skill-name (slash command),
│     ce qui couvre le cas d'usage des anciens « workflows ».
│
├─ La correction est-elle spécifique à un répertoire ?
│  └─ OUI → AGENTS.md dans ce répertoire
│
└─ La correction est-elle un fait ponctuel ?
   └─ Memory (via create_memory tool)
```

**Règle d'or** : Préférer `model_decision` à `always_on` pour économiser le contexte permanent. Une rule `always_on` ne se justifie que si l'erreur peut se reproduire à tout moment sans signal contextuel.

### Phase 3 — Création de l'artifact

1. **Consulter les références génériques** (dans le répertoire global du skill) :
   - `references/rules-guide.md` — syntaxe et modes d'activation des rules
   - `references/skills-guide.md` — structure, progressive disclosure, best practices
   - `references/agents-md-guide.md` — scoping et format AGENTS.md

2. **Consulter la mémoire projet** (dans `<projet>/.devin/memory/devin-self-config/`) :
   - `project-tooling.md` — outils disponibles et leurs limites pour ce projet
   - `diagnostic-catalog.md` — erreurs déjà diagnostiquées dans ce projet

3. **Vérifier l'existence** : Avant de créer, vérifier si une rule/skill similaire existe déjà
   - Lister `.devin/rules/`
   - Lister `.devin/skills/`
   - Si similaire → **mettre à jour** l'artifact existant plutôt qu'en créer un nouveau

   > **Règle** : Tous les artifacts vont dans `.devin/`. Le dossier `.windsurf/` est déprécié (legacy fallback uniquement). Ne jamais créer de nouvel artifact dans `.windsurf/`.

4. **Créer/modifier l'artifact** avec le bon format :
   - Rule : frontmatter `trigger` + `description` (si model_decision/glob), contenu concis
   - Skill : frontmatter `name` + `description`, procédure claire, fichiers supports
   - AGENTS.md : markdown simple, pas de frontmatter, instructions ciblées

   > **Règles de formatage SKILL.md critiques** (violation = skill non détecté par l'agent) :
   > - Frontmatter YAML avec **exactement 3 tirets** `---` (PAS 4 tirets `----`)
   > - Le frontmatter doit être **le tout premier contenu du fichier** — aucun titre ni commentaire avant
   > - La **description ne doit pas contenir `: ` (colon+espace)** — cela casse le parsing YAML. Remplacer par ` -` ou mettre la valeur entre guillemets
   > - Vérifier après création : `head -1 SKILL.md` (Linux/macOS) ou `Get-Content SKILL.md -TotalCount 1` (Windows PowerShell) doit afficher `---`

5. **Respecter les limites et best practices** :
   - Rule : simple, concise, spécifique. Pas de règles génériques (déjà dans le training data). Utiliser bullet points et markdown. < 12 000 caractères
   - Skill SKILL.md : < 500 lignes. Description claire pour l'invocation automatique
   - AGENTS.md : concis, focalisé sur le répertoire, pas de redondance avec les parents
   - Utiliser des XML tags pour grouper des règles similaires dans une rule

### Phase 4a — Rapport de diagnostic (avant action)

Avant de créer ou modifier des artifacts, présenter un rapport de diagnostic à l'utilisateur pour validation :

```
🔍 Diagnostic des erreurs de Devin Local

Conversation analysée : [nom/référence]

Erreurs identifiées :
1. [catégorie] — [description courte] → correction proposée : [type d'artifact]
2. [catégorie] — [description courte] → correction proposée : [type d'artifact]
...

J'ai lu la doc officielle (Phase 0) et vérifié les artifacts existants.
Je propose de créer/modifier les fichiers suivants :
- [chemin] (nouveau/modifié)
- [chemin] (nouveau/modifié)

Valides-tu ces modifications ?
```

> **Ne crée aucun artifact tant que l'utilisateur n'a pas validé.** Si l'utilisateur rediscute ou ajuste, adapter les propositions puis re-présenter.

### Phase 4b — Création et rapport de validation (après validation)

Après validation explicite de l'utilisateur :

1. **Créer/modifier les artifacts** (Phase 3)

2. **Vérifier la non-surcharge** :
   - Compter les rules `always_on` — trop = contexte gonflé
   - Si > 6 rules `always_on`, envisager de convertir certaines en `model_decision`

3. **Vérifier la non-duplication** :
   - La nouvelle rule ne duplique-t-elle pas une existante ?
   - Le nouveau skill ne chevauche-t-il pas un existant ?

4. **Mettre à jour l'inventaire** :
   - Si nouveau skill → ajouter à `.devin/AGENTS.md` (section Inventory des Skills)
   - Si nouvelle rule → ajouter à `.devin/AGENTS.md` (section Inventory des Rules)

5. **Mettre à jour le catalogue projet** : Ajouter les nouvelles entrées ERR-XXX dans `<projet>/.devin/memory/devin-self-config/diagnostic-catalog.md`

6. **Présenter le rapport de validation** :
   ```
   ✅ Auto-configuration appliquée

   Erreur diagnostiquée : [catégorie] — [description]
   Correction créée : [type d'artifact] → [chemin]
   Mode d'activation : [trigger]
   Impact contexte : [always_on/model_decision/glob/manual]

   Fichiers modifiés :
   - [chemin] (créé/modifié)
   - [chemin] (créé/modifié)

   Prêt à commiter. Veux-tu que je commite ces changements ?
   ```

### Phase 5 — Commit (après validation explicite)

> Ne commiter **qu'après validation explicite** de l'utilisateur sur le rapport de validation (Phase 4b).

1. **Ne commiter QUE les fichiers `.devin/` modifiés** dans le cadre de cette auto-configuration :
   - Rules (`.devin/rules/*.md`)
   - Skills (`.devin/skills/*/SKILL.md` et `references/`)
   - AGENTS.md (`.devin/AGENTS.md`)
   - Mémoire projet (`.devin/memory/devin-self-config/`)
2. **Ne jamais commiter** des fichiers de code applicatif (`lib/`, `functions/`, `test/`, etc.) dans ce workflow
3. **Message de commit** : `chore(devin-config): [description courte de la correction]`
4. **Un seul commit** regroupant toutes les modifications de config liées à cette session

## Checklist avant de répondre à l'utilisateur

- [ ] J'ai lu les 4 URLs de documentation (Phase 0) — **avant toute autre action**
- [ ] J'ai vérifié/migré la mémoire projet-spécifique (Phase 0b)
- [ ] J'ai utilisé `dcr` (ou le fallback manuel : contenu collé par l'utilisateur) pour analyser la conversation source (Phase 1)
- [ ] J'ai identifié chaque erreur de Devin Local dans la conversation
- [ ] J'ai catégorisé chaque erreur (cli-syntax, tool-selection, etc.)
- [ ] J'ai choisi un type d'artifact pour chaque correction (Phase 2)
- [ ] J'ai présenté le rapport de diagnostic et attendu la validation (Phase 4a)
- [ ] Après validation : j'ai créé/modifié les artifacts (Phase 3)
- [ ] J'ai présenté le rapport de validation (Phase 4b)
- [ ] J'ai attendu la validation de commit avant de commiter (Phase 5)

Si une case n'est pas cochée, **ne réponds pas encore** — complète l'étape manquante.

## Exemple de bon vs mauvais déroulement

> **Bon comportement** :
> 1. Lit les 4 URLs de doc (Phase 0)
> 2. Vérifie/crée la mémoire projet (Phase 0b)
> 3. `dcr show` / `dcr export` sur la conversation (Phase 1) — voir `references/dcr-diagnostic-patterns.md`
> 4. "J'ai identifié 3 erreurs : ERR-A (process-gap), ERR-B (missing-prerequisite), ERR-C (cli-syntax)"
> 5. Propose des corrections (Phase 2) → présente le rapport de diagnostic (Phase 4a)
> 6. L'utilisateur valide → crée les artifacts (Phase 3) → rapport de validation (Phase 4b)
> 7. L'utilisateur valide le commit → commit (Phase 5)

> **Mauvais comportement — NE PAS FAIRE** :
> 1. Saute la Phase 0 (doc non lue)
> 2. Résume la conversation au lieu de diagnostiquer les erreurs de Devin Local
> 3. Donne des recommandations sur le projet (logging, code, architecture)
> 4. Demande "Quelles actions veux-tu que je mette en œuvre ?" au lieu de proposer un diagnostic
> 5. Crée des artifacts sans validation utilisateur

## Fichiers de référence

### Références génériques (globales, dans le répertoire du skill)

| Fichier | Contenu | Quand le charger |
|---|---|---|
| `references/rules-guide.md` | Syntaxe, modes, exemples de rules | Avant de créer/modifier une rule |
| `references/skills-guide.md` | Structure, progressive disclosure, best practices | Avant de créer/modifier un skill |
| `references/agents-md-guide.md` | Scoping, format, comparaison avec rules | Avant de créer/modifier un AGENTS.md |
| `references/dcr-diagnostic-patterns.md` | Procédures dcr pour le diagnostic de l'agent (sync, search, show, export, compare) | Pendant la Phase 1 — diagnostic de conversation |

### Mémoire projet-spécifique (dans `<projet>/.devin/memory/devin-self-config/`)

| Fichier | Contenu | Quand le charger |
|---|---|---|
| `project-tooling.md` | Outils CLI, MCP, navigateur + limites de ce projet | Pour vérifier les capacités d'un outil |
| `diagnostic-catalog.md` | Erreurs connues et corrections appliquées dans ce projet | Pour éviter de réinventer une correction |

### Templates (globaux, pour création initiale)

| Fichier | Contenu |
|---|---|
| `references/project-tooling-template.md` | Template pour créer `project-tooling.md` dans un nouveau projet |
| `references/diagnostic-catalog-template.md` | Template pour créer `diagnostic-catalog.md` dans un nouveau projet |
