# Workflows CI/CD

## Environnements

| Environnement | Branche | Déploiement | Usage |
|---------------|---------|-------------|-------|
| Development | `feat/*`, `fix/*` | Manuel / local | Développement actif |
| Staging | `develop` | Automatique | Validation avant prod |
| Production | `main` | Automatique après validation | Utilisateurs réels |

## Pipeline CI (GitHub Actions)

### Déclencheurs
- Push sur toutes les branches : lint + typecheck + tests unitaires
- Push sur `develop` : CI complète + déploiement staging
- Push sur `main` : CI complète + déploiement production

### Étapes CI
```
1. Install         → pnpm install --frozen-lockfile
2. Typecheck       → turbo run typecheck
3. Lint            → turbo run lint
4. Test unitaires  → turbo run test
5. Build           → turbo run build
6. Test intégration → turbo run test:integration (Docker requis)
```

### Optimisations
- Cache Turborepo entre les runs CI (Remote Cache Vercel)
- Cache `node_modules` via actions/cache
- Tests en parallèle par package (`turbo run test --parallel`)
- Build uniquement des packages affectés (`--filter=[HEAD^1]`)

## Pipeline CD

### Déploiement staging (branche `develop`)
1. CI verte
2. Build des images Docker
3. Push sur le registry (GitHub Container Registry)
4. Déploiement sur l'environnement staging
5. Smoke tests automatiques (health checks)
6. Notification Slack

### Déploiement production (branche `main`)
1. CI verte
2. Build des images Docker taguées avec le SHA du commit
3. Push sur le registry
4. Déploiement progressif (rolling update)
5. Smoke tests + vérification des métriques (5 min)
6. Rollback automatique si taux d'erreur > 1%
7. Notification Slack

## Stratégie de branches

```
main          ← production, protégée
  └─ develop  ← staging, protégée
       └─ feat/xxx  ← features
       └─ fix/xxx   ← bugfixes
       └─ chore/xxx ← maintenance
```

- PR obligatoire pour merger dans `develop` et `main`
- Au moins 1 review requise
- CI verte obligatoire avant merge
- Squash merge pour garder un historique propre

## Versioning
- **Semantic Versioning** : `MAJOR.MINOR.PATCH`
- Tags Git sur `main` pour chaque release : `v1.2.3`
- CHANGELOG généré automatiquement depuis les commits conventionnels

## À faire
- [ ] Créer les GitHub Actions workflows (`.github/workflows/`)
- [ ] Configurer le Turborepo Remote Cache
- [ ] Choisir la plateforme de déploiement (Fly.io, Railway, Render...)
- [ ] Configurer les secrets GitHub (registry, déploiement)
- [ ] Mettre en place les smoke tests post-déploiement
