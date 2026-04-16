# Stack Technique

## Monorepo

| Outil | Rôle | Pourquoi |
|---|---|---|
| **Turborepo** | Orchestration du monorepo | Cache intelligent, pipeline de build optimisé |
| **pnpm** | Gestionnaire de paquets | Plus rapide que npm/yarn, workspaces natifs, économe en espace disque |
| **TypeScript** | Langage | Typage strict, meilleure maintenabilité, erreurs détectées à la compilation |

## Back-end

| Outil | Rôle | Pourquoi |
|---|---|---|
| **Node.js 18+** | Runtime | Écosystème riche, performance suffisante, équipe JS unifiée front/back |
| **Fastify** | Framework HTTP | Plus rapide qu'Express, TypeScript natif, validation intégrée, écosystème plugins |
| **Zod** | Validation des données | Inférence de types TypeScript automatique, composable, messages d'erreur clairs |
| **Drizzle ORM** | ORM PostgreSQL | Type-safety totale, migrations contrôlées, pas de magie cachée contrairement à Prisma |
| **PostgreSQL** | Base de données principale | Relationnel robuste, JSONB pour la flexibilité, extensions (uuid, pg_trgm...) |
| **Meilisearch** | Moteur de recherche | Full-text performant, facile à déployer, typo-tolérant, open source |

## Infrastructure

| Outil | Rôle | Pourquoi |
|---|---|---|
| **Docker** | Conteneurisation | Environnements reproductibles, déploiement cohérent |
| **Docker Compose** | Orchestration locale | Démarrer tous les services localement avec profils |

## Tests

| Outil | Rôle | Pourquoi |
|---|---|---|
| **Vitest** | Tests unitaires et intégration | Compatible ESM, rapide, API Jest-like, watch mode performant |
| **Testcontainers** | Tests d'intégration | Base de données réelle éphémère, pas de mocks trompeurs |

## Front-end *(prévu)*

| Outil | Rôle | Pourquoi |
|---|---|---|
| **Astro** | Site public / front-office | Performances excellentes, SSG/SSR, îles d'interactivité |
| **React** | Back-office | Écosystème riche, composants réutilisables |

## À venir

| Outil | Rôle | Phase |
|---|---|---|
| **Redis** | Cache et sessions | V1 étendu |
| **BullMQ** | File de messages (jobs asynchrones) | V1 étendu |
| **RabbitMQ / Kafka** | Message broker inter-services | V2 |
| **S3 / R2** | Stockage médias | V2 |
| **Kubernetes** | Orchestration production | V3 |
