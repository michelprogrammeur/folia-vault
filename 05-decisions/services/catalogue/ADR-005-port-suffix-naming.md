# ADR-005 — Suffixe Port pour les interfaces de repositories

## Statut
Accepté

## Contexte
En hexagonal architecture, les interfaces de repositories sont des ports. Il faut choisir une convention de nommage : préfixe `I` (style C#/Java) ou suffixe `Port`.

## Options considérées

### Option 1 — Préfixe `I`
Convention issue de C# et Java : `IPlantRepository`.
- ✅ Très répandu dans les langages statiquement typés
- ❌ Peu naturel en TypeScript
- ❌ Le préfixe `I` n'apporte pas d'information sémantique sur le rôle
- ❌ Style daté — déconseillé dans les guidelines TypeScript modernes

### Option 2 — Suffixe `Port` ✅
Convention de l'architecture hexagonale : `PlantRepositoryPort`.
- ✅ Terme métier de l'architecture hexagonale — communique l'intention
- ✅ Plus naturel en TypeScript
- ✅ Indique clairement que c'est un port (point d'entrée/sortie du domaine)
- ✅ Cohérent avec la terminologie DDD/hexagonale utilisée dans le projet

### Option 3 — Pas de convention particulière
`PlantRepository` pour l'interface, `PostgresPlantRepository` pour l'implémentation.
- ✅ Simple
- ❌ Ambiguïté — on ne sait pas immédiatement si c'est une interface ou une classe
- ❌ Risque de collision de noms

## Décision
**Suffixe `Port` pour les interfaces de repositories.**

## Raisons
- Cohérence avec la terminologie de l'architecture hexagonale
- Communique clairement le rôle de l'interface dans l'architecture
- Préférence explicite du développeur principal — le préfixe `I` n'est pas apprécié

## Conséquences
- Toutes les interfaces de repositories se terminent par `Port` : `PlantRepositoryPort`, `SpeciesRepositoryPort`
- Les implémentations sont préfixées par la technologie : `PostgresPlantRepository`
- La convention s'applique à tous les services du monorepo
