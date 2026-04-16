# Infrastructure

## Environnements

| Environnement | Description | Base de données |
|---|---|---|
| `local` | Machine du développeur | Docker Compose |
| `staging` | Environnement de test | Cloud (à définir) |
| `production` | Environnement live | Cloud (à définir) |

## Docker Compose

Le fichier `docker-compose.yml` à la racine du monorepo gère toute l'infrastructure locale.
Les services sont regroupés par **profils** pour ne démarrer que ce dont on a besoin.

```bash
# Démarrer l'infrastructure du service catalogue
pnpm infra:catalogue

# Arrêter tous les containers
pnpm infra:down

# Reset complet (supprime les volumes)
pnpm infra:reset
```

### Profils disponibles

| Profil | Services démarrés |
|---|---|
| `catalogue` | PostgreSQL (catalogue) |
| `identity` | PostgreSQL (identity), Redis |
| `search` | Meilisearch |
| `all` | Tout |

## Variables d'environnement

Chaque service possède un fichier `.env.example` à sa racine.
Le fichier `.env` réel n'est jamais commité (`.gitignore`).

### Variables communes à tous les services
```env
NODE_ENV=development
PORT=300x
HOST=0.0.0.0
LOG_LEVEL=debug
```

### Variables spécifiques au service catalogue
```env
DATABASE_URL=postgresql://folia:folia@localhost:5432/folia_catalogue
ALLOWED_ORIGINS=http://localhost:3000
```

## Base de données

### Migrations
Les migrations sont gérées par **Drizzle Kit** dans chaque service.

```bash
# Générer une migration
pnpm --filter @folia/catalogue db:generate

# Appliquer les migrations
pnpm --filter @folia/catalogue db:migrate

# Ouvrir Drizzle Studio
pnpm db:studio
```

Les fichiers de migration sont versionnés dans `src/features/<feature>/infrastructure/database/migrations/`.

### Conventions
- Une migration = un changement logique (pas un dump complet)
- Jamais de `DROP COLUMN` sans migration de données préalable
- Les migrations sont toujours testées en local avant commit

## CI/CD *(à venir)*

### Pipeline cible

```
Push → Lint → Type check → Tests unitaires → Build
                                    ↓
                         Tests d'intégration
                                    ↓
                    Deploy staging (branche develop)
                                    ↓
                    Deploy production (tag release)
```

### Outils envisagés
- **GitHub Actions** pour le CI/CD
- **Docker Hub ou GHCR** pour les images
- **Fly.io ou Railway** pour le déploiement initial (simple, peu coûteux)
- **Kubernetes** pour la production à grande échelle (V3)

## Logging

- **Développement** : logs structurés via Fastify logger (Pino)
- **Production** : logs JSON centralisés (à définir : Datadog, Grafana Loki...)
- Niveau de log configurable via `LOG_LEVEL`
- Jamais de données personnelles dans les logs

## Monitoring *(à venir)*

- **Health check** : `GET /health` sur chaque service
- **Métriques** : Prometheus + Grafana
- **Alerting** : PagerDuty ou équivalent
- **Tracing distribué** : OpenTelemetry
