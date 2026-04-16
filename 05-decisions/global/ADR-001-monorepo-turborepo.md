# ADR-001 — Monorepo avec Turborepo + pnpm

## Statut
Accepté

## Contexte
Folia est composé de plusieurs services back-end, applications front-end et packages partagés. Il faut choisir une stratégie de gestion du code : monorepo ou multirepo.

## Options considérées

### Option 1 — Multirepo
Un repo Git par service/application.
- ✅ Isolation totale entre les équipes
- ✅ Déploiements indépendants
- ❌ Partage de code complexe (packages publiés sur npm)
- ❌ Coordination difficile entre services liés
- ❌ Configuration dupliquée (TypeScript, ESLint, CI...)

### Option 2 — Monorepo sans outil
Un seul repo, gestion manuelle.
- ✅ Simple à démarrer
- ❌ Builds lents — tout recompile à chaque changement
- ❌ Pas de cache, pas d'optimisation

### Option 3 — Monorepo avec Turborepo + pnpm ✅
Un seul repo, orchestré par Turborepo avec pnpm workspaces.
- ✅ Cache intelligent — seuls les packages modifiés sont reconstruits
- ✅ Partage de code natif via workspaces (`@folia/errors`, `@repo/typescript-config`)
- ✅ Pipeline de build optimisé (parallélisation)
- ✅ Configuration partagée (TypeScript, ESLint)
- ✅ Un seul repo à cloner, une seule CI à configurer
- ❌ Complexité initiale de setup
- ❌ Tous les services dans le même repo — accès partagé

## Décision
**Monorepo avec Turborepo + pnpm workspaces.**

## Raisons
- Projet développé seul au départ — la coordination entre repos serait une surcharge inutile
- Les packages partagés (`@folia/errors`) justifient le monorepo
- Turborepo rend les builds rapides même avec de nombreux packages
- pnpm est plus rapide et économe que npm/yarn

## Conséquences
- Structure imposée : `apps/`, `packages/`, `services/`
- Chaque package a son propre `package.json` et `tsconfig.json`
- Les scripts sont lancés via `pnpm --filter` ou `turbo run`
- Si l'équipe grandit et nécessite une isolation forte, migration vers multirepo possible sans tout réécrire
