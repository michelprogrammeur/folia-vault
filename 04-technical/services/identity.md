# Service — Identity

## Rôle
Gestion de l'identité des utilisateurs : inscription, authentification, profil, rôles et permissions. Source de vérité pour tout ce qui concerne un utilisateur Folia.

## Stack

| Outil | Version | Rôle |
|---|---|---|
| Node.js | 18+ | Runtime |
| TypeScript | 5.x | Langage |
| Fastify | 5.x | Framework HTTP |
| Zod | 3.x | Validation des entrées |
| Drizzle ORM | 0.x | ORM PostgreSQL |
| PostgreSQL | 16 | Base de données |
| bcrypt | — | Hashage des mots de passe |
| jose | — | Génération et vérification des JWT |
| Vitest | 2.x | Tests unitaires et intégration |
| Testcontainers | — | PostgreSQL éphémère pour les tests |

## Port local
```
3002
```

## Base de données
```
folia_identity (PostgreSQL)
```

### Tables

| Table | Description |
|---|---|
| `users` | Comptes utilisateurs |
| `refresh_tokens` | Tokens de rafraîchissement actifs |

## Agrégats

### User
- Propriétaire de son cycle de vie complet
- Évènements : `UserRegistered`, `UserUpdated`, `UserDeleted`
- Invariants : email unique, mot de passe hashé (jamais en clair), age minimum 13 ans

## Endpoints HTTP

| Méthode | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/health` | Health check | Non |
| `POST` | `/auth/register` | Inscription | Non |
| `POST` | `/auth/login` | Connexion → access + refresh token | Non |
| `POST` | `/auth/refresh` | Renouveler l'access token | Refresh token |
| `POST` | `/auth/logout` | Révoquer le refresh token | Oui |
| `GET` | `/users/me` | Profil de l'utilisateur connecté | Oui |
| `PATCH` | `/users/me` | Mettre à jour son profil | Oui |
| `DELETE` | `/users/me` | Supprimer son compte | Oui |

## Stratégie JWT

- **Access token** : durée de vie courte (15 min), signé avec clé secrète
- **Refresh token** : durée de vie longue (30 jours), stocké en DB avec `revokedAt`
- La gateway vérifie l'access token — elle n'appelle pas identity à chaque requête
- En cas d'expiration, le client appelle `POST /auth/refresh` pour obtenir un nouvel access token

## Sécurité

- Mots de passe hashés avec bcrypt (coût 12) — jamais stockés en clair
- Rate limiting sur `/auth/login` et `/auth/register` (5 tentatives / 15 min)
- Soft delete avec `deletedAt` — les comptes supprimés restent 30 jours avant purge RGPD
- Voir `08-legal/gdpr.md` pour les obligations RGPD

## Structure interne

```
src/
  features/
    user/
      domain/
        aggregates/        → User.ts
        value-objects/     → UserId.ts, Email.ts, HashedPassword.ts
        repositories/      → UserRepositoryPort.ts, RefreshTokenRepositoryPort.ts
        events/            → UserRegistered.ts, UserUpdated.ts, UserDeleted.ts
      application/
        commands/
          RegisterUser/    → RegisterUserCommand, RegisterUserHandler
          LoginUser/       → LoginUserCommand, LoginUserHandler
          RefreshToken/    → RefreshTokenCommand, RefreshTokenHandler
          LogoutUser/      → LogoutUserCommand, LogoutUserHandler
          UpdateUser/      → UpdateUserCommand, UpdateUserHandler
          DeleteUser/      → DeleteUserCommand, DeleteUserHandler
        queries/
          GetCurrentUser/  → GetCurrentUserQuery, GetCurrentUserHandler, GetCurrentUserResult
          UserMapper.ts
      infrastructure/
        database/
          schema.ts
          migrations/
        mappers/
          UserPersistenceMapper.ts
        repositories/
          PostgresUserRepository.ts
          PostgresRefreshTokenRepository.ts
      interface/
        controllers/       → AuthController.ts, UserController.ts
        routes/            → auth.routes.ts, user.routes.ts
        schemas/           → auth.schema.ts, user.schema.ts (Zod)
  shared/
    database/              → connection.ts
    events/                → AggregateRoot.ts, DomainEvent.ts
```

## Événements émis

| Événement | Déclencheur |
|---|---|
| `UserRegistered` | POST /auth/register |
| `UserUpdated` | PATCH /users/me |
| `UserDeleted` | DELETE /users/me |

## Dépendances inter-services

| Service | Type | Usage |
|---|---|---|
| `notification` | Asynchrone | Email de bienvenue, confirmation d'email |
| `gateway` | Synchrone | Vérification JWT sur chaque requête entrante |

## Variables d'environnement

```env
NODE_ENV=development
PORT=3002
HOST=0.0.0.0
DATABASE_URL=postgresql://folia:folia@localhost:5432/folia_identity
JWT_SECRET=changeme
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=30d
ALLOWED_ORIGINS=http://localhost:3000
LOG_LEVEL=debug
```

## Commandes utiles

```bash
# Développement
pnpm --filter @folia/identity dev

# Tests unitaires
pnpm --filter @folia/identity test

# Tests intégration
pnpm --filter @folia/identity test:integration

# Générer une migration
pnpm --filter @folia/identity db:generate

# Appliquer les migrations
pnpm --filter @folia/identity db:migrate
```

## État actuel

- ⬜ Domain (User, value objects, events)
- ⬜ Application (Register, Login, Refresh, Logout, Update, Delete, GetCurrentUser)
- ⬜ Infrastructure (PostgresUserRepository, PostgresRefreshTokenRepository, migrations)
- ⬜ Interface (AuthController, UserController, routes, Zod schemas)
- ⬜ Tests unitaires
- ⬜ Tests intégration
