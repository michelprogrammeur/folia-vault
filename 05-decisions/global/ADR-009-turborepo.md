# ADR-009 — Turborepo pour le monorepo

## Statut
Accepté

## Contexte
Folia est organisé en monorepo avec plusieurs services, packages partagés et applications. Il faut choisir un outil d'orchestration des builds et des tâches.

## Options considérées

### Option 1 — Lerna
L'outil historique de gestion de monorepos JavaScript.
- ✅ Très répandu, documentation abondante
- ✅ Gestion du versioning et des publications npm
- ❌ Maintenance réduite depuis le rachat par Nx
- ❌ Pas de cache de build natif performant
- ❌ Configuration complexe

### Option 2 — Nx
Framework complet pour monorepos — build, test, lint, génération de code.
- ✅ Très puissant — cache distribué, affected commands, générateurs
- ✅ Support natif TypeScript, React, Node
- ❌ Opinionated — impose une structure de projet
- ❌ Courbe d'apprentissage élevée
- ❌ Surcharge pour un projet qui ne nécessite pas tous ses outils

### Option 3 — Turborepo ✅
Orchestrateur de tâches pour monorepos — léger et performant.
- ✅ Léger — ne dicte pas la structure du projet
- ✅ Cache local et distant (Vercel Remote Cache) — builds ultra-rapides
- ✅ Pipeline de tâches déclaratif (`turbo.json`)
- ✅ Compatible avec n'importe quel package manager
- ✅ Parallélisation automatique des tâches indépendantes
- ✅ Intégration native avec Vercel pour le déploiement
- ❌ Moins de fonctionnalités que Nx (pas de générateurs de code)

## Décision
**Turborepo.**

## Raisons
- La légèreté et la non-opinionatedness correspondent à notre besoin
- Le cache de build est la fonctionnalité clé — les builds répétés sont quasi instantanés
- La parallélisation des tâches accélère significativement le CI
- Pas besoin des générateurs de code de Nx à ce stade

## Conséquences
- `turbo.json` à la racine définit le pipeline (`build`, `test`, `lint`, `typecheck`)
- `turbo run build` reconstruit uniquement les packages dont les sources ont changé
- Les dépendances entre packages sont respectées automatiquement (`catalogue` attend que `@folia/errors` soit buildé)
- Le cache Vercel Remote Cache peut être activé pour partager le cache entre développeurs et CI
- `turbo run dev --filter=catalogue` démarre uniquement le service catalogue et ses dépendances
