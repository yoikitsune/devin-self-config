---
name: devin-self-config
description: Auto-configuration de Devin Local pour améliorer son comportement. Analyser et corriger des erreurs ou optimiser des méthodes, diagnostiquer les causes, et créer/modifier les artifacts .devin (rules, skills, AGENTS.md) pour éviter la récurrence ou adopter de meilleures pratiques. Utiliser quand l'utilisateur dit "auto-configure-toi", "améliore ta config", "tu as fait une erreur", "tu pourrais faire mieux", "vérifie la conformité", "check alignment", ou après une session où Devin Local a été imprécis ou peu efficace.
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

- **`dcr` (Devin Conversations Retriever)** requis pour le **mode conversation** uniquement. Le **mode alignment** (ADR-0005) n'en a pas besoin.
  - Installation : `devin-conversations-retriever/scripts/install-skills.sh` (crée le wrapper `~/.local/bin/dcr` + le skill `dcr-conversation` global)
  - Vérifier : `dcr status` (depuis n'importe quel répertoire)
  - Si `dcr` n'est pas disponible, le mode conversation tombe sur le fallback manuel : demander à l'utilisateur de coller le contenu de la conversation à analyser (Devin Local n'a pas d'outil natif de recherche dans l'historique des conversations — `trajectory_search` était un outil Cascade, désormais EOL). Le mode alignment fonctionne sans `dcr`.

## Quand utiliser ce skill

### Mode conversation (diagnostic réactif)

- L'utilisateur signale une **erreur** de fonctionnement de Devin Local
- L'utilisateur suggère une **amélioration** de méthode ("tu pourrais faire X plus efficacement", "utilise plutôt tel outil dans ce cas")
- L'utilisateur dit "auto-configure-toi", "améliore ta config", "tu as fait une erreur", "tu pourrais faire mieux"
- Après une session de débogage où Devin Local a gaspillé des étapes (mauvaise syntaxe CLI, mauvais outil, manque de prérequis)
- L'utilisateur fournit une conversation passée à analyser
- L'utilisateur veut que Devin Local adopte une nouvelle pratique de travail

### Mode alignment (diagnostic préventif — ADR-0005)

- L'utilisateur invoque `@devin-self-config` **sans référence de conversation**
- L'utilisateur dit "vérifie la conformité", "check alignment", "est-ce que mes artifacts sont à jour"
- L'utilisateur développe un plugin qui crée des artifacts Devin (skills, rules, agents) et veut vérifier qu'ils respectent la doc courante
- L'utilisateur veut un audit préventif des artifacts `.devin/` du projet courant

## Ce que ce skill N'EST PAS

- ❌ Ce skill **n'est pas** un outil de résumé de conversation
- ❌ Ce skill **n'est pas** un outil de recommandations sur le **code applicatif** ou l'**architecture du projet** (logging, architecture, choix techniques)
- ❌ Ce skill **ne demande pas** à l'utilisateur quoi faire — il diagnostique, propose, puis agit après validation
- ✅ Mode conversation : analyse **le comportement de Devin Local** (erreurs, améliorations) et crée des corrections dans `.devin/`
- ✅ Mode alignment : analyse **la conformité des artifacts Devin** (format, limites, best practices) et les corrige

> Si tu te surprends à résumer la conversation, à faire une revue de code, ou à donner des recommandations sur l'architecture du projet, **tu es hors-sujet**. Reviens au diagnostic (comportement de Devin Local ou conformité des artifacts Devin).

## Procédure d'invocation attendue

L'utilisateur ouvre une **nouvelle conversation dédiée** (pas dans la conversation à analyser) et fournit :

1. `@devin-self-config` — invocation du skill
2. `@[conversation:...]` — référence de la conversation à analyser (mode conversation uniquement)
3. Optionnellement : une description explicite du problème observé

Trois modes d'analyse :
- **Mode explicite** (conversation) : l'utilisateur a décrit le problème → se concentrer sur ce point précis
- **Mode libre** (conversation) : l'utilisateur n'a rien décrit → parcours systématique de la conversation pour identifier les erreurs de Devin Local
- **Mode alignment** (sans conversation — ADR-0005) : l'utilisateur invoque le skill sans `@[conversation:...]` → diagnostiquer la conformité des artifacts Devin du projet courant avec la doc officielle, sans analyser de conversation

Le déroulement attendu est :

```
Phase 0  → lire la doc officielle (PREMIÈRE ACTION, avant toute analyse)
Phase 0b → vérifier/migrer la mémoire projet-spécifique
Phase 1  → diagnostic :
           ├─ mode conversation : dcr sur la conversation + diagnostic des erreurs
           └─ mode alignment : inspecter les artifacts .devin/ + comparer avec la doc
Phase 2  → choix d'artifact pour chaque correction
Phase 3  → création/modification des artifacts
Phase 4a → rapport de diagnostic à l'utilisateur (avant action)
    → l'utilisateur valide ou rediscute
Phase 4b → après validation : auto-review + rapport de ce qui a été créé/modifié
Phase 5  → commit après validation explicite de l'utilisateur
```

## Phase 0 — Chargement de la documentation officielle (OBLIGATOIRE — PREMIÈRE ACTION)

> ⚠️ **STOP** : Cette phase doit être exécutée **avant toute autre action**, y compris avant toute recherche avec `dcr` sur la conversation à analyser. Ne pas passer à la Phase 0b tant que les URLs n'ont pas été lues.

**À chaque invocation de ce skill**, lire les 4 pages de documentation officielle pour avoir une vue d'ensemble à jour du fonctionnement des outils de configuration de Devin Local :

1. **Devin Local** : `https://docs.devin.ai/desktop/devin-local` — présentation de l'agent Devin Local, ses modes (Normal/Plan/Ask), ses permissions (Deny/Ask/Allow), ses subagents, le sandboxing
2. **Skills — overview** : `https://docs.devin.ai/cli/extensibility/skills/overview` — format, découverte, scopes (projet/global), triggers (`user`/`model`)
3. **Skills — creating** : `https://docs.devin.ai/cli/extensibility/skills/creating-skills` — référence complète du format `SKILL.md` : frontmatter (`name`, `description`, `argument-hint`, `model`, `subagent`, `agent`, `allowed-tools`, `permissions`, `triggers`), contenu du prompt, exemples
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

#### Mode conversation (explicite ou libre)

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

#### Mode alignment (sans conversation — ADR-0005)

> Ce mode ne nécessite pas `dcr`. Il diagnostique la **conformité des artifacts
> existants** avec la doc officielle lue en Phase 0, pas le comportement de l'agent
> dans une conversation.

1. **Inventorier les artifacts Devin du projet courant** :
   ```bash
   ls .devin/rules/*.md 2>/dev/null
   ls .devin/skills/*/SKILL.md 2>/dev/null
   ls .devin/agents/*.md 2>/dev/null
   test -f AGENTS.md && echo "AGENTS.md present"
   ```

2. **Consulter le catalogue projet** : Lire `diagnostic-catalog.md` (projet) pour
   vérifier si des gaps similaires ont déjà été diagnostiqués et corrigés.

3. **Pour chaque artifact, vérifier la conformité avec la doc** (lue en Phase 0) et les
   guides `references/` correspondants. Checklist compacte — consulter le guide pour le
   détail des critères :

   **Rules** (`.devin/rules/*.md`) — critères détaillés dans `references/rules-guide.md` :
   - Frontmatter valide (`trigger`, `globs` si glob, `description` si model_decision/glob)
   - Taille < 12 000 caractères
   - Pas de règle générique ("write good code" — déjà dans le training data)

   **Skills** (`.devin/skills/*/SKILL.md`) — critères dans `references/skills-guide.md` :
   - Frontmatter valide (3 tirets `---`, `name`+`description`, pas de `: ` dans la description)
   - `head -1 SKILL.md` affiche `---`
   - Taille < 500 lignes
   - `allowed-tools` manquant → signaler (recommander restriction si safety-critique)
   - Fields utiles non utilisés : `subagent`, `agent`, `permissions`, `model`

   **Subagent profiles** (`.devin/agents/*.md`) — doc : `https://docs.devin.ai/cli/subagents` :
   - Frontmatter avec `name`, `description`, `allowed-tools`
   - `model:` présent ou justifié par ADR
   - Note : ces fichiers sont des profils de subagents custom, **différents** de `AGENTS.md` (rules)

   **AGENTS.md** — critères dans `references/agents-md-guide.md` :
   - Plain markdown, pas de frontmatter, concis, pas de redondance avec les rules

4. **Catégoriser les gaps** :
   - `format` : Violation de format (frontmatter cassé, champ manquant, `: ` dans description)
   - `size` : Limite dépassée (rule > 12 000 chars, skill > 500 lignes)
   - `best-practice` : Bonne pratique non respectée (allowed-tools manquant, rule trop générique)
   - `deprecated` : Usage d'un field ou pattern déprécié par la doc courante
   - `unused-capability` : Field ou capacité disponible que l'artifact pourrait exploiter

5. **Déterminer la correction** : Quelle modification de l'artifact le rendrait conforme
   à la doc courante ? (La correction modifie l'artifact existant, pas le comportement de
   l'agent — c'est la différence avec le mode conversation.)

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
   └─ AGENTS.md (section ponctuelle) ou rule model_decision
```

**Règle d'or** : Préférer `model_decision` à `always_on` pour économiser le contexte permanent. Une rule `always_on` ne se justifie que si l'erreur peut se reproduire à tout moment sans signal contextuel.

> **Recommandation officielle** : la doc conseille de **privilégier les Skills plutôt que les Rules** quand possible (les skills ne sont injectés dans le contexte que quand pertinent). Le pattern recommandé est d'utiliser une rule pour **référencer** les skills que le modèle doit utiliser dans des scénarios particuliers.

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

4. **Créer/modifier l'artifact** avec le bon format. **Consulter le guide
   correspondant** pour les règles de formatage détaillées (frontmatter, limites,
   best practices) — les valeurs clés ne sont pas répétées ici :
   - Rule : `references/rules-guide.md` (frontmatter `trigger` + `description`, < 12 000 caractères)
   - Skill : `references/skills-guide.md` (frontmatter `name` + `description`, < 500 lignes, **3 tirets `---` exactement**, pas de `: ` dans la description)
   - AGENTS.md : `references/agents-md-guide.md` (markdown simple, pas de frontmatter)

### Phase 4a — Rapport de diagnostic (avant action)

Avant de créer ou modifier des artifacts, présenter un rapport de diagnostic à l'utilisateur pour validation :

```
🔍 Diagnostic [mode conversation | mode alignment]

[Mode conversation] Conversation analysée : [nom/référence]
[Mode alignment] Artifacts inspectés : [N rules, N skills, N agents, AGENTS.md]

[Erreurs | Gaps] identifié·e·s :
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

2. **Auto-review des modifications** — appliquer les critères du mode alignment (Phase 1)
   aux artifacts fraîchement créés/modifiés. Le skill doit pratiquer ce qu'il diagnostique :
   - **Redondance interne** : le SKILL.md ne répète-t-il pas le contenu de ses `references/` ?
     Préférer un pointeur vers la référence plutôt qu'une copie inline
   - **Cohérence des modes** : si le skill supporte plusieurs modes (ex. conversation + alignment),
     les templates, rapports et exemples couvrent-ils tous les modes ?
   - **Limites** : `wc -l` et `wc -c` sur les fichiers modifiés — signaler si dépassé
   - **Description frontmatter** : couvre-t-elle tous les triggers d'invocation des modes supportés ?
   - **Non-surcharge** : compter les rules `always_on` — si > 6, envisager `model_decision`
   - **Non-duplication** : la nouvelle rule/skill ne chevauche-t-elle pas une existante ?

   > Si un gap est trouvé lors de l'auto-review, **le corriger immédiatement** avant de
   > passer à l'étape suivante. Ne pas présenter un rapport de validation sur un artifact
   > qui ne passe pas ses propres critères de diagnostic.

3. **Mettre à jour l'inventaire** :
   - Si nouveau skill → ajouter à `.devin/AGENTS.md` (section Inventory des Skills)
   - Si nouvelle rule → ajouter à `.devin/AGENTS.md` (section Inventory des Rules)

4. **Mettre à jour le catalogue projet** : Ajouter les nouvelles entrées ERR-XXX dans `<projet>/.devin/memory/devin-self-config/diagnostic-catalog.md`

5. **Présenter le rapport de validation** :
   ```
   ✅ Auto-configuration appliquée

   [Erreur | Gap] diagnostiqué·e : [catégorie] — [description]
   Correction créée : [type d'artifact] → [chemin]
   Mode d'activation : [trigger]
   Impact contexte : [always_on/model_decision/glob/manual]
   Auto-review : [✅ passé | ⚠️ N gaps corrigés avant rapport]

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
- [ ] J'ai identifié le mode (conversation ou alignment) et suivi la Phase 1 correspondante
- [ ] **Mode conversation** : j'ai utilisé `dcr` (ou le fallback manuel : contenu collé par l'utilisateur) pour analyser la conversation source
- [ ] **Mode alignment** : j'ai inventorié les artifacts `.devin/` et les ai comparés à la doc
- [ ] J'ai identifié chaque erreur/gap et catégorisé (cli-syntax, tool-selection, format, size, etc.)
- [ ] J'ai choisi un type d'artifact pour chaque correction (Phase 2)
- [ ] J'ai présenté le rapport de diagnostic et attendu la validation (Phase 4a)
- [ ] Après validation : j'ai créé/modifié les artifacts (Phase 3)
- [ ] J'ai fait l'auto-review (Phase 4b étape 2) — redondance, cohérence, limites, description
- [ ] J'ai présenté le rapport de validation (Phase 4b)
- [ ] J'ai attendu la validation de commit avant de commiter (Phase 5)

Si une case n'est pas cochée, **ne réponds pas encore** — complète l'étape manquante.

## Exemple de bon vs mauvais déroulement

> **Bon comportement — mode conversation** :
> 1. Lit les 4 URLs de doc (Phase 0)
> 2. Vérifie/crée la mémoire projet (Phase 0b)
> 3. `dcr show` / `dcr export` sur la conversation (Phase 1) — voir `references/dcr-diagnostic-patterns.md`
> 4. "J'ai identifié 3 erreurs : ERR-A (process-gap), ERR-B (missing-prerequisite), ERR-C (cli-syntax)"
> 5. Propose des corrections (Phase 2) → présente le rapport de diagnostic (Phase 4a)
> 6. L'utilisateur valide → crée les artifacts (Phase 3) → **auto-review** (Phase 4b étape 2) → rapport de validation (Phase 4b)
> 7. L'utilisateur valide le commit → commit (Phase 5)

> **Bon comportement — mode alignment** :
> 1. Lit les 4 URLs de doc (Phase 0)
> 2. Vérifie/crée la mémoire projet (Phase 0b)
> 3. Inventorie les artifacts `.devin/` du projet (Phase 1 alignment)
> 4. "J'ai identifié 2 gaps : GAP-A (format — description avec `: ` dans pp-plan), GAP-B (unused-capability — allowed-tools manquant sur 3 skills)"
> 5. Propose des corrections (Phase 2) → présente le rapport de diagnostic (Phase 4a)
> 6. L'utilisateur valide → modifie les artifacts (Phase 3) → **auto-review** (Phase 4b étape 2) → rapport de validation (Phase 4b)
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
