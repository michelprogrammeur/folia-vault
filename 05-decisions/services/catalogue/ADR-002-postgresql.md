# ADR-002 — PostgreSQL comme base de données

## Statut
Accepté

## Contexte
Le service catalogue stocke des données relationnelles complexes (taxonomie hiérarchique, fiches plantes avec de nombreux champs). Il faut choisir un système de base de données.

## Options considérées

### Option 1 — MySQL
Base relationnelle très répandue, performante.
- ✅ Très répandu, hébergement facile
- ✅ Performances solides
- ❌ Support JSON moins avancé que PostgreSQL
- ❌ Moins de types avancés (arrays, enums natifs...)
- ❌ Extensions moins riches

### Option 2 — MongoDB
Base NoSQL orientée documents.
- ✅ Flexibilité du schéma
- ✅ Facile pour les données imbriquées
- ❌ Pas de relations natives — JOINs complexes
- ❌ Cohérence transactionnelle moins robuste
- ❌ La taxonomie hiérarchique est naturellement relationnelle

### Option 3 — PostgreSQL ✅
Base relationnelle avancée avec des fonctionnalités proches du NoSQL.
- ✅ Enums natifs — parfait pour nos nombreux types (PlantType, GrowthRate...)
- ✅ Arrays natifs — stockage des tags, commonNames, origin
- ✅ JSONB pour les données semi-structurées si nécessaire
- ✅ Extensions puissantes (uuid-ossp, pg_trgm pour la recherche)
- ✅ Transactions ACID robustes
- ✅ Performances excellentes
- ✅ Hébergement facile (Supabase, Neon, Railway...)

## Décision
**PostgreSQL 16.**

## Raisons
- Les données botaniques sont naturellement relationnelles (famille → genre → espèce → plante)
- Les enums natifs PostgreSQL sont idéaux pour nos nombreux types énumérés
- Les arrays natifs évitent des tables de jointure pour les tags et listes simples
- PostgreSQL est le standard de l'industrie pour les applications sérieuses

## Conséquences
- Chaque service a sa propre base PostgreSQL
- Les enums sont définis au niveau PostgreSQL — pas uniquement en TypeScript
- `pg_trgm` est activé pour la recherche textuelle sur les noms scientifiques
- UUID v4 via `gen_random_uuid()` pour tous les identifiants
