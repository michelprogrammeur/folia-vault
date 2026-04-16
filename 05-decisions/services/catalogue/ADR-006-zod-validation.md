# ADR-006 — Zod pour la validation des entrées HTTP

## Statut
Accepté

## Contexte
Les routes Fastify reçoivent des données externes (body, params, query). Il faut valider et typer ces entrées. Fastify propose nativement JSON Schema pour la validation.

## Options considérées

### Option 1 — JSON Schema natif de Fastify
Fastify peut valider automatiquement les requêtes via JSON Schema.
- ✅ Intégration native — validation automatique avant le handler
- ✅ Performances maximales (AJV sous le capot)
- ❌ JSON Schema verbeux — difficile à écrire et maintenir
- ❌ Pas de typage TypeScript inféré — types à déclarer manuellement
- ❌ Messages d'erreur peu lisibles sans configuration avancée

### Option 2 — class-validator + class-transformer
Style NestJS avec des classes décorées.
- ✅ Familier pour les développeurs NestJS
- ✅ Puissant pour la validation de classes complexes
- ❌ Nécessite `reflect-metadata` et les decorators expérimentaux
- ❌ Classes DTO en plus des types TypeScript
- ❌ Verbosité élevée

### Option 3 — Zod ✅
Bibliothèque de validation avec inférence TypeScript native.
- ✅ Inférence de type automatique — `z.infer<typeof schema>` élimine la duplication
- ✅ API fluide et lisible
- ✅ Messages d'erreur structurés et i18n-ready
- ✅ Compatible avec `@folia/errors` via `.safeParse()`
- ✅ Réutilisable entre controllers et tests
- ✅ Zod v4 — performances améliorées
- ❌ JSON Schema natif de Fastify non utilisé — validation manuelle dans le controller

## Décision
**Zod pour la validation de toutes les entrées HTTP.**

## Raisons
- L'inférence de type élimine la duplication entre schéma et type TypeScript
- L'API est lisible et maintenable
- `safeParse()` s'intègre naturellement avec le pattern de gestion d'erreurs du controller

## Conséquences
- Les schémas sont dans `interface/schemas/plant.schema.ts`
- La validation est faite en début de handler avec `schema.safeParse(request.body)`
- En cas d'échec, une `ValidationError` (`@folia/errors`) est lancée
- Les types des handlers sont inférés depuis les schémas Zod : `z.infer<typeof createPlantSchema>`
- `z.nativeEnum()` est utilisé pour tous les enums du domaine
- `z.coerce.number()` est utilisé pour les query params (toujours des strings en HTTP)
