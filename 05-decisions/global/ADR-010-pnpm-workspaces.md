# ADR-010 — pnpm workspaces

## Statut
Accepté

## Contexte
Le monorepo Folia contient plusieurs packages et services partageant des dépendances. Il faut choisir un package manager compatible avec les workspaces.

## Options considérées

### Option 1 — npm workspaces
Package manager natif Node.js avec support workspaces depuis v7.
- ✅ Natif — aucune installation supplémentaire
- ✅ Simple à prendre en main
- ❌ `node_modules` dupliqués entre packages — consommation disque élevée
- ❌ Performances d'installation plus lentes
- ❌ Pas de vérification stricte des dépendances fantômes

### Option 2 — Yarn Berry (v2+)
Version moderne de Yarn avec Plug'n'Play.
- ✅ Performances excellentes avec PnP
- ✅ Zero-installs possible
- ❌ Plug'n'Play incompatible avec certains outils (notamment les outils natifs)
- ❌ Configuration `.yarnrc.yml` complexe
- ❌ Moins de compatibilité écosystème

### Option 3 — pnpm workspaces ✅
Package manager performant avec liens symboliques et store partagé.
- ✅ Store de packages partagé — une seule copie de chaque package sur le disque
- ✅ Liens symboliques stricts — impossible d'utiliser une dépendance non déclarée
- ✅ Performances d'installation supérieures à npm et Yarn
- ✅ `pnpm-workspace.yaml` simple et lisible
- ✅ Compatible avec Turborepo et l'écosystème moderne
- ✅ `--filter` pour cibler un package spécifique : `pnpm --filter catalogue dev`

## Décision
**pnpm workspaces.**

## Raisons
- La stricte vérification des dépendances évite les bugs de dépendances fantômes
- Les performances d'installation sont meilleures — important en CI
- Le store partagé économise l'espace disque sur les machines de développement
- Compatibilité native avec Turborepo

## Conséquences
- `pnpm-workspace.yaml` liste les workspaces : `packages/*`, `services/*`, `apps/*`
- `pnpm install` depuis la racine installe toutes les dépendances
- Les packages internes sont référencés avec `"@folia/errors": "workspace:*"`
- `pnpm --filter @folia/catalogue test` lance les tests uniquement pour le service catalogue
- Les scripts CI utilisent `pnpm` — `npm` et `yarn` ne sont pas utilisés dans le projet
