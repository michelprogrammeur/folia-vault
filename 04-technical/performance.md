# Performance

## Principes

- Optimiser sur mesure uniquement ce qui est mesuré — pas de spéculation
- Mettre en cache ce qui est lu souvent et change rarement
- Paginer toutes les listes sans exception
- Les requêtes lentes sont loggées et tracées

## Pagination

Toutes les listes sont paginées — jamais de retour de liste complète non bornée.

```typescript
// Format standard de pagination
interface PaginatedResult<T> {
  data: T[]
  total: number
  page: number
  limit: number
  totalPages: number
}

// Limites
const DEFAULT_PAGE_SIZE = 20
const MAX_PAGE_SIZE = 100
```

## Indexation base de données

### Index obligatoires
- Toutes les clés étrangères
- Les colonnes utilisées dans les filtres fréquents (`type`, `difficulty`, `isPublished`)
- Les colonnes de recherche textuelle (`scientific_name` avec `pg_trgm`)
- `created_at` et `updated_at` pour les tris chronologiques

### Index à éviter
- Colonnes à faible cardinalité (booléens seuls)
- Tables petites (< 10 000 lignes) — le scan est souvent plus rapide

## Cache *(V1 étendu)*

### Redis
- Cache des fiches plantes (TTL : 1 heure) — données qui changent rarement
- Cache des sessions utilisateurs
- Rate limiting distribué

### Cache HTTP
- `Cache-Control` sur les endpoints publics du catalogue
- `ETag` pour la validation conditionnelle
- CDN pour les assets statiques et médias

## Recherche

Meilisearch est utilisé pour toute recherche full-text :
- Index mis à jour via événements domaine (asynchrone)
- Jamais de `LIKE %terme%` en PostgreSQL pour la recherche full-text
- `pg_trgm` uniquement pour les recherches exactes ou préfixes courts

## Médias

- Images redimensionnées en plusieurs variantes à l'upload (thumbnail, mobile, web)
- Format WebP privilégié pour le web
- Lazy loading sur toutes les images
- CDN obligatoire en production pour la distribution

## Monitoring des performances *(à venir)*

- Logs des requêtes SQL > 100ms
- Alertes sur les endpoints > 500ms (P95)
- Dashboard Grafana avec les métriques clés
- Profiling en staging avant chaque release majeure
