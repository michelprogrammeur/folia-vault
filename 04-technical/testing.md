# Stratégie de Tests

## Philosophie

- Tester le comportement, pas l'implémentation
- Préférer les tests d'intégration réels aux mocks pour la couche infrastructure
- Les tests unitaires couvrent le domaine et l'application
- Pas de test pour du code trivial (getters, constructeurs simples)

## Pyramide de tests

```
         /\
        /e2e\          → peu nombreux, couvrent les parcours critiques
       /──────\
      /integr. \       → couche infrastructure (repositories, DB réelle)
     /────────────\
    /  unit tests  \   → domaine + application (handlers, value objects)
   /────────────────\
```

## Tests unitaires

**Ce qu'on teste** :
- Agrégats et leur logique domaine (Plant.create, Plant.publish...)
- Value objects et leurs règles de validation (ScientificName, Care...)
- Handlers (commands et queries) avec repositories mockés
- Erreurs domaine

**Outils** : Vitest
**Localisation** : `tests/unit/`
**Commande** : `pnpm test`

**Exemple de structure** :
```
tests/unit/
  domain/
    Plant.test.ts
    ScientificName.test.ts
    Care.test.ts
  application/
    CreatePlantHandler.test.ts
    GetPlantHandler.test.ts
```

**Règles** :
- Pas de base de données, pas de réseau
- Mocks uniquement pour les ports (repositories)
- Chaque test est indépendant et peut tourner dans n'importe quel ordre
- Un `describe` par classe, un `it` par comportement testé

## Tests d'intégration

**Ce qu'on teste** :
- Repositories avec une vraie base de données
- Migrations et schéma
- Requêtes complexes avec filtres et pagination

**Outils** : Vitest + Testcontainers (PostgreSQL éphémère)
**Localisation** : `tests/integration/`
**Commande** : `pnpm test:integration`

**Règles** :
- Un container PostgreSQL par suite de tests (démarré via `setup.ts`)
- Les migrations sont appliquées automatiquement avant les tests
- Chaque test nettoie ses données ou utilise des données isolées
- Ne jamais mocker la base de données dans les tests d'intégration

## Tests e2e *(à venir)*

**Ce qu'on teste** :
- Parcours utilisateur critiques de bout en bout
- Via l'API HTTP (pas directement les handlers)

**Outils** : Vitest + supertest ou Hono test client
**Localisation** : `tests/e2e/`
**Commande** : `pnpm test:e2e`

## Couverture cible

| Couche | Cible | Priorité |
|---|---|---|
| Domain (agrégats, value objects) | 90%+ | Haute |
| Application (handlers) | 80%+ | Haute |
| Infrastructure (repositories) | via intégration | Haute |
| Interface (controllers, routes) | via e2e | Moyenne |

## Conventions de nommage des tests

```typescript
describe('Plant', () => {
  describe('create', () => {
    it('should create a plant with valid props', () => {})
    it('should throw ValidationError when scientific name is invalid', () => {})
  })

  describe('publish', () => {
    it('should emit PlantPublished event when published', () => {})
    it('should throw ConflictError when already published', () => {})
  })
})
```

## Exemple concret — Test d'intégration avec Testcontainers

```typescript
// tests/integration/setup.ts
import { PostgreSqlContainer } from '@testcontainers/postgresql'
import { drizzle } from 'drizzle-orm/postgres-js'
import { migrate } from 'drizzle-orm/postgres-js/migrator'

let db: Database

beforeAll(async () => {
  const container = await new PostgreSqlContainer('postgres:16').start()
  const client = postgres(container.getConnectionUri())
  db = drizzle(client)
  await migrate(db, { migrationsFolder: './src/features/plant/infrastructure/database/migrations' })
}, 30_000)

export const getDb = () => db

// tests/integration/PostgresPlantRepository.test.ts
describe('PostgresPlantRepository', () => {
  it('should save and retrieve a plant', async () => {
    const repo = new PostgresPlantRepository(getDb())
    const plant = makePlant('plant-1', 'Monstera deliciosa')

    await repo.save(plant)
    const found = await repo.findById(plant.id)

    expect(found).not.toBeNull()
    expect(found!.id.toString()).toBe(plant.id.toString())
  })
})
```

## Helpers de test

Centraliser les factories dans `tests/helpers/` :

```typescript
// tests/helpers/makePlant.ts
export function makePlant(id = 'plant-1', scientificName = 'Monstera deliciosa') {
  return Plant.create({ id, scientificName: ScientificName.from(scientificName), ... })
}
```
