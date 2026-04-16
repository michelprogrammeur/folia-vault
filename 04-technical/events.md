# Événements — Infrastructure Technique

## Principe

Les événements domaine sont le moyen de communication asynchrone entre les services.
Ce fichier documente l'infrastructure technique qui les transporte — pas les événements métier eux-mêmes (voir `03-domain/domain-events.md`).

---

## Stack par phase

### MVP — BullMQ + Redis
Simple, peu coûteux, suffisant pour démarrer.

```
Service A → Redis (BullMQ queue) → Service B
```

- Pas de broker externe à gérer
- Redis est déjà dans la stack pour le cache
- Garanties : at-least-once delivery
- Limites : pas de replay facile, pas de fan-out natif

### V2 — RabbitMQ
Quand plusieurs services consomment le même événement.

```
Service A → RabbitMQ Exchange → Queue B → Service B
                             → Queue C → Service C
```

- Fan-out natif (un événement → plusieurs consommateurs)
- Dead letter queue intégrée
- Interface de management visuelle

### V3 — Kafka
Quand le volume devient important et que le replay est nécessaire.

```
Service A → Kafka Topic → Consumer Group B
                       → Consumer Group C
                       → Consumer Group D (analytics)
```

- Rétention des événements (replay possible)
- Haute disponibilité et scalabilité
- Idéal pour `analytics` qui consomme tous les événements

---

## Garanties de livraison

### At-least-once (MVP et V2)
- Un événement peut être livré plusieurs fois en cas d'erreur
- Les consommateurs **doivent être idempotents**
- Vérifier `eventId` avant de traiter pour éviter les doublons

```typescript
// Pattern idempotent
async function handlePlantCreated(event: PlantCreated) {
  const alreadyProcessed = await processedEventsRepo.exists(event.eventId)
  if (alreadyProcessed) return

  await doWork(event)
  await processedEventsRepo.mark(event.eventId)
}
```

### Exactly-once (V3 — Kafka)
- Kafka avec transactions garantit exactly-once
- Plus complexe à implémenter mais élimine le besoin d'idempotence manuelle

---

## Dead Letter Queue (DLQ)

Quand un consommateur échoue après N tentatives, l'événement est envoyé en DLQ.

```
Événement → Queue → Consommateur (échec x3)
                          ↓
                    Dead Letter Queue
                          ↓
                    Alerte + traitement manuel
```

- Nombre de tentatives : 3 (configurable)
- Délai entre tentatives : exponentiel (1s, 5s, 30s)
- Alerte automatique quand un message arrive en DLQ
- Interface de replay pour renvoyer les événements échoués

---

## Structure d'un événement (rappel)

```json
{
  "eventId": "uuid-v4",
  "eventType": "PlantCreated",
  "occurredAt": "2025-04-14T10:30:00Z",
  "version": 1,
  "aggregateId": "uuid",
  "aggregateType": "Plant",
  "service": "catalogue",
  "userId": "uuid-or-null",
  "correlationId": "uuid-requete-origine",
  "payload": {}
}
```

---

## Nommage des queues et topics

### BullMQ / RabbitMQ
```
folia.catalogue.plant.created
folia.catalogue.plant.published
folia.collection.specimen.added
folia.gamification.points.earned
```

Format : `folia.<service>.<aggregate>.<action>`

### Kafka topics
```
folia-catalogue-events
folia-collection-events
folia-gamification-events
folia-all-events              ← agrégation pour analytics
```

---

## Versioning des événements

Un événement peut évoluer dans le temps sans casser les consommateurs :

```typescript
// Version 1 — initial
{ eventType: "PlantCreated", version: 1, payload: { scientificName, speciesId } }

// Version 2 — ajout de champs
{ eventType: "PlantCreated", version: 2, payload: { scientificName, speciesId, difficulty } }
```

- Les consommateurs ignorent les champs inconnus
- On ne supprime jamais un champ existant sans bump de version
- Les anciens consommateurs continuent de fonctionner avec la version 1

---

## Monitoring des événements

- Nombre de messages en queue (alerte si > seuil)
- Nombre de messages en DLQ (alerte immédiate)
- Latence de traitement par consommateur
- Taux d'erreur par type d'événement
