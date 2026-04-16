# ADR-003 — Testcontainers pour les tests d'intégration

## Statut
Accepté

## Contexte
Les repositories PostgreSQL doivent être testés contre une vraie base de données. Il faut choisir la stratégie pour les tests d'intégration.

## Options considérées

### Option 1 — Mocks des repositories
Remplacer PostgreSQL par des implementations en mémoire dans les tests.
- ✅ Rapide à exécuter
- ✅ Pas de dépendance externe
- ❌ Les mocks ne reproduisent pas le comportement réel de PostgreSQL
- ❌ On a été brûlés par cette approche — les mocks passaient mais la prod cassait
- ❌ Ne teste pas les migrations, les contraintes, les index...

### Option 2 — Base de données partagée (CI)
Une base PostgreSQL fixe dans le pipeline CI.
- ✅ Proche de la production
- ❌ Tests inter-dépendants — les données d'un test polluent les autres
- ❌ Difficile à reproduire en local
- ❌ Nettoyage des données complexe

### Option 3 — Testcontainers ✅
Un container PostgreSQL éphémère démarré pour chaque suite de tests.
- ✅ Base de données réelle — comportement identique à la production
- ✅ Isolation totale — container détruit après les tests
- ✅ Fonctionne en local et en CI (Docker requis)
- ✅ Migrations appliquées automatiquement avant les tests
- ❌ Démarrage plus lent (5-10 secondes)
- ❌ Nécessite Docker

## Décision
**Testcontainers avec PostgreSQL éphémère.**

## Raisons
- Un mock ne remplace pas une vraie base de données pour tester un repository
- L'isolation garantit que les tests sont indépendants et reproductibles
- Le surcoût de temps (5-10s) est acceptable comparé à la fiabilité gagnée

## Conséquences
- Docker est requis pour lancer les tests d'intégration
- Les tests d'intégration sont séparés des tests unitaires (`pnpm test:integration`)
- Le setup Testcontainers est centralisé dans `tests/integration/setup.ts`
- Les migrations sont appliquées automatiquement au démarrage du container
- Timeout `beforeAll` à 30 secondes pour laisser le temps au container de démarrer
