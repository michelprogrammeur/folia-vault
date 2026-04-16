# Monitoring & Observabilité

## Philosophie
Les trois piliers de l'observabilité : **logs**, **métriques**, **traces**.

## Logs

### Stack
- **Pino** — logger structuré JSON intégré à Fastify
- **Centralisation** : Loki (auto-hébergé) ou Datadog / Logtail
- Niveau de log par environnement : `debug` (dev), `info` (staging), `warn` (prod)

### Ce qu'on log
- Toutes les requêtes HTTP (method, path, statusCode, responseTime)
- Erreurs applicatives avec stack trace
- Événements métier importants (plante publiée, compte créé...)
- Accès aux données sensibles (audit trail)

## Métriques

### Infrastructure
- CPU, mémoire, disque par service
- Latence et throughput des bases de données
- Taille des queues BullMQ

### Applicatives
- Requêtes par seconde (RPS)
- Taux d'erreur HTTP (4xx, 5xx)
- Latence P50, P95, P99 par route
- Taux de succès de l'identification par IA

### Business
- Nouveaux utilisateurs / jour
- Plantes ajoutées / jour
- Identifications réussies / jour
- Taux de rétention J1, J7, J30

### Stack
- **Prometheus** — collecte des métriques
- **Grafana** — dashboards et alertes
- Alternative managed : Datadog, New Relic

## Traces distribuées
- **OpenTelemetry** — instrumentation standard
- Trace end-to-end d'une requête à travers les services
- Utile pour diagnostiquer les latences dans les appels inter-services

## Alertes

### Seuils critiques (PagerDuty / alerte immédiate)
- Taux d'erreur 5xx > 1% sur 5 minutes
- Latence P99 > 2 secondes
- Service down (health check échoué)
- Espace disque < 10%

### Seuils warning (notification Slack)
- Taux d'erreur 4xx > 5%
- Latence P95 > 500ms
- Queue BullMQ > 1000 jobs en attente

## Health checks
Chaque service expose `/health` :
```json
{ "status": "ok", "db": "ok", "version": "1.0.0" }
```
