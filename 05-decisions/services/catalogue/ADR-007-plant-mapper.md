# ADR-007 — PlantMapper partagé entre les queries

## Statut
Accepté

## Contexte
Les queries (`GetPlantHandler`, `ListPlantsHandler`) transforment des lignes Drizzle en DTOs de réponse HTTP. Cette transformation peut être dupliquée ou centralisée.

## Options considérées

### Option 1 — Mapping inline dans chaque handler
Chaque handler transforme lui-même les données Drizzle en réponse.
- ✅ Simple — tout le code dans un seul endroit
- ❌ Duplication — `GetPlant` et `ListPlants` mappent les mêmes champs
- ❌ Incohérence possible si un champ change — risque d'oublier un handler
- ❌ Tests de mapping dupliqués

### Option 2 — PlantMapper centralisé ✅
Une classe `PlantMapper` avec une méthode statique `toResponse(row)`.
- ✅ Single source of truth pour la transformation domaine → DTO
- ✅ Facile à tester isolément
- ✅ Un seul endroit à modifier si la structure de réponse change
- ✅ Nommage explicite — communique l'intention

### Option 3 — Mapping dans le repository
Le repository retourne directement des DTOs formatés.
- ❌ Viole la séparation des responsabilités — le repository ne doit pas connaître la forme des réponses HTTP
- ❌ Couplage fort entre la couche infrastructure et la couche interface

## Décision
**PlantMapper centralisé dans la couche application.**

## Raisons
- Évite la duplication entre handlers
- La logique de mapping est testable unitairement
- Un changement de DTO n'impacte qu'un seul fichier

## Conséquences
- `features/plant/application/queries/shared/PlantMapper.ts` contient le mapper
- `PlantMapper.toResponse(row: DrizzlePlantRow): PlantResponse` est la méthode principale
- `GetPlantHandler` et `ListPlantsHandler` importent et utilisent `PlantMapper`
- Les tests du mapper couvrent tous les champs et les cas limites (valeurs null, arrays vides...)
- Si un nouveau champ est ajouté à la réponse, le mapper est le seul fichier à modifier
