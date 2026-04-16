# CI/CD — Intégration et Déploiement Continu

## Outil

**GitHub Actions** — intégré au repo, gratuit pour les projets open source.

---

## Branches et environnements

| Branche | Environnement | Déclencheur |
|---|---|---|
| `feat/*`, `fix/*` | — | Tests uniquement (pas de déploiement) |
| `develop` | Staging | Merge automatique après tests |
| `main` | Production | Tag de release `v*.*.*` |

---

## Pipeline complet

```
Push / PR
    │
    ▼
┌─────────────────────────────────┐
│  CI — Quality Gate               │
│  1. Lint (ESLint)                │
│  2. Type check (tsc --noEmit)    │
│  3. Tests unitaires (Vitest)     │
│  4. Build (tsc)                  │
└─────────────────────────────────┘
    │ (si develop ou main)
    ▼
┌─────────────────────────────────┐
│  CI — Integration Tests          │
│  5. Tests intégration            │
│     (Testcontainers)             │
└─────────────────────────────────┘
    │ (si develop)
    ▼
┌─────────────────────────────────┐
│  CD — Staging                    │
│  6. Build Docker images          │
│  7. Push sur registry            │
│  8. Deploy staging               │
│  9. Smoke tests                  │
└─────────────────────────────────┘
    │ (si tag v*.*.*)
    ▼
┌─────────────────────────────────┐
│  CD — Production                 │
│  10. Deploy production           │
│  11. Migrations DB               │
│  12. Health checks               │
│  13. Rollback auto si échec      │
└─────────────────────────────────┘
```

---

## Fichiers GitHub Actions

```
.github/
  workflows/
    ci.yml           → lint, types, tests unitaires (toutes les branches)
    integration.yml  → tests intégration (develop, main)
    staging.yml      → déploiement staging (develop)
    production.yml   → déploiement production (tags v*.*.*)
```

### ci.yml — Quality Gate
```yaml
name: CI

on:
  push:
    branches: ['*']
  pull_request:

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm check-types
      - run: pnpm test
      - run: pnpm build
```

### integration.yml — Tests d'intégration
```yaml
name: Integration Tests

on:
  push:
    branches: [develop, main]

jobs:
  integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm test:integration
        env:
          # Testcontainers démarre Docker automatiquement
          DOCKER_HOST: unix:///var/run/docker.sock
```

---

## Gestion des secrets

### Règles strictes
- Aucun secret dans le code ou les fichiers de config
- Tous les secrets dans **GitHub Secrets** (Settings → Secrets)
- Différents secrets par environnement (`_STAGING`, `_PRODUCTION`)

### Secrets requis
```
DATABASE_URL_STAGING
DATABASE_URL_PRODUCTION
JWT_SECRET_STAGING
JWT_SECRET_PRODUCTION
DOCKER_REGISTRY_TOKEN
DEPLOYMENT_TOKEN
```

---

## Docker

### Build des images
```dockerfile
# Dockerfile par service (multi-stage)
FROM node:18-alpine AS builder
WORKDIR /app
COPY . .
RUN pnpm install --frozen-lockfile
RUN pnpm --filter @folia/catalogue build

FROM node:18-alpine AS runner
WORKDIR /app
COPY --from=builder /app/services/catalogue/dist ./dist
COPY --from=builder /app/services/catalogue/package.json .
RUN pnpm install --prod
CMD ["node", "dist/index.js"]
```

### Registry
- **GitHub Container Registry (GHCR)** — intégré à GitHub, gratuit
- Tags : `latest` (develop), `v1.2.3` (releases)

---

## Migrations en production

Les migrations sont appliquées **avant** le déploiement du nouveau code :

```yaml
steps:
  - name: Run migrations
    run: pnpm --filter @folia/catalogue db:migrate
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL_PRODUCTION }}

  - name: Deploy service
    # déploiement seulement si migrations OK
```

---

## Rollback

En cas d'échec des health checks après déploiement :
- Rollback automatique vers l'image précédente
- Alerte immédiate (Slack ou email)
- Les migrations déjà appliquées ne sont pas rollbackées (forward-only)

---

## Smoke Tests

Tests rapides post-déploiement pour vérifier que le service répond :

```bash
# Health check
curl -f https://staging.folia.app/health

# Endpoint critique
curl -f https://staging.folia.app/v1/plants?limit=1
```

---

## Mobile — EAS Build

```yaml
# .github/workflows/mobile.yml
name: Mobile Build

on:
  push:
    branches: [develop]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: eas build --platform all --profile preview --non-interactive
```
