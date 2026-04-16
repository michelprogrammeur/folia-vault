# ADR-003 — Pattern CQRS pour la couche application

## Statut
Accepté

## Contexte
La couche application orchestre la logique métier. Il faut choisir comment structurer les use cases : service applicatif classique ou CQRS.

## Options considérées

### Option 1 — Service applicatif classique
Une classe `PlantService` avec toutes les méthodes : `create`, `update`, `delete`, `findById`, `findAll`.
- ✅ Simple, familier
- ❌ La classe grossit avec le temps — God Object
- ❌ Difficile de tester chaque use case isolément
- ❌ Lecture et écriture mélangées — modèles souvent incompatibles

### Option 2 — CQRS (Command Query Responsibility Segregation) ✅
Séparation stricte entre commandes (mutations) et queries (lectures).
```
commands/CreatePlant/CreatePlantCommand.ts + CreatePlantHandler.ts
queries/GetPlant/GetPlantQuery.ts + GetPlantHandler.ts
```
- ✅ Chaque use case dans son propre fichier — responsabilité unique
- ✅ Facile à tester isolément
- ✅ Les queries peuvent optimiser leurs lectures indépendamment
- ✅ Scalable — les handlers peuvent être distribués
- ❌ Plus de fichiers à créer
- ❌ Peut sembler over-engineered pour de petits projets

## Décision
**CQRS avec Commands et Queries séparés.**

## Raisons
- Folia a des besoins de lecture et d'écriture très différents
- Chaque use case est une unité testable et compréhensible
- Le pattern s'aligne naturellement avec les événements domaine
- La séparation facilite l'optimisation des queries sans impacter les commandes

## Conséquences
- Chaque command/query dans son propre dossier avec son fichier + handler
- Les handlers sont nommés `<Action><Entité>Handler`
- Les commands retournent uniquement un ID ou void
- Les queries retournent un résultat typé (`GetPlantResult`, `ListPlantsResult`)
- Un `PlantMapper` partagé évite la duplication entre les queries
