# Logging

## Outil

**Pino** via Fastify logger — logs structurés JSON, haute performance.

## Niveaux de log

| Niveau | Usage | Environnement |
|---|---|---|
| `trace` | Détails très fins, rarement utilisé | local uniquement |
| `debug` | Informations de développement | local, staging |
| `info` | Événements normaux importants | tous |
| `warn` | Situation anormale mais non bloquante | tous |
| `error` | Erreur récupérée avec impact utilisateur | tous |
| `fatal` | Erreur non récupérée, service en arrêt | tous |

Configuration par environnement :
```env
LOG_LEVEL=debug   # local
LOG_LEVEL=info    # staging, production
```

## Format

### Développement — texte lisible (pretty print)
```
[10:30:00] INFO: GET /v1/plants 200 45ms
[10:30:01] ERROR: Plant not found: uuid-123
```

### Production — JSON structuré
```json
{
  "level": "info",
  "time": "2025-04-14T10:30:00.000Z",
  "service": "catalogue",
  "requestId": "req-uuid-123",
  "method": "GET",
  "url": "/v1/plants",
  "statusCode": 200,
  "responseTime": 45
}
```

## Ce qu'on log

### Toujours logger
- Démarrage et arrêt du service (`info`)
- Chaque requête HTTP : méthode, path, status, durée (`info`)
- Erreurs 5xx avec stack trace (`error`)
- Connexion/déconnexion base de données (`info`)
- Événements domaine émis (`info`)
- Requêtes SQL > 100ms (`warn`)

### Logger selon le contexte
- Erreurs 4xx — uniquement `warn`, pas `error` (c'est le comportement attendu)
- Résultats de cache (hit/miss) — `debug` uniquement
- Détails des requêtes entrantes — `debug` uniquement

## Ce qu'on ne log JAMAIS

- Mots de passe, tokens JWT, clés API
- Données personnelles : email, nom, adresse, téléphone
- Numéros de carte bancaire ou données de paiement
- Contenu des messages privés
- Coordonnées GPS précises

## Corrélation des requêtes

Chaque requête reçoit un `requestId` unique (UUID) généré par la gateway.
Ce `requestId` est propagé dans tous les logs et dans les appels inter-services.

```typescript
// Fastify — génération automatique
server.addHook('onRequest', (request, reply, done) => {
  request.log = request.log.child({ requestId: request.id })
  done()
})
```

## Contexte obligatoire dans chaque log

```json
{
  "service": "catalogue",
  "requestId": "uuid",
  "userId": "uuid-or-null"
}
```

## Erreurs

Les erreurs loggées incluent toujours :
- Le message d'erreur
- Le stack trace (en développement et staging)
- Le `requestId` pour la corrélation
- Le contexte métier (ex: `plantId`, `speciesId`)

```typescript
request.log.error({
  err: error,
  plantId: command.plantId,
  msg: 'Failed to publish plant'
})
```

## Centralisation *(production)*

En production, les logs JSON sont envoyés vers un système centralisé :
- **Grafana Loki** (option open source)
- **Datadog** (option SaaS)
- **AWS CloudWatch** (si hébergé AWS)

Rétention des logs :
- `debug` / `info` : 30 jours
- `warn` / `error` / `fatal` : 90 jours
