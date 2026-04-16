# ADR-004 — Fastify comme framework HTTP

## Statut
Accepté

## Contexte
Le service catalogue expose une API REST. Il faut choisir un framework HTTP Node.js.

## Options considérées

### Option 1 — Express
Framework HTTP le plus utilisé dans l'écosystème Node.js.
- ✅ Très répandu, documentation abondante
- ✅ Écosystème de middlewares immense
- ❌ Pas de TypeScript natif — types via `@types/express`
- ❌ Performances moindres vs Fastify
- ❌ Validation non intégrée
- ❌ Architecture vieillissante

### Option 2 — Hono
Framework moderne, ultra-léger, compatible Edge.
- ✅ Très rapide
- ✅ TypeScript natif
- ✅ Compatible Cloudflare Workers, Deno, Bun
- ❌ Écosystème plus petit
- ❌ Moins de plugins disponibles

### Option 3 — Fastify ✅
Framework HTTP haute performance avec TypeScript natif.
- ✅ Performances excellentes — parmi les plus rapides des frameworks Node.js
- ✅ TypeScript natif avec inférence des types de routes
- ✅ Écosystème de plugins riche (`@fastify/cors`, `@fastify/sensible`...)
- ✅ Validation JSON Schema intégrée (extensible avec Zod)
- ✅ Logger Pino intégré — structuré et performant
- ✅ Plugins isolés — pas de conflit entre les dépendances
- ❌ Courbe d'apprentissage légèrement plus élevée qu'Express

## Décision
**Fastify.**

## Raisons
- Performances supérieures importantes pour une API qui sera sollicitée
- TypeScript natif cohérent avec le reste du projet
- L'écosystème de plugins couvre tous nos besoins (CORS, sensible, JWT...)
- Pino logger intégré — pas besoin d'une librairie de log supplémentaire

## Conséquences
- La validation HTTP est faite avec Zod dans les controllers (pas le JSON Schema natif de Fastify)
- Les routes sont organisées en plugins Fastify avec le pattern `async function routes(fastify, options)`
- `@fastify/sensible` est utilisé pour les helpers HTTP standards
- `@fastify/cors` est configuré avec les origines autorisées via variable d'environnement
