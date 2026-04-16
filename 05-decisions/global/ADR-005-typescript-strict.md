# ADR-005 — TypeScript en mode strict

## Statut
Accepté

## Contexte
Le projet Folia est entièrement en TypeScript. Il faut choisir le niveau de rigueur du typage : JavaScript pur, TypeScript permissif ou TypeScript strict.

## Options considérées

### Option 1 — JavaScript pur
Pas de typage statique.
- ✅ Démarrage ultra-rapide
- ❌ Erreurs découvertes uniquement à l'exécution
- ❌ Pas d'autocomplétion fiable
- ❌ Refactoring dangereux sur un grand codebase

### Option 2 — TypeScript permissif
TypeScript avec `strict: false`.
- ✅ Migration progressive possible
- ❌ `any` implicite partout — fausse impression de sécurité
- ❌ Les erreurs les plus dangereuses ne sont pas détectées

### Option 3 — TypeScript strict ✅
TypeScript avec `strict: true` + options supplémentaires.
```json
{
  "strict": true,
  "noUncheckedIndexedAccess": true,
  "moduleDetection": "force",
  "module": "NodeNext",
  "moduleResolution": "NodeNext"
}
```
- ✅ Erreurs détectées à la compilation — pas en production
- ✅ Refactoring sûr — le compilateur guide les changements
- ✅ Autocomplétion précise dans les IDEs
- ✅ `noUncheckedIndexedAccess` — accès aux tableaux toujours vérifiés
- ❌ Plus verbeux — types explicites souvent nécessaires
- ❌ Courbe d'apprentissage plus élevée

## Décision
**TypeScript strict avec `noUncheckedIndexedAccess`.**

## Raisons
- Un projet long terme comme Folia bénéficie massivement du typage strict
- Les erreurs de type en production sont coûteuses à déboguer
- `noUncheckedIndexedAccess` évite les bugs subtils sur les accès de tableaux
- La rigueur dès le début est moins coûteuse que de l'introduire plus tard

## Conséquences
- `@repo/typescript-config/base.json` centralise la config TypeScript partagée
- Chaque package étend cette config via `"extends": "@repo/typescript-config/base.json"`
- `any` explicite est banni — utiliser `unknown` puis narrowing
- Les accès aux index de tableaux (`array[0]`) retournent `T | undefined`
- `module: NodeNext` impose les extensions `.js` dans les imports
