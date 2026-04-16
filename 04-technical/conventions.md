# Conventions de Code

## Langue

- **Code** : anglais (noms de variables, fonctions, classes, fichiers)
- **Commentaires** : anglais
- **Commits** : anglais
- **Vault / documentation** : français

## Nommage

### Fichiers et dossiers
```
PascalCase    → classes, composants, agrégats       Plant.ts, PlantController.ts
camelCase     → fonctions, variables                createPlant.ts
kebab-case    → dossiers de features                plant-catalogue/
SCREAMING     → constantes globales                 MAX_COLLECTION_SIZE
```

### Classes et interfaces
```typescript
class PlantController {}          // PascalCase
interface PlantRepositoryPort {}  // Port suffix — pas de préfixe I
class CreatePlantHandler {}       // Handler suffix pour les use cases
class CreatePlantCommand {}       // Command suffix
class GetPlantQuery {}            // Query suffix
class PlantId {}                  // Value objects — PascalCase
class PlantCreated {}             // Domain events — passé
```

### Méthodes
```typescript
// Commands handlers
async execute(command: CreatePlantCommand): Promise<CreatePlantResult>

// Query handlers
async execute(query: GetPlantQuery): Promise<GetPlantResult>

// Repository ports
async save(plant: Plant): Promise<void>
async findById(id: PlantId): Promise<Plant | null>
async findAll(options: FindPlantsOptions): Promise<PaginatedResult<Plant>>
async delete(id: PlantId): Promise<void>
async exists(id: PlantId): Promise<boolean>
```

## Architecture par feature

### Règles strictes
- Tout ce qui concerne une feature est dans son dossier — pas de dossier `utils/` global
- Les features ne s'importent jamais entre elles directement
- Si deux features partagent du code → extraire dans `shared/` ou `packages/`
- Un agrégat = un repository port = une implémentation repository

### Structure obligatoire d'un handler
```typescript
export class CreatePlantHandler {
  constructor(
    private readonly plantRepository: PlantRepositoryPort,  // injection
    private readonly speciesRepository: SpeciesRepositoryPort,
  ) {}

  async execute(command: CreatePlantCommand): Promise<CreatePlantResult> {
    // 1. Valider les préconditions métier
    // 2. Récupérer les agrégats nécessaires
    // 3. Appeler la méthode domaine
    // 4. Persister
    // 5. Retourner le résultat
  }
}
```

## CQRS

- **Commands** : mutent l'état, retournent uniquement l'ID ou void
- **Queries** : lisent l'état, ne mutent jamais rien
- Chaque command/query dans son propre dossier avec son fichier + handler
- Les queries peuvent contourner le domaine et accéder directement à la DB si besoin (read model)

## Erreurs

- Erreurs génériques dans `@folia/errors` : `NotFoundError`, `ValidationError`, `ConflictError`
- Erreurs spécifiques dans `src/shared/errors/` du service : `PlantNotFoundError extends NotFoundError`
- Chaque erreur a un `code` (machine-readable) et un `statusCode` (HTTP)
- On ne throw jamais d'erreurs génériques JavaScript dans le domaine

## Tests

- Tests unitaires dans `tests/unit/` — pas de dépendances externes
- Tests d'intégration dans `tests/integration/` — avec Testcontainers
- Nommage : `<SujetTest>.test.ts`
- Un fichier de test par classe testée
- `pnpm test` → unitaires uniquement
- `pnpm test:integration` → intégration uniquement
- `pnpm test:all` → tout

## Imports

- Toujours utiliser les imports avec extension `.js` (module ESM NodeNext)
- Pas d'imports circulaires
- Imports ordonnés : externes → internes → relatifs

```typescript
// Correct
import { eq } from 'drizzle-orm'
import { PlantRepositoryPort } from '../../domain/repositories/PlantRepositoryPort.js'
import { PlantNotFoundError } from '../../../../shared/errors/index.js'
```

## Git

- Branches : `feat/<ticket>-<description>`, `fix/<ticket>-<description>`
- Commits : Conventional Commits — `feat(catalogue): add plant publishing`
- Types : `feat`, `fix`, `refactor`, `test`, `chore`, `docs`
- Scope : nom du service ou package (`catalogue`, `errors`, `gateway`...)
- Un commit = une intention claire
