# Service — Catalogue

## Rôle

Référentiel botanique de toutes les espèces végétales. Source de vérité pour tout ce qui concerne la taxonomie et les données d'une plante.

## Stack

| Outil | Version | Rôle |
|---|---|---|
| Node.js | 18+ | Runtime |
| TypeScript | 5.x | Langage |
| Fastify | 5.x | Framework HTTP |
| Zod | 3.x | Validation des entrées |
| Drizzle ORM | 0.x | ORM PostgreSQL |
| PostgreSQL | 16 | Base de données |
| Vitest | 2.x | Tests unitaires et intégration |
| Testcontainers | — | PostgreSQL éphémère pour les tests |

## Port local

```
3001
```

## Base de données

```
folia_catalogue (PostgreSQL)
```

### Tables

| Table | Description |
|---|---|
| `families` | Familles botaniques |
| `genera` | Genres botaniques |
| `species` | Espèces botaniques |
| `plants` | Fiches plantes complètes |

### Enums PostgreSQL

| Enum | Valeurs |
|---|---|
| `plant_type` | PLANT, SHRUB, TREE, FLOWER, VEGETABLE, FRUIT, SUCCULENT, CACTUS, AQUATIC, EPIPHYTE, GRASS, FERN, PALM |
| `growth_rate` | SLOW, MEDIUM, FAST |
| `difficulty` | EASY, MEDIUM, HARD |
| `watering_level` | LOW, MEDIUM, HIGH |
| `light_level` | SHADE, INDIRECT, BRIGHT_INDIRECT, FULL_SUN |
| `humidity_level` | LOW, MEDIUM, HIGH |
| `leaf_shape` | OVAL, LANCEOLATE, HEART, ROUND, LINEAR, LOBED, PINNATE, PALMATE |
| `leaf_color` | GREEN, DARK_GREEN, VARIEGATED, PURPLE, SILVER, GOLDEN |
| `leaf_texture` | SMOOTH, WAXY, FUZZY, ROUGH, GLOSSY |
| `leaf_size` | TINY, SMALL, MEDIUM, LARGE, EXTRA_LARGE |
| `flower_color` | WHITE, YELLOW, RED, PINK, PURPLE, ORANGE, BLUE, NONE |
| `flowering_period` | SPRING, SUMMER, AUTUMN, WINTER, YEAR_ROUND |

## Structure interne

```
src/
  features/
    plant/
      domain/
        aggregates/        → Plant.ts
        entities/          → Family.ts, Genus.ts, Species.ts
        value-objects/     → PlantId, ScientificName, Care, Characteristics, Appearance
        repositories/      → PlantRepositoryPort, SpeciesRepositoryPort...
        events/            → PlantCreated, PlantPublished, PlantUnpublished, PlantDeleted
      application/
        commands/
          CreatePlant/     → CreatePlantCommand, CreatePlantHandler
          UpdatePlant/     → UpdatePlantCommand, UpdatePlantHandler
          DeletePlant/     → DeletePlantCommand, DeletePlantHandler
          PublishPlant/    → PublishPlantCommand, PublishPlantHandler
          UnpublishPlant/  → UnpublishPlantCommand, UnpublishPlantHandler
        queries/
          GetPlant/        → GetPlantQuery, GetPlantHandler, GetPlantResult
          ListPlants/      → ListPlantsQuery, ListPlantsHandler, ListPlantsResult
          PlantMapper.ts
      infrastructure/
        database/
          schema.ts        → schéma Drizzle
          migrations/      → fichiers SQL générés
        mappers/
          PlantPersistenceMapper.ts
        repositories/
          PostgresPlantRepository.ts
          PostgresSpeciesRepository.ts
      interface/
        controllers/       → PlantController.ts
        routes/            → plant.routes.ts
        schemas/           → plant.schema.ts (Zod)
  shared/
    database/              → connection.ts
    errors/                → PlantNotFoundError, SpeciesNotFoundError...
    events/                → AggregateRoot.ts, DomainEvent.ts
```

## Endpoints HTTP

| Méthode | Path | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST` | `/plants` | Créer une plante |
| `GET` | `/plants` | Lister les plantes (paginé, filtrable) |
| `GET` | `/plants/:id` | Récupérer une plante |
| `PATCH` | `/plants/:id` | Mettre à jour une plante |
| `DELETE` | `/plants/:id` | Supprimer une plante |
| `POST` | `/plants/:id/publish` | Publier une plante |
| `POST` | `/plants/:id/unpublish` | Dépublier une plante |

## Filtres disponibles sur `GET /plants`

| Paramètre | Type | Description |
|---|---|---|
| `page` | number | Page (défaut: 1) |
| `limit` | number | Taille de page (défaut: 20, max: 100) |
| `type` | PlantType | Filtrer par type |
| `difficulty` | Difficulty | Filtrer par difficulté |
| `isPublished` | boolean | Filtrer par statut de publication |
| `tags` | string | Tags séparés par virgule |

## Dépendances inter-services

| Service | Type | Usage |
|---|---|---|
| `identity` | Synchrone | Vérification des permissions (via gateway) |
| `search` | Asynchrone | Indexation des fiches publiées |
| `notification` | Asynchrone | Notifications lors de publications |

## Événements émis

| Événement | Déclencheur |
|---|---|
| `PlantCreated` | POST /plants |
| `PlantPublished` | POST /plants/:id/publish |
| `PlantUnpublished` | POST /plants/:id/unpublish |
| `PlantUpdated` | PATCH /plants/:id |
| `PlantDeleted` | DELETE /plants/:id |

## Variables d'environnement

```env
NODE_ENV=development
PORT=3001
HOST=0.0.0.0
DATABASE_URL=postgresql://folia:folia@localhost:5432/folia_catalogue
ALLOWED_ORIGINS=http://localhost:3000
LOG_LEVEL=debug
```

## Commandes utiles

```bash
# Développement
pnpm --filter @folia/catalogue dev

# Build
pnpm --filter @folia/catalogue build

# Tests unitaires
pnpm --filter @folia/catalogue test

# Tests intégration
pnpm --filter @folia/catalogue test:integration

# Générer une migration
pnpm --filter @folia/catalogue db:generate

# Appliquer les migrations
pnpm --filter @folia/catalogue db:migrate

# Drizzle Studio
pnpm db:studio

# Démarrer l'infra (PostgreSQL)
pnpm infra:catalogue
```

## État actuel

- ✅ Domain (Plant, Family, Genus, Species, value objects, events)
- ✅ Application (CreatePlant, UpdatePlant, DeletePlant, PublishPlant, UnpublishPlant, GetPlant, ListPlants)
- ✅ Infrastructure (PostgresPlantRepository, PostgresSpeciesRepository, migrations)
- ✅ Interface (PlantController, plant.routes, Zod schemas)
- ✅ Tests unitaires (102 tests)
- ✅ Tests intégration (PostgresPlantRepository — 9 tests)
- ⬜ Événements domaine publiés vers un message broker
- ⬜ SearchPlants (Meilisearch)
- ⬜ Family/Genus/Species endpoints HTTP
