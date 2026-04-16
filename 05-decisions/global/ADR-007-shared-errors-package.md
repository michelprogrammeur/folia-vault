# ADR-007 — Package d'erreurs partagé (@folia/errors)

## Statut
Accepté

## Contexte
Les services Folia ont besoin de classes d'erreur métier typées (NotFoundError, ConflictError, ValidationError...). Il faut décider où et comment ces erreurs sont définies.

## Options considérées

### Option 1 — Erreurs définies dans chaque service
Chaque service définit ses propres classes d'erreur localement.
- ✅ Autonomie totale de chaque service
- ❌ Duplication massive — chaque service réimplémente les mêmes erreurs
- ❌ Inconsistance entre services — même erreur, noms différents
- ❌ Impossible de partager un handler d'erreurs HTTP entre services

### Option 2 — Erreurs dans un package utilitaire générique
Un package `@folia/utils` contenant erreurs, helpers, types communs.
- ✅ Centralisation
- ❌ Package fourre-tout — responsabilités floues
- ❌ Couplage fort — un changement d'un helper peut impacter tous les services

### Option 3 — Package dédié @folia/errors ✅
Un package `packages/errors` exposé sous `@folia/errors`, exclusivement dédié aux erreurs métier.
- ✅ Responsabilité unique — uniquement les erreurs
- ✅ Partagé entre tous les services via pnpm workspaces
- ✅ Typage fort — `instanceof NotFoundError` fonctionne partout
- ✅ Facilite la gestion centralisée des erreurs HTTP dans les controllers
- ✅ Versioning indépendant possible

## Décision
**Package dédié `@folia/errors`.**

## Raisons
- La responsabilité unique rend le package stable et prévisible
- Le check `instanceof` est essentiel pour mapper les erreurs métier en réponses HTTP
- La duplication entre services est inacceptable sur un monorepo

## Conséquences
- `packages/errors/src/index.ts` exporte : `NotFoundError`, `ConflictError`, `ValidationError`, `ForbiddenError`, `UnauthorizedError`
- Toutes les classes héritent de `Error` avec un `code` machine-readable et un `statusCode` HTTP
- Les controllers importent `@folia/errors` pour le mapping erreur → HTTP
- Les use cases lancent des erreurs `@folia/errors` — jamais des strings ou Error génériques
- Chaque service déclare `@folia/errors` dans ses `dependencies`

## Structure
```typescript
export class NotFoundError extends Error {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;
  constructor(message: string) { super(message); this.name = 'NotFoundError'; }
}
```
