# ADR-011 — Architecture Vertical Slice (Feature-based)

## Statut
Accepté

## Contexte
Chaque service Folia doit être organisé en couches. Il faut choisir entre une organisation horizontale (par couche technique) ou verticale (par feature métier).

## Options considérées

### Option 1 — Architecture en couches horizontales
Organisation par couche technique : `controllers/`, `services/`, `repositories/`.
- ✅ Simple à comprendre pour les nouveaux arrivants
- ✅ Convention très répandue (NestJS, Spring...)
- ❌ Une feature est dispersée dans toutes les couches — navigation difficile
- ❌ Couplage croissant entre features au sein d'une même couche
- ❌ Difficile de extraire une feature dans un autre service

### Option 2 — Vertical Slice Architecture (Feature-based) ✅
Organisation par feature métier : `features/plant/`, `features/species/`. Chaque feature contient ses propres couches.
- ✅ Une feature = un dossier — navigation intuitive
- ✅ Isolation forte entre features — pas de couplage accidentel
- ✅ Extraction d'une feature vers un autre service facilitée
- ✅ Cohérent avec le DDD — une feature correspond à un use case ou un agrégat
- ✅ Les équipes peuvent travailler sur des features sans conflit
- ❌ Plus de fichiers et de dossiers
- ❌ Duplication possible de certains patterns entre features

## Décision
**Vertical Slice Architecture avec DDD par feature.**

## Raisons
- La navigation par feature est plus naturelle que par couche technique
- L'isolation entre features réduit le couplage involontaire
- Cohérence avec le DDD et les bounded contexts
- Facilite l'évolution vers des microservices plus granulaires si nécessaire

## Conséquences
- Chaque feature suit la structure : `domain/`, `application/`, `infrastructure/`, `interface/`
- Les features ne s'importent pas mutuellement — communication via domain events ou ports
- Les éléments vraiment partagés entre features vont dans `shared/` (value objects communs, types...)
- La structure type pour une feature :
```
features/plant/
  domain/
    Plant.ts
    PlantRepositoryPort.ts
    value-objects/
    events/
  application/
    commands/
    queries/
  infrastructure/
    repositories/
  interface/
    controllers/
    routes/
    schemas/
```
- Un nouveau développeur trouve le code d'une feature en cherchant `features/<nom>/`
