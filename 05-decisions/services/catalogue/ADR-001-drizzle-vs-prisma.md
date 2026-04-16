# ADR-001 — Drizzle ORM vs Prisma

## Statut
Accepté

## Contexte
Le service catalogue a besoin d'un ORM pour interagir avec PostgreSQL. Les deux principaux choix dans l'écosystème TypeScript sont Drizzle et Prisma.

## Options considérées

### Option 1 — Prisma
ORM mature avec un DSL propre (`schema.prisma`) et une grande communauté.
- ✅ Documentation excellente, très répandu
- ✅ Prisma Studio pour explorer les données
- ✅ Migrations automatiques
- ❌ Couche d'abstraction opaque — le SQL généré est difficile à contrôler
- ❌ Client Prisma lourd avec un binaire natif
- ❌ Types générés depuis le schéma — pas de type-safety native TypeScript
- ❌ Performances moindres sur les requêtes complexes
- ❌ Prisma Client doit être re-généré après chaque changement de schéma

### Option 2 — Drizzle ORM ✅
ORM léger, type-safe natif TypeScript, SQL-first.
- ✅ Type-safety totale sans génération de code externe
- ✅ SQL transparent — on sait exactement ce qui est exécuté
- ✅ Très léger — pas de binaire natif
- ✅ Performances excellentes — proche du SQL brut
- ✅ Drizzle Studio disponible
- ✅ Schéma défini en TypeScript — cohérence avec le reste du projet
- ❌ Moins mature que Prisma
- ❌ Communauté plus petite
- ❌ API parfois moins intuitive pour les requêtes complexes

## Décision
**Drizzle ORM.**

## Raisons
- La type-safety native est critique pour un projet TypeScript strict
- Le contrôle total du SQL est important pour les performances
- L'absence de binaire natif simplifie les builds et le déploiement
- La cohérence TypeScript du schéma facilite la maintenance

## Conséquences
- Le schéma est défini en TypeScript dans `infrastructure/database/schema.ts`
- Les migrations sont générées par Drizzle Kit : `pnpm db:generate`
- Les requêtes complexes utilisent `sql` template literal quand Drizzle ne suffit pas
- Les types des colonnes sont inférés automatiquement depuis le schéma
