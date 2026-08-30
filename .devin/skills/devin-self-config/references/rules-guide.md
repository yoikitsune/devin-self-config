# Guide des Rules Devin Local

## Syntaxe

Une rule est un fichier markdown avec un frontmatter YAML obligatoire :

```yaml
---
trigger: always_on | model_decision | glob | manual
description: Description du contexte (requis pour model_decision/glob)
---
```

### Modes d'activation

| Trigger | Quand s'applique | Usage typique |
|---|---|---|
| `always_on` | Toujours chargé dans le contexte | Erreurs critiques qui peuvent survenir à tout moment sans signal contextuel |
| `model_decision` | Devin Local décide quand charger en fonction du contexte | Préférence par défaut pour économiser le contexte |
| `glob` | S'applique uniquement aux fichiers correspondant au pattern | Règles spécifiques à certains types de fichiers |
| `manual` | Jamais chargé automatiquement, invocation explicite via @rule-name | Documentation ou procédures rarement utilisées |

### Syntaxe glob

Le trigger `glob` utilise `globs` (pluriel) pour spécifier les patterns :

```yaml
---
trigger: glob
globs: **/*.test.ts
description: Règles pour les fichiers de test
---
```

Patterns valides : `*.js`, `src/**/*.ts`, `**/*.test.ts`, etc.

## Emplacements de stockage

### Rules projet
- `.devin/rules/*.md` (emplacement recommandé — un fichier par rule, avec frontmatter `trigger`)
- `.devin/global_rules.md` (fichier unique always-on, alternative au format directory)
- `.windsurf/rules/*.md` (legacy, déprécié — `.devin/` prend le pas)

### Rules globales (utilisateur)
<Tabs>
  <Tab title="Linux / macOS">
    - `~/.config/devin/AGENTS.md` (fichier unique, appliqué à tous les projets — always-on)
    - `~/.devin/rules/*.md` (un fichier par rule, avec frontmatter `trigger`)
    - `~/.devin/global_rules.md` (fichier unique always-on)
  </Tab>
  <Tab title="Windows">
    - `%APPDATA%\devin\AGENTS.md` (fichier unique, appliqué à tous les projets — always-on)
    - `~/.devin/rules/*.md` et `~/.devin/global_rules.md` (mêmes chemins que Linux/macOS via le home)
  </Tab>
</Tabs>

> **Précédence** : `.devin/` est l'emplacement préféré et prend le pas sur `.windsurf/`. Si `.devin/global_rules.md` et `.windsurf/global_rules.md` existent tous les deux, seul `.devin/global_rules.md` est chargé. Les fichiers de rules dans `.devin/rules/` et `.windsurf/rules/` sont eux chargés en complément.

### Rules système (Enterprise)
- `/etc/devin/rules/` ou `/etc/windsurf/rules/` (déployées par IT, read-only pour l'utilisateur)

### Discovery
- Workspace et sous-répertoires : tous les `.devin/rules/` trouvés
- Git : recherche jusqu'au git root dans les répertoires parents
- Multi-workspace : déduplication avec le chemin relatif le plus court
- Les rules en sous-répertoires sont découvertes paresseusement quand l'agent accède à des fichiers dans ce répertoire

### Imports depuis autres outils
Devin Local peut lire les rules d'autres outils AI. Contrôler quels formats sont importés via `read_config_from` dans `config.json` (`~/.config/devin/config.json` ou `%APPDATA%\devin\config.json` sur Windows, ou `.devin/config.json`) :

```json
{
  "read_config_from": {
    "agents_standard": true,
    "cursor": true,
    "windsurf": true,
    "claude": true
  }
}
```

Formats supportés : `.cursor/rules/*.md` et `.mdc`, `.windsurf/rules/*.md`, `.claude/` (Claude Code), `AGENTS.md`/`AGENTS.local.md`/`AGENT.md`/`.windsurfrules` (standard AGENTS, activé par défaut).

## Exemples

### always_on (erreur critique)

```yaml
---
trigger: always_on
---

Toujours vérifier les imports avant de les utiliser. Ne jamais inventer de noms de classes.
```

### model_decision (contextuel)

```yaml
---
trigger: model_decision
description: Quand l'utilisateur travaille avec Firebase CLI
---

Pour les logs Firebase, utiliser `firebase functions:log --project <id> | tail -N` plutôt que `--limit` (option inexistante).
```

### glob (fichiers spécifiques)

```yaml
---
trigger: glob
globs: **/*.dart
description: Pour les fichiers Dart dans le projet
---

Dans les fichiers Dart, utiliser uniquement les imports absolus `package:project/...` jamais les chemins relatifs.
```

## Best practices (documentation officielle)

- **Garder les rules simples, concises et spécifiques**. Les rules trop longues ou vagues peuvent confondre Devin Local.
- **Pas de règles génériques** (ex: "write good code") — déjà dans le training data de Devin Local.
- **Formater avec bullet points, numbered lists, et markdown** — plus facile à suivre qu'un long paragraphe.
- **XML tags pour grouper des règles similaires** :

```xml
<coding_guidelines>
- My project's programming language is python
- Use early returns when possible
- Always add documentation when creating new functions and classes
</coding_guidelines>
```

## Limites

- Rule : < 12 000 caractères
- Préférer `model_decision` à `always_on` pour économiser le contexte permanent
- Si > 6 rules `always_on`, envisager de convertir certaines en `model_decision`
- Pour les procédures complexes multi-étapes, créer un skill à la place
