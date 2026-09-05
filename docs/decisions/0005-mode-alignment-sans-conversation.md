# ADR-0005: Mode alignment — diagnostic de conformité sans conversation source

> Status: Accepted
> Date: 2026-09-04

## Context

Le skill `devin-self-config` est exclusivement axé sur l'analyse d'une conversation
passée pour diagnostiquer le comportement de Devin Local. Sa procédure d'invocation
dit explicitement :

> "L'utilisateur ouvre une nouvelle conversation dédiée et fournit
> `@devin-self-config` + `@[conversation:...]`"

La Phase 1 entière est construite autour de `dcr` sur une conversation source. Sans
conversation, le skill n'a pas de diagnostic à produire.

### Le gap

Il existe un cas d'usage légitime que le skill ne couvre pas : **vérifier la conformité
des artifacts Devin d'un projet avec la doc courante, sans conversation à analyser**.

Ce cas se présente notamment quand on développe un plugin qui crée des artifacts Devin
(comme `OUTILS_16_pair-programming-agentic` qui shippe 8 skills, 4 agents, et 3 rules).
On veut s'assurer que les artifacts respectent le format courant (frontmatter fields,
limites de taille, bonnes pratiques) — mais il n'y a pas de "conversation à analyser",
juste des fichiers à vérifier contre la doc.

Aujourd'hui, cette vérification se fait manuellement : lire les 4 URLs de doc (Phase 0),
puis comparer chaque artifact à la main. C'est exactement ce que devin-self-config sait
faire en Phase 0 — mais seulement quand on lui fournit une conversation.

### Ce qui manque

devin-self-config a la capacité de lire et comprendre la doc officielle (Phase 0), mais
ne l'utilise que pour valider les artifacts qu'il crée en réponse à un diagnostic de
conversation. Il n'exploite pas cette capacité pour **diagnostiquer la conformité des
artifacts existants** d'un projet.

### Origine

Ce mode alignment est né d'un besoin concret sur le projet `OUTILS_16_pair-programming-agentic`
(septembre 2026). L'utilisateur travaillait sur OUTILS_16 (un plugin qui crée des artifacts
Devin) et a ajouté devin-self-config au workspace pour vérifier la conformité des
artifacts sans avoir de conversation à analyser. Le mode alignment est donc un cas
d'usage **cross-projet** : il permet à devin-self-config de vérifier la conformité
d'artifacts depuis n'importe quel projet qui en crée.

L'auto-review Phase 4b (systématisation d'une passe de vérification après modification
d'artifacts) vient d'une remarque de l'utilisateur : "ce skill est censé savoir tout
seul qu'il doit faire cette passe de vérification à chaque fois qu'il fait des
modifications dans les artifacts devin". L'auto-review est la systématisation d'une
pratique observée et validée pendant cette session.

## Decision

Ajouter un **mode alignment** au skill, qui se déclenche quand devin-self-config est
invoqué **sans référence de conversation**.

### Détection du mode

| Signal | Mode |
|---|---|
| `@[conversation:...]` fourni ou problème décrit | **Mode conversation** (existant) |
| `@devin-self-config` sans conversation, ou "vérifie la conformité", "check alignment", "est-ce que mes artifacts sont à jour" | **Mode alignment** (nouveau) |

### Procédure du mode alignment

1. **Phase 0** (existante, inchangée) : lire les 4 URLs de doc officielle
2. **Phase 0b** (existante, inchangée) : vérifier/migrer la mémoire projet
3. **Phase 1 — Mode alignment** (nouvelle branche) : au lieu d'analyser une conversation
   avec `dcr`, inspecter les artifacts du projet courant et les comparer avec la doc lue
   en Phase 0 :
   - Lister `.devin/rules/`, `.devin/skills/`, `agents/`, `AGENTS.md`
   - Pour chaque artifact, vérifier :
     - Format du frontmatter (fields valides, pas de `: ` dans la description)
     - Limites de taille (rules < 12 000 caractères, skills < 500 lignes)
     - Bonnes pratiques (allowed-tools sur les skills safety-critiques, trigger approprié)
     - Fields non utilisés qui pourraient être pertinents (subagent, permissions, etc.)
     - Cohérence avec la doc courante (dépréciations, nouveaux fields, changements de format)
   - Signaler les gaps catégorisés (`format`, `size`, `best-practice`, `deprecated`,
     `unused-capability`)
4. **Phases 2-5** (existantes, inchangées) : choix d'artifact, création, rapport, commit
   — la procédure de correction est la même, seul le diagnostic change

### Ce qui ne change pas

- Le mode conversation existe et fonctionne exactement comme avant
- La Phase 0 (lecture des 4 URLs) est partagée entre les deux modes
- Les Phases 2-5 (correction, validation, commit) sont identiques
- La mémoire projet (`diagnostic-catalog.md`, `project-tooling.md`) est utilisée dans
  les deux modes
- `dcr` reste un prérequis pour le mode conversation, mais **n'est pas requis** pour le
  mode alignment

## Consequences

- **Positives** :
  - devin-self-config couvre deux cas d'usage complémentaires : réactif (conversation)
    et préventif (alignment)
  - Les plugins qui créent des artifacts Devin (OUTILS_16, etc.) peuvent invoquer
    `@devin-self-config` sans conversation pour vérifier leur conformité
  - Le lien entre devin-self-config et les projets qui créent des artifacts Devin devient
    explicite — devin-self-config est la source de vérité pour la conformité, pas
    seulement pour le diagnostic d'erreurs
  - Le mode alignment ne nécessite pas `dcr` — utilisable même si dcr n'est pas installé
  - L'auto-review Phase 4b assure que le skill pratique ce qu'il diagnostique — les
    artifacts créés passent les mêmes critères de conformité que ceux diagnostiqués
    en mode alignment

- **Négatives** :
  - Le SKILL.md grandit (nouvelle branche dans la Phase 1) — à surveiller pour rester
    sous 500 lignes
  - Deux modes à documenter et maintenir — mais la procédure de correction (Phases 2-5)
    est partagée

- **Risques** :
  - **Confusion entre les modes** : mitigation par détection claire (présence/absence de
    référence conversation)
  - **Mode alignment trop large** : mitigation par focus sur les artifacts Devin
    (`.devin/`, `agents/`, `AGENTS.md`) — pas de revue de code applicatif

## Relations

- **Étend** : la Phase 1 du SKILL.md (ajoute une branche "mode alignment" à côté des
  modes explicite et libre existants)
- **Conserve** : Phases 0, 0b, 2-5 inchangées
- **Réutilise** : la Phase 0 (lecture des 4 URLs) qui existait déjà mais n'était exploitée
  que pour valider les artifacts créés, pas pour diagnostiquer les artifacts existants
- **Dépend de** : ADR-0004 (noms d'outils canoniques + guides refondus) — le mode
  alignment utilise les guides `references/` refondus par ADR-0004 pour vérifier la
  conformité. Sans cette refonte, les critères de conformité seraient basés sur une
  doc obsolète
- **Crée le lien avec** : `OUTILS_16_pair-programming-agentic` et tout projet qui crée
  des artifacts Devin — devin-self-config devient utilisable comme outil de vérification
  de conformité, pas seulement de diagnostic d'erreurs
