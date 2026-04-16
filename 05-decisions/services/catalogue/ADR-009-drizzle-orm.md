# ADR-009 — Drizzle ORM pour l'accès à PostgreSQL

## Statut
Accepté

## Contexte
Le service catalogue utilise PostgreSQL. Il faut choisir comment interagir avec la base de données depuis TypeScript.

## Options considérées

### Option 1 — TypeORM
ORM historique TypeScript, style ActiveRecord ou DataMapper.
- ✅ Très répandu, documentation abondante
- ✅ Migrations intégrées
- ❌ Decorators expérimentaux requis (`reflect-metadata`)
- ❌ Types imprécis — les queries retournent souvent `any`
- ❌ Performances moindres — beaucoup de magie cachée
- ❌ Bugs connus avec les relations complexes

### Option 2 — Prisma
ORM moderne avec génération de client TypeScript.
- ✅ DX excellente — autocomplétion parfaite
- ✅ Migrations robustes (`prisma migrate`)
- ✅ Très populaire
- ❌ Client généré — overhead au runtime
- ❌ Schéma dans un fichier `.prisma` séparé — duplication avec les types TypeScript
- ❌ Moins de contrôle sur le SQL généré
- ❌ Requêtes complexes difficiles à exprimer

### Option 3 — Drizzle ORM ✅
ORM léger TypeScript-first avec SQL typé.
- ✅ TypeScript-first — inférence de type maximale sans génération de code
- ✅ SQL-like — le SQL généré est prévisible et lisible
- ✅ Schéma défini en TypeScript — source unique de vérité
- ✅ Migrations avec `drizzle-kit`
- ✅ Performances proches du SQL brut — pas de magie cachée
- ✅ Compatible avec `pg` (node-postgres)
- ❌ Écosystème plus jeune que Prisma
- ❌ Moins de ressources en ligne

### Option 4 — SQL brut avec `pg`
Requêtes SQL écrites manuellement.
- ✅ Contrôle total
- ✅ Performances maximales
- ❌ Pas de typage des résultats
- ❌ Migrations manuelles
- ❌ Verbosité importante

## Décision
**Drizzle ORM.**

## Raisons
- L'inférence TypeScript sans génération de code est supérieure à Prisma
- Le SQL prévisible facilite l'optimisation des requêtes
- La légèreté correspond à nos besoins — pas besoin de la puissance de Prisma
- Le schéma TypeScript est la source de vérité pour les types et les migrations

## Conséquences
- Le schéma Drizzle est dans `src/infrastructure/database/schema.ts`
- Les migrations sont générées avec `drizzle-kit generate` et appliquées avec `drizzle-kit migrate`
- Les repositories utilisent `db.select().from(plants).where(...)` — style SQL-like
- Les types de lignes Drizzle (`typeof plants.$inferSelect`) sont utilisés dans les mappers
- `drizzle-kit` est configuré dans `drizzle.config.ts` à la racine du service
