# Gestion des Erreurs

## Principe général

Les erreurs sont typées, hiérarchiques et portent suffisamment d'informations pour être exploitables aussi bien par le code que par les clients de l'API.

## Hiérarchie des erreurs

```
Error (JavaScript)
  └── FoliaError (base)
        ├── NotFoundError          → 404
        ├── ValidationError        → 400
        ├── ConflictError          → 409
        ├── UnauthorizedError      → 401
        └── ForbiddenError         → 403
```

Les erreurs génériques vivent dans `@folia/errors`.
Les erreurs spécifiques à un service étendent ces bases :

```typescript
// Dans services/catalogue/src/shared/errors/
export class PlantNotFoundError extends NotFoundError {
  constructor(id: string) {
    super(`Plant not found: ${id}`, 'PLANT_NOT_FOUND')
  }
}
```

## Structure d'une erreur

```typescript
class FoliaError extends Error {
  readonly code: string        // machine-readable : PLANT_NOT_FOUND
  readonly statusCode: number  // HTTP status code : 404
  readonly message: string     // human-readable : "Plant not found: xxx"
}
```

## Format de réponse d'erreur HTTP

Toutes les erreurs retournées par l'API suivent ce format JSON :

```json
{
  "error": "Plant not found: 550e8400-e29b-41d4-a716-446655440000",
  "code": "PLANT_NOT_FOUND",
  "statusCode": 404
}
```

Pour les erreurs de validation Zod :
```json
{
  "error": "Validation error",
  "details": {
    "scientificName": ["Required"],
    "care.watering": ["Invalid enum value"]
  }
}
```

## Gestion dans le controller

Le controller intercepte les erreurs connues et laisse remonter les inconnues :

```typescript
private handleError(err: unknown, reply: FastifyReply): void {
  if (err instanceof NotFoundError) {
    reply.status(err.statusCode).send({ error: err.message, code: err.code })
    return
  }
  // Erreur inconnue → Fastify global error handler → 500
  throw err
}
```

## Gestion entre services

Quand un service appelle un autre service via REST :
- Le service appelant reçoit une réponse HTTP avec le format standard
- Il convertit l'erreur en erreur de son propre domaine si nécessaire
- Il ne propage jamais l'erreur brute d'un autre service vers son client

```typescript
// Mauvais — propage l'erreur interne d'un autre service
throw externalServiceError

// Correct — traduit dans le contexte local
if (response.status === 404) {
  throw new SpeciesNotFoundError(speciesId)
}
```

## Codes d'erreur par service

### catalogue
| Code | Status | Description |
|---|---|---|
| `PLANT_NOT_FOUND` | 404 | Plante introuvable |
| `SPECIES_NOT_FOUND` | 404 | Espèce introuvable |
| `GENUS_NOT_FOUND` | 404 | Genre introuvable |
| `FAMILY_NOT_FOUND` | 404 | Famille introuvable |
| `SCIENTIFIC_NAME_CONFLICT` | 409 | Nom scientifique déjà utilisé |
| `PLANT_NOT_PUBLISHED` | 400 | Action impossible sur une plante non publiée |

### identity
| Code | Status | Description |
|---|---|---|
| `USER_NOT_FOUND` | 404 | Utilisateur introuvable |
| `EMAIL_ALREADY_EXISTS` | 409 | Email déjà utilisé |
| `PSEUDO_ALREADY_EXISTS` | 409 | Pseudo déjà utilisé |
| `INVALID_CREDENTIALS` | 401 | Email ou mot de passe incorrect |
| `TOKEN_EXPIRED` | 401 | Token JWT expiré |
| `TOKEN_INVALID` | 401 | Token JWT invalide |

### collection
| Code | Status | Description |
|---|---|---|
| `SPECIMEN_NOT_FOUND` | 404 | Spécimen introuvable |
| `COLLECTION_LIMIT_REACHED` | 403 | Limite de collection atteinte (freemium) |
| `INVALID_PLANTING_DATE` | 400 | Date de plantation dans le futur |

## Erreurs non gérées

Toute erreur non interceptée remonte jusqu'au error handler global de Fastify qui retourne un `500` :

```json
{
  "error": "Internal Server Error",
  "code": "INTERNAL_ERROR",
  "statusCode": 500
}
```

Les détails de l'erreur interne sont loggés côté serveur mais jamais exposés au client.
