# ADR-012 — Stratégie de lecture : PostgreSQL vs moteur de recherche

## Statut
Accepté

## Contexte
Les services Folia exposent des endpoints de lecture (queries) consommés par la gateway. Selon la complexité de la query, différentes sources de données sont plus adaptées.

## Décision

### Queries simples → PostgreSQL direct
- `getById`, `listWithFilters` basiques (pagination, tri, filtres sur colonnes indexées)
- Lecture directe avec Drizzle ORM + index SQL sur les colonnes fréquemment filtrées
- Suffisant pour la grande majorité des cas

### Queries complexes → MeiliSearch
- Recherche full-text (e.g. `SearchPlants("monstera")`)
- Autocomplete sur les noms de plantes
- Recherche facettée (filtres combinés type + difficulté + lumière)
- PostgreSQL n'est pas conçu pour ces cas — MeiliSearch est plus performant et pertinent

## Responsabilité de l'indexation
L'indexation MeiliSearch est gérée par le service lui-même via les domain event handlers :

```
OnPlantPublished  → indexe la plante dans MeiliSearch
OnPlantUpdated    → met à jour l'index
OnPlantDeleted    → retire de l'index
```

La gateway appelle uniquement les endpoints HTTP du service — elle ne sait pas si la donnée vient de PostgreSQL ou MeiliSearch.

## Options considérées

### Option 1 — PostgreSQL uniquement
- ✅ Simple, pas de service supplémentaire
- ❌ Full-text search limité (`ILIKE`, `tsvector`) — peu pertinent sur les noms botaniques multilingues
- ❌ Performances dégradées sur les recherches complexes à grande échelle

### Option 2 — PostgreSQL + MeiliSearch ✅
- ✅ Chaque outil utilisé pour ce qu'il fait bien
- ✅ MeiliSearch : typo-tolerant, multilingue, ultra-rapide
- ✅ L'indexation event-driven garde les deux sources synchronisées
- ❌ Complexité opérationnelle supplémentaire (un service de plus à maintenir)

### Option 3 — Elasticsearch
- ✅ Plus puissant que MeiliSearch
- ❌ Beaucoup plus lourd à opérer
- ❌ Surdimensionné pour Folia au stade actuel

## Conséquences
- Les queries simples utilisent Drizzle directement dans les query handlers
- Les queries de recherche utilisent un `MeiliSearch<Entity>Repository` adapter
- Un index partiel SQL `WHERE deleted_at IS NULL` est créé sur toutes les tables principales
- MeiliSearch n'est pas introduit avant que la recherche soit un besoin réel (YAGNI)
- En cas de désynchronisation index/DB, un job de réindexation complète doit être possible
