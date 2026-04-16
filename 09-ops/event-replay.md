# Rejeu des événements échoués

## Contexte
Dans une architecture event-driven, les événements peuvent échouer pour diverses raisons : perte réseau, service temporairement indisponible, erreur inattendue, timeout... Il faut une stratégie pour ne perdre aucun événement.

## Sources d'événements

| Source | Technologie | Stratégie de retry |
|--------|------------|-------------------|
| Jobs background | BullMQ | Retry automatique avec backoff |
| Events inter-services | RabbitMQ → Kafka | Dead Letter Queue |
| Webhooks entrants | Fastify | Idempotence + retry côté émetteur |

## Stratégie de retry

### Retry avec backoff exponentiel
Ne jamais retenter immédiatement — attendre de plus en plus longtemps.

```
Tentative 1 : immédiat
Tentative 2 : 30 secondes
Tentative 3 : 5 minutes
Tentative 4 : 30 minutes
Tentative 5 : 2 heures
→ Dead Letter Queue
```

### Configuration BullMQ
```typescript
const job = await queue.add('send-notification', data, {
  attempts: 5,
  backoff: {
    type: 'exponential',
    delay: 30_000, // 30s
  },
});
```

## Dead Letter Queue (DLQ)

### Principe
Après épuisement des tentatives, l'événement est placé dans une DLQ — une queue spéciale pour les événements en échec.

### Ce qu'on stocke dans la DLQ
- L'événement original complet
- Le nombre de tentatives effectuées
- La dernière erreur (message + stack trace)
- Le timestamp du premier et dernier échec

### Interface de gestion
- **BullMQ** : Bull Board (UI web) pour visualiser et rejouer les jobs échoués
- **RabbitMQ** : management plugin pour inspecter les DLQ
- Alertes Grafana si la DLQ dépasse un seuil (ex: > 10 jobs)

## Idempotence

Un événement peut être traité plusieurs fois (at-least-once delivery). Les handlers doivent être idempotents.

### Pattern : clé d'idempotence
```typescript
// Stocker les event IDs traités
await db.insert(processedEvents)
  .values({ eventId, processedAt: new Date() })
  .onConflictDoNothing(); // Ignore si déjà traité
```

### Ce qui doit être idempotent
- Envoi d'emails (pas deux emails pour une même action)
- Incréments de compteurs (gamification)
- Webhooks vers des services tiers

## Rejeu manuel

### Quand rejouer ?
- Bug corrigé après une série d'échecs
- Service tiers était indisponible et est revenu
- Erreur de configuration corrigée

### Procédure
1. Identifier les événements échoués (Bull Board ou requête DLQ)
2. Vérifier que la cause racine est résolue
3. Rejouer par batch — pas tout d'un coup pour éviter de surcharger le service
4. Surveiller les métriques pendant le rejeu

### Commande de rejeu (BullMQ)
```typescript
// Rejouer tous les jobs échoués d'une queue
const failedJobs = await queue.getFailed();
for (const job of failedJobs) {
  await job.retry();
}
```

## Événements non rejouables
Certains événements ne doivent pas être rejoués mécaniquement :
- Paiements (risque de double facturation)
- Envoi de SMS / notifications push (doublon visible par l'utilisateur)

Pour ces cas : analyse manuelle obligatoire avant tout rejeu.

## À faire
- [ ] Configurer Bull Board pour la visualisation des jobs BullMQ
- [ ] Mettre en place les alertes DLQ dans Grafana
- [ ] Implémenter la table `processed_events` pour l'idempotence
- [ ] Documenter les événements non rejouables par service
