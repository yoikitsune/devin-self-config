# Patterns diagnostiques avec `dcr` — devin-self-config

> Ce fichier contient les procédures spécifiques pour utiliser `dcr` dans le contexte du diagnostic de comportement de Devin Local. Il est chargé par le skill `devin-self-config` lors de la Phase 1.
>
> Pour la documentation générique de `dcr`, utiliser `dcr --help` ou consulter `/home/julien/Sources/devin-conversations-retriever/README.md`.

## Prérequis

- `dcr` est installé globalement et accessible sur PATH (via le wrapper `~/.local/bin/dcr` créé par `devin-conversations-retriever/scripts/install-skills.sh`). Chemin de fallback : `/home/julien/Sources/devin-conversations-retriever/.venv/bin/dcr`
- La DB est à `~/.local/share/dcr/dcr.db`
- **Auto-sync** : toutes les commandes `dcr` font un sync automatique avant de s'exécuter (sauf `--no-sync`). Ne **pas** lancer `dcr sync` manuellement avant une autre commande — c'est redondant. Si la DB n'existe pas, la première commande la créera automatiquement.

## Pattern 1 — Analyser une conversation spécifique

**Quand** : l'utilisateur fournit une conversation à analyser (mode explicite ou libre).

```bash
# 1. Lister les conversations récentes pour trouver l'ID
#    (auto-sync s'effectue avant la commande — pas besoin de dcr sync manuel)
dcr list -l 10

# 2. Afficher la conversation complète (prefix d'ID suffisant)
dcr show <id_prefix>

# 3. Exporter en markdown pour analyse approfondie
dcr export <id_prefix> -o /tmp/conversation-analysis.md
```

> **Avantage sur le fallback manuel** : `dcr show` retourne la conversation complète sans limite. `dcr export` produit un markdown structuré par rounds/steps, plus facile à analyser qu'une conversation collée en bloc. (Devin Local n'a pas d'outil natif de recherche d'historique — `trajectory_search` était un outil Cascade, désormais EOL.)

## Pattern 2 — Rechercher des erreurs récurrentes across conversations

**Quand** : l'utilisateur signale un problème récurrent ou veut identifier des patterns d'erreurs.

```bash
# Rechercher par mot-clé d'erreur (auto-sync avant chaque commande)
dcr search "command not found"
dcr search "error: "
dcr search "failed to"

# Filtrer par projet
dcr search "erreur" -p <project_name>

# Rechercher des patterns spécifiques
dcr search "command not found"
dcr search "exec"
dcr search "MCP"
```

## Pattern 3 — Comparer deux conversations

**Quand** : l'utilisateur veut comparer une session réussie avec une session échouée.

```bash
# Exporter les deux conversations (auto-sync avant chaque commande)
dcr export <id1_prefix> -o /tmp/conv1.md
dcr export <id2_prefix> -o /tmp/conv2.md

# Lire et comparer manuellement les deux exports
```

## Pattern 4 — Diagnostiquer un problème de sélection d'outil

**Quand** : Devin Local a utilisé le mauvais outil (MCP au lieu de CLI, navigateur au lieu de script, etc.).

```bash
# Chercher les mentions d'outils dans les conversations
dcr search "MCP"
dcr search "browser_preview"
dcr search "exec"
dcr search "grep"

# Voir les conversations où ces outils ont été mentionnés
dcr list -l 20
dcr show <id_prefix>
```

## Pattern 5 — Identifier des étapes gaspillées

**Quand** : Devin Local a fait trop d'étapes pour une tâche simple.

```bash
# Exporter la conversation
dcr export <id_prefix> -o /tmp/conv.md

# Analyser le markdown exporté :
# - Compter les rounds (séparés par ## Round N)
# - Identifier les étapes de recherche vs action
# - Repérer les allers-retours inutiles
```

## Pattern 6 — Vérifier si une erreur a déjà été diagnostiquée

**Quand** : avant de créer une nouvelle correction, vérifier si le même pattern d'erreur existe dans d'autres conversations.

```bash

# Rechercher le pattern d'erreur
dcr search "<pattern_specifique>"

# Voir dans quelles conversations ce pattern apparaît
dcr list -l 20
```

## Interprétation des résultats

### Format de sortie `dcr search`

```
[conversation_id] [date] [project] [round/step] [snippet]
```

- **conversation_id** : prefix utilisable avec `dcr show`
- **project** : projet dans lequel la conversation a eu lieu
- **snippet** : extrait autour du match, avec `<query>` surligné

### Format de sortie `dcr show`

Affiche :
- Métadonnées (ID, date, projet, modèle, titre)
- Rounds (regroupements logiques)
- Steps (actions individuelles : user, assistant, tool_call, tool_result, checkpoint)
- Checkpoints (résumés de session)

### Format de sortie `dcr export`

Markdown structuré :
- `# Conversation <id>` — en-tête
- `## Round N` — chaque round
- `### Step N (type)` — chaque step avec son type et contenu
- `### Checkpoint N` — checkpoints

## Limites et fallbacks

- Si `dcr` n'est pas installé ou la DB est vide → fallback manuel : demander à l'utilisateur de coller le contenu de la conversation à analyser (Devin Local n'a pas d'outil natif de recherche d'historique)
- Si la conversation recherchée n'est pas dans la DB → l'auto-sync l'ajoutera à la prochaine commande `dcr` (pas besoin de `dcr sync` manuel)
- Si la conversation a été supprimée par Windsurf mais est dans la DB → `dcr show` fonctionne (archive permanente)
- `dcr search` ne fait pas de recherche sémantique (FTS5 keyword only) — pour une recherche sémantique dans le code, utiliser `code_search` en complément (mais `code_search` ne couvre pas l'historique des conversations)
