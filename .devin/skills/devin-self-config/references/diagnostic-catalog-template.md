# Catalogue d'erreurs diagnostiquées et corrections appliquées

> Ce fichier est projet-spécifique. Il est créé automatiquement par le skill `devin-self-config` lors de la première invocation dans un projet. Il s'enrichit à chaque utilisation du skill pour documenter les erreurs déjà identifiées et leurs corrections dans ce projet.

## Template pour nouvelle entrée

```markdown
## ERR-XXX : <titre court>

**Catégorie** : <cli-syntax|tool-selection|missing-prerequisite|api-misuse|code-error|process-gap|security|efficiency|practice-adoption|format|size|best-practice|deprecated|unused-capability>
**Date** : <YYYY-MM-DD>
**Source** : <"conversation: <nom>" | "alignment check">

### Erreur
<Description de l'erreur, commande exacte, message d'erreur>
<En mode alignment : description du gap de conformité, artifact concerné, écart avec la doc>

### Correction appliquée
<Description de la correction, artifact créé ou modifié>

### Artifact créé
- <chemin de l'artifact> (type: rule/skill/AGENTS.md/reference)
```

## Catégories par mode

**Mode conversation** (diagnostic réactif) :
- `cli-syntax` : Erreur de syntaxe dans une commande CLI
- `tool-selection` : Mauvais choix d'outil pour la tâche
- `missing-prerequisite` : Prérequis d'environnement manquant
- `api-misuse` : Mauvaise utilisation d'une API
- `code-error` : Bug dans le code produit
- `process-gap` : Mauvaise anticipation d'un flux ou d'une intégration
- `security` : Problème de sécurité (secret committé, commande insecure, permission excessive)
- `efficiency` : Méthode fonctionnelle mais sous-optimale (trop d'étapes, outil suboptimal)
- `practice-adoption` : Nouvelle pratique à systématiser

**Mode alignment** (diagnostic préventif — ADR-0005) :
- `format` : Violation de format (frontmatter cassé, champ manquant, `: ` dans description)
- `size` : Limite dépassée (rule > 12 000 chars, skill > 500 lignes)
- `best-practice` : Bonne pratique non respectée (allowed-tools manquant, rule trop générique)
- `deprecated` : Usage d'un field ou pattern déprécié par la doc courante
- `unused-capability` : Field ou capacité disponible que l'artifact pourrait exploiter
