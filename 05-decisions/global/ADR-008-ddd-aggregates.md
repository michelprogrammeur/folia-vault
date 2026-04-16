# ADR-008 — DDD et agrégats comme modèle de conception

## Statut
Accepté

## Contexte
Folia est un projet complexe avec une logique métier riche (plantes, collections, identification, gamification...). Il faut choisir un style architectural pour modéliser le domaine.

## Options considérées

### Option 1 — Architecture en couches classique (MVC)
Controllers → Services → Repositories. Logique métier dans les services.
- ✅ Simple à comprendre
- ✅ Très répandu
- ❌ Les services deviennent des "god objects" avec le temps
- ❌ La logique métier est dispersée dans les services
- ❌ Difficile à tester sans infrastructure
- ❌ Les invariants métier ne sont pas protégés au niveau objet

### Option 2 — Domain-Driven Design (DDD) avec agrégats ✅
Entités riches, agrégats, value objects, domain events. Logique métier encapsulée dans le domaine.
- ✅ La logique métier est au cœur — testable sans infrastructure
- ✅ Les invariants sont protégés dans les agrégats (`plant.publish()` valide les règles)
- ✅ Les domain events permettent le couplage lâche entre bounded contexts
- ✅ Le langage ubiquitaire aligne le code sur le métier
- ✅ L'architecture hexagonale protège le domaine des détails d'infrastructure
- ❌ Plus verbeux — classes, value objects, events...
- ❌ Courbe d'apprentissage plus élevée

## Décision
**DDD avec agrégats, value objects et domain events.**

## Raisons
- La complexité du domaine botanique et social justifie un modèle riche
- Les invariants métier (une plante ne peut pas être publiée sans image) doivent être protégés au niveau objet
- Les domain events sont essentiels pour la communication entre bounded contexts
- Un projet long terme bénéficie de la maintenabilité du DDD

## Conséquences
- Chaque feature a une couche `domain/` avec entités, value objects, events, ports
- Les agrégats héritent de `AggregateRoot<DomainEvent>` et exposent des méthodes de commande
- Les value objects sont immutables et validés à la construction
- Les domain events sont collectés dans l'agrégat et dispatché après la persistance
- Les repositories sont des ports (interfaces) — les implémentations sont dans `infrastructure/`
- La couche `application/` orchestre sans logique métier : valide les inputs, appelle le domaine, persiste

## Structure type
```
features/plant/
  domain/
    Plant.ts              # Agrégat racine
    PlantPublished.ts     # Domain event
    value-objects/
      PlantName.ts
  application/
    commands/PublishPlant/
      PublishPlantCommand.ts
      PublishPlantHandler.ts
  infrastructure/
    repositories/PostgresPlantRepository.ts
  interface/
    controllers/PlantController.ts
    routes/plant.routes.ts
    schemas/plant.schema.ts
```
