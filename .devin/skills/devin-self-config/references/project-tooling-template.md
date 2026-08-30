# Outils disponibles pour ce projet — Capacités et Limites

> Ce fichier est projet-spécifique. Il est créé automatiquement par le skill `devin-self-config` lors de la première invocation dans un projet. Adaptez-le avec les outils de votre projet.

## CLI [Nom de l'outil]

### Commandes valides
| Commande | Usage | Notes |
|---|---|---|
| commande1 | Usage 1 | Note 1 |
| commande2 | Usage 2 | Note 2 |

### Pièges connus
- Piège 1
- Piège 2

## MCP [Nom du MCP]

### Capacités
| Tool | Ce qu'il fait |
|---|---|
| tool1 | Description |
| tool2 | Description |

### Limites connues
- Limite 1
- Limite 2

### Bonnes pratiques
- Pratique 1
- Pratique 2

## Navigateur (Playwright)

### Cas d'usage
- Cas 1
- Cas 2

### Bonnes pratiques
- Préférer `browser_snapshot` à `browser_take_screenshot` pour l'interaction
- Consulter la console avec `browser_console_messages` pour les erreurs JS
- Vérifier le réseau avec `browser_network_requests` pour les appels API

## Outils Devin Local internes

| Tool | Ce qu'il fait |
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

> **Note** : Devin Local n'a pas d'outil natif de recherche dans l'historique des conversations (`trajectory_search` était un outil Cascade, désormais EOL). Pour l'analyse de conversations passées, utiliser `dcr` (Devin Conversations Retriever) — voir `references/dcr-diagnostic-patterns.md`.
