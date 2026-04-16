# Architecture Globale

## Vue d'ensemble

```
┌─────────────────────────────────────────┐
│              Clients                     │
│   Mobile (React Native)  Web (Astro)    │
└─────────────────┬───────────────────────┘
                  │ HTTPS
                  ▼
┌─────────────────────────────────────────┐
│               Gateway                   │
│   Auth · Routing · Rate limiting · BFF  │
└──┬──────┬──────┬──────┬──────┬──────────┘
   │      │      │      │      │
   ▼      ▼      ▼      ▼      ▼
 cat.  identity coll.  social  ...
   │      │      │      │
   └──────┴──────┴──────┘
              │
     Message Broker (V2)
    (événements asynchrones)
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
 notif.  gamif.    analytics
```

## Monorepo — Structure

```
folia/
  apps/
    front-public/          → Astro (site public, SSG/SSR)
    back-office/           → React (administration)
  packages/
    errors/                → @folia/errors — erreurs partagées
    typescript-config/     → @repo/typescript-config
    ui/                    → @repo/ui — composants partagés (futur)
  services/
    catalogue/             → service catalogue (existant)
    gateway/               → point d'entrée unique (à créer)
    identity/              → auth et profils (à créer)
    collection/            → spécimens et carnet (à créer)
    notification/          → notifications (à créer)
    ...
```

## Architecture interne d'un service

Chaque service suit la **Clean Architecture par feature** (Vertical Slice) :

```
src/
  features/
    <feature>/
      domain/              → agrégats, entités, value objects, ports
      application/         → commands, queries (CQRS)
      infrastructure/      → repositories, schéma DB, mappers
      interface/           → routes HTTP, controllers, schemas Zod
  shared/
    database/              → connexion DB
    errors/                → erreurs spécifiques au service
    events/                → AggregateRoot, DomainEvent
```

## Base de données

- Chaque service a **sa propre base de données** — pas de base partagée
- Les services ne s'accèdent jamais mutuellement via SQL
- Communication uniquement via API REST (synchrone) ou événements (asynchrone)

```
catalogue   → folia_catalogue   (PostgreSQL)
identity    → folia_identity    (PostgreSQL)
collection  → folia_collection  (PostgreSQL)
search      → Meilisearch
...
```

## Flux de données — Exemple concret : créer une plante

```
1. Client mobile
   POST /v1/plants { scientificName: "Monstera deliciosa", ... }
        │
        ▼
2. Gateway (port 3000)
   - Vérifie le JWT → appel synchrone vers identity
   - Vérifie les permissions (rôle ADMIN requis)
   - Route vers catalogue (port 3001)
        │
        ▼
3. Catalogue — Interface layer
   - PlantController.create()
   - Validation Zod du body
        │
        ▼
4. Catalogue — Application layer
   - CreatePlantHandler.execute(command)
   - Vérifie que l'espèce existe (SpeciesRepository)
   - Plant.create() → émet PlantCreated
        │
        ▼
5. Catalogue — Infrastructure layer
   - PostgresPlantRepository.save()
   - INSERT dans folia_catalogue
        │
        ▼
6. Réponse HTTP 201 { plantId: "uuid" }
        │
        ▼ (asynchrone)
7. Événement PlantCreated publié
   → search (indexation)
   → notification (si abonnements)
```

## Ports exposés (développement local)

| Service | Port |
|---|---|
| Gateway | 3000 |
| Catalogue | 3001 |
| Identity | 3002 |
| Collection | 3003 |
| Notification | 3004 |
| Identification | 3005 |
| Gamification | 3006 |
| Social | 3007 |
| Geo | 3008 |
| Search (Meilisearch) | 7700 |
| PostgreSQL | 5432 |
| Redis | 6379 |
