# ADR-006 — Vitest plutôt que Jest

## Statut
Accepté

## Contexte
Le projet Folia utilise TypeScript et Turborepo. Il faut choisir un framework de test unitaire.

## Options considérées

### Option 1 — Jest
Framework de test le plus utilisé dans l'écosystème JavaScript.
- ✅ Très répandu, documentation abondante
- ✅ Large écosystème de plugins et matchers
- ❌ Configuration ESM complexe avec TypeScript — nécessite `ts-jest` ou `babel-jest`
- ❌ Performances plus lentes sur les gros projets
- ❌ Incompatible nativement avec les modules ESM (`"type": "module"`)
- ❌ Watch mode moins performant

### Option 2 — Vitest ✅
Framework de test moderne basé sur Vite.
- ✅ Support natif de TypeScript et ESM — aucune configuration supplémentaire
- ✅ Syntaxe compatible Jest — migration quasi transparente
- ✅ Performances supérieures grâce au HMR de Vite
- ✅ Watch mode ultra-rapide
- ✅ Interface UI intégrée (`vitest --ui`)
- ✅ Coverage avec `@vitest/coverage-v8` — sans configuration
- ✅ Cohérent avec l'outillage moderne (Vite, Turborepo)
- ❌ Écosystème légèrement plus jeune que Jest

## Décision
**Vitest.**

## Raisons
- Le support natif ESM évite toute la complexité de configuration avec `ts-jest`
- Les performances en watch mode sont significativement meilleures
- La syntaxe identique à Jest (`describe`, `it`, `expect`, `vi.fn()`) limite la courbe d'apprentissage
- Cohérence avec l'outillage moderne du projet

## Conséquences
- `vitest` est installé dans chaque package qui a des tests
- La configuration est dans `vitest.config.ts` à la racine du package
- Les tests unitaires sont dans `src/**/*.test.ts` (co-localisés avec le code)
- Les tests d'intégration sont dans `tests/integration/**/*.test.ts`
- `vi.fn()` remplace `jest.fn()`, `vi.mock()` remplace `jest.mock()`
- Coverage disponible via `pnpm test:coverage` avec `@vitest/coverage-v8`
