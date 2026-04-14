# Événements Domaine

> Liste exhaustive de tous les événements émis dans Folia.
> Un événement domaine = quelque chose qui s'est passé dans le système, nommé au passé.
> Format : `<Agrégat><Action>` en PascalCase.

---

## Format standard d'un événement

Chaque événement domaine respecte cette structure JSON :

```json
{
  "eventId": "uuid-v4",
  "eventType": "PlantCreated",
  "occurredAt": "2025-04-14T10:30:00Z",
  "version": 1,
  "aggregateId": "uuid-de-l-agregat",
  "aggregateType": "Plant",
  "service": "catalogue",
  "userId": "uuid-de-l-utilisateur-ou-null",
  "correlationId": "uuid-de-la-requete-origine",
  "payload": {
    // données spécifiques à l'événement
  }
}
```

**Champs obligatoires** : `eventId`, `eventType`, `occurredAt`, `version`, `aggregateId`, `aggregateType`, `service`
**Champs optionnels** : `userId` (null pour les événements système), `correlationId` (traçabilité)

**Règles** :
- `eventId` est unique — permet la déduplication en cas de rejeu
- `version` permet l'évolution du schéma sans casser les consommateurs
- `occurredAt` est toujours en UTC ISO 8601
- Le `payload` ne contient que les données nécessaires — pas l'agrégat complet

**Exemple concret** :
```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "PlantCreated",
  "occurredAt": "2025-04-14T10:30:00Z",
  "version": 1,
  "aggregateId": "7f3d9a1b-4c2e-4f8a-b6d5-1e2f3a4b5c6d",
  "aggregateType": "Plant",
  "service": "catalogue",
  "userId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "correlationId": "req-abc-123",
  "payload": {
    "scientificName": "Monstera deliciosa",
    "speciesId": "species-uuid",
    "type": "PLANT",
    "difficulty": "EASY"
  }
}
```

---

## catalogue

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `PlantCreated` | Nouvelle fiche espèce créée | search, notification |
| `PlantPublished` | Fiche rendue publique | search, notification |
| `PlantUnpublished` | Fiche retirée du catalogue | search |
| `PlantUpdated` | Fiche mise à jour | search, collection |
| `PlantDeleted` | Fiche supprimée | search, collection |

---

## collection

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `SpecimenAdded` | Plante ajoutée à une collection | gamification, notification |
| `SpecimenRemoved` | Plante retirée d'une collection | gamification |
| `JournalEntryCreated` | Événement noté dans le carnet | gamification |
| `ReminderTriggered` | Heure d'un rappel d'entretien atteinte | notification |
| `ReminderCompleted` | Rappel marqué comme effectué | gamification |
| `CareStreakUpdated` | Série d'entretien mise à jour | gamification, notification |
| `PlantingDateSet` | Date de plantation enregistrée | notification (anniversaire) |

---

## identity

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `UserRegistered` | Nouvel utilisateur inscrit | gamification, notification, social |
| `UserVerified` | Email vérifié | notification |
| `ProfileUpdated` | Profil mis à jour | social, search |
| `ProfileTypeChanged` | Type de profil modifié | gamification |
| `ExpertVerified` | Utilisateur obtient le statut expert | social, notification |
| `UserDeleted` | Compte supprimé | tous les services |

---

## identification

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `PlantIdentified` | Identification réussie | gamification, geo, collection |
| `IdentificationFailed` | Identification impossible | notification (suggestion expert) |
| `DiagnosticCompleted` | Diagnostic maladie terminé | gamification, notification |
| `ObservationValidated` | Observation validée par un expert | geo, gamification |

---

## gamification

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `PointsEarned` | Points attribués à un utilisateur | notification |
| `BadgeUnlocked` | Badge obtenu | notification, social |
| `GradeReached` | Nouveau grade atteint | notification, social |
| `ChallengeCompleted` | Défi terminé avec succès | notification, social |
| `SeasonEnded` | Fin d'une saison de jeu | notification, gamification |
| `StreakBroken` | Série interrompue | notification |
| `RewardUnlocked` | Récompense partenaire débloquée | notification |

---

## social

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `PostCreated` | Publication créée | notification, search |
| `PostLiked` | Publication aimée | gamification, notification |
| `CommentAdded` | Commentaire posté | gamification, notification |
| `ExpertAnswered` | Expert répond à une question | gamification, notification |
| `MessageSent` | Message privé envoyé | notification |
| `GroupCreated` | Groupe thématique créé | search, notification |
| `GroupJoined` | Utilisateur rejoint un groupe | gamification, notification |
| `LiveStarted` | Live streaming démarré | notification |

---

## geo

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `ObservationSubmitted` | Observation terrain soumise | gamification |
| `ObservationValidated` | Observation validée | geo, gamification, notification |
| `EventCreated` | Événement communautaire créé | notification, search |
| `EventAttended` | Utilisateur participe à un événement | gamification |
| `PlaceVisited` | Parc ou jardin botanique visité | gamification, notification |

---

## marketplace

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `ListingCreated` | Annonce publiée | search, notification |
| `ListingClosed` | Annonce clôturée | search |
| `TradeRequested` | Demande d'échange envoyée | notification |
| `TransactionCompleted` | Échange ou vente finalisé | gamification, notification |
| `SellerRated` | Vendeur noté après transaction | social |

---

## content

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `ArticleSubmitted` | Article soumis pour validation | notification (modérateurs) |
| `ArticlePublished` | Article validé et publié | search, notification, gamification |
| `PhotoContributed` | Photo soumise pour une fiche | notification (modérateurs) |
| `ContentValidated` | Contenu UGC accepté | gamification, notification |
| `ContentRejected` | Contenu UGC refusé | notification |

---

## i18n

| Événement | Déclencheur | Consommateurs |
|---|---|---|
| `TranslationContributed` | Traduction soumise par un utilisateur | notification (modérateurs), gamification |
| `TranslationValidated` | Traduction approuvée par deux locuteurs | catalogue, content, notification |
| `TranslationRejected` | Traduction refusée | notification |
| `LanguageActivated` | Langue atteint 90% de couverture | notification, tous les services |
| `LanguageDeactivated` | Langue désactivée | tous les services |

---

## Règles générales

- Les événements sont **immuables** — ils décrivent ce qui s'est passé, jamais ce qui doit se passer
- Chaque événement contient : `eventId`, `occurredAt`, `userId` (si applicable), et les données métier
- Les services **n'accèdent jamais directement** à la base de données d'un autre service
- En cas d'échec de traitement, les événements sont **rejouables** (idempotence requise)
