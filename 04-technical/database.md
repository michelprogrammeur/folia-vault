# Base de données — Conventions

## Principes généraux

- Chaque service a sa **propre base de données** — pas de base partagée
- Les services ne s'accèdent jamais mutuellement via SQL
- PostgreSQL 16 est la base de données standard de tous les services back-end
- Drizzle ORM est l'unique ORM utilisé — pas de requêtes SQL brutes sauf cas exceptionnels documentés

## Nommage

### Tables
- **snake_case**, **pluriel**
- Préfixe si plusieurs domaines dans la même base (peu recommandé)

```sql
-- Correct
plants
species
journal_entries
plant_care_reminders

-- Incorrect
Plant, plant, tblPlant, Plant_Table
```

### Colonnes
- **snake_case**, **singulier**
- Booléens préfixés par `is_` ou `has_`
- Clés étrangères suffixées par `_id`

```sql
-- Correct
scientific_name
is_published
is_pet_friendly
species_id
created_at

-- Incorrect
scientificName, IsPublished, speciesID
```

### Enums PostgreSQL
- **snake_case**, **singulier**
- Valeurs en **SCREAMING_SNAKE_CASE**

```sql
CREATE TYPE plant_type AS ENUM ('PLANT', 'SHRUB', 'TREE');
CREATE TYPE growth_rate AS ENUM ('SLOW', 'MEDIUM', 'FAST');
```

### Index
- Format : `idx_<table>_<colonne(s)>`

```sql
idx_plants_type
idx_plants_species_id
idx_plants_is_published_created_at
```

## Colonnes obligatoires sur toutes les tables

```typescript
// Toutes les tables doivent avoir ces colonnes
id         uuid        PRIMARY KEY DEFAULT gen_random_uuid()
created_at timestamp   NOT NULL DEFAULT now()
updated_at timestamp   NOT NULL DEFAULT now()
```

Pour les tables avec soft delete :
```typescript
deleted_at timestamp   NULL  -- NULL = actif, date = supprimé
```

## Identifiants

- Toujours des **UUID v4** — jamais d'auto-increment entier
- Générés par PostgreSQL : `DEFAULT gen_random_uuid()`
- Jamais exposés sous forme d'entier séquentiel en dehors de la DB

## Migrations

### Règles strictes
- Une migration = un changement logique atomique
- Les migrations sont **irréversibles** en production — bien réfléchir avant
- Toujours tester une migration en local avant de commiter
- Jamais de `DROP COLUMN` sans avoir d'abord retiré toute référence dans le code

### Nommage des fichiers Drizzle
```
0000_<description>.sql
0001_add_plants_tags.sql
0002_add_species_status.sql
```

### Migrations dangereuses
Ces opérations nécessitent une attention particulière en production :

| Opération | Risque | Solution |
|---|---|---|
| `ADD COLUMN NOT NULL` | Bloque si table peuplée | Ajouter nullable puis contraindre |
| `DROP COLUMN` | Irréversible | Soft delete d'abord, drop après déploiement |
| `RENAME COLUMN` | Casse les requêtes existantes | Ajouter nouvelle colonne, migrer, supprimer |
| `ALTER TYPE enum` | Peut nécessiter un lock | Créer nouveau type, migrer, supprimer |

## Index

### Index obligatoires (créés automatiquement)
- Clé primaire (`id`)
- Clés étrangères (`species_id`, `user_id`...)

### Index à créer manuellement
- Colonnes filtrées fréquemment (`type`, `difficulty`, `is_published`)
- Colonnes de tri (`created_at`, `updated_at`)
- Recherche textuelle : `pg_trgm` pour les colonnes texte

```typescript
// Exemple dans le schéma Drizzle
export const plantsTable = pgTable('plants', {
  // ...colonnes
}, (table) => ({
  typeIdx: index('idx_plants_type').on(table.type),
  publishedIdx: index('idx_plants_is_published').on(table.isPublished),
  scientificNameIdx: index('idx_plants_scientific_name').on(table.scientificName),
}))
```

## Soft Delete

Pattern standard pour les tables avec soft delete :

```typescript
// Requête standard — exclure les supprimés
where(isNull(table.deletedAt))

// Soft delete
update(table).set({ deletedAt: new Date() }).where(eq(table.id, id))
```

## Transactions

Utiliser les transactions pour les opérations multi-tables :

```typescript
await db.transaction(async (tx) => {
  await tx.insert(plantsTable).values(plantData)
  await tx.insert(plantTagsTable).values(tagsData)
})
```

## Connexion

```typescript
// shared/database/connection.ts
import postgres from 'postgres'
import { drizzle } from 'drizzle-orm/postgres-js'

const client = postgres(process.env.DATABASE_URL ?? 'postgresql://folia:folia@localhost:5432/folia_catalogue')
export const db = drizzle(client)
export type Database = typeof db
```

- Pool de connexions géré par `postgres.js`
- `DATABASE_URL` obligatoire en production
- Fermeture propre du pool au shutdown du service
